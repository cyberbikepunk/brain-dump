# Loic's design notes

__Warning!__ These the notes of a open data newbie. I'm still in that phase where a heavy fog prevents me from seeing the big picture. 

## What the CLI should do

### Set the user up

* Get an API key
* Create user config
* Create Sqlite DB

__How?__ Use __OS-conductor__ API. Oauth / JWT (string)

### Validate the model

__How?__ The `DataPackage` provides a validation method it seems.

__Better__ It could be nice to help the user build the `datapackage.json`, like the UX does. One possibility would be using something like `prompt_toolkit`. I've never used it before though, I don't know how hard it is do implement.

### Validate the data

It looks like `goodtables` was made for this. So let's use that for now.

### Modify a model

Download the `datapackage.json` model from ES and upload it back after validation.

### Upload/download a model or even data

Use __OS-API__ for that. I haven't looked precisely if the API does everything I want.

### Explore datasets

Explore the ES database. Show basic information about packages and data.

- Search datasets: search pattern and list the results 
- Preview a dataset: echo a few lines of the table
- Describe a dataset in a way people can understand 

I think the `PETL` library can be quite nice for previewing data. It has no dependencies and lazy loads so can handle very large datasets. Also a little bit of auto-completion would go a long way towards user-friendliness

Use __OS-conductor__ for that.

### Local database

I think I need to maintain a little local database with the following tables:

* collections `{'id', 'name'}`
* packages `{'id', 'name', 'path', 'collection_id'}`

The problem with this approach is that now we've got two databases (ES on the cloud and Sqlite locally) and the CLI interacts with both, which could generate some confusion. 

I still think it's useful though. I will help make the API much nicer (see below).

## Draft CLI usage 

### Brain dump

This is what the CLI usage could look like...

```bash

# Set-up the API-KEY
os setup [USERNAME] [--password PASSWORD] 
os collect [DIRECTORY] [--collection NAME] [--update]
os model [FILENAME | FILEPATH]
os validate [FILENAME | FILEPATH | COLLECTION]
os register [FILENAME | FILEPATH | COLLECTION] [--overwrite]
os push PACKAGE [--overwrite]
os pull PACKAGE [--data] [--overwrite]
os list [PATTERN | PACKAGE | COLLECTION] [--cloud]
os describe [FILENAME | FILEPATH | PACKAGE | COLLECTION] [--output FILENAME]
os view [FILENAME | FILEPATH | PACKAGE [RESSOURCE]] 

```

### The need for a small local SQLite DB

The above requires a little local database with the following tables:

* collections `{'id', 'name'}`
* packages `{'id', 'name', 'path', 'collection_id'}`

The problem with this approach is that now we've got two databases (ES on the cloud and Sqlite locally) and the CLI interacts with both, which could generate some confusion. 

## Python API

Is this package the right place to explose a Python API for what needs or what purpose? This is a completely dumb question at this stage. Like they say: if you don't need it right now, don't implement it.

* authenticate OAuth plus a JWT string
* upload datapackage tree to bucket  
* load data into postgres (need model) and into ES
* publish toggles (a switch)

## Metaphysical design questions going on in my head

### CKAN to OpenSpending

Okay so I found the [plugin](https://github.com/openspending/ckanext-openspending/blob/master/ckanext/openspending/plugin.py). The more I explore the more I discover that there's already been quite a lot of work done on the problem of data ingestion. People are coming at it from a lot of angles.

### If there's CKAN - Why OpenSpending?

Wasn't Rufus' idea to put all open-data into one place? Why build something specific? 

__Note__: It looks like the CKAN package JSON descriptors are not the same as datapackage standard.

### Embedding ETL into data packages

I've stumbled upon 2 cases where this design is kind of a natural way of looking at things: [farm-subsidies](http://github.com/openspending/farm-subsidies) and [cohesion-funds](http://github.com). Basically, it's naturalto organise data sources by country. And it follows that 

Also I guess it's already been somewhat implemented using __GNU Make__ in the [datasets repo](https://github.com/datasets). 

A good ETL pipeline requires logging, tracability, data quality control and maybe more.

I'm sure there's already been discussions of whether ETL processes should be included within datapackage specs. It's tempting at first, but then you're really moving away from the orginal idea of the datapackage as a *container*.

### Collections

I'd like the Open-Spending to support the notion of collections. Cohesion Funds are a collection for example.

### Sematics are ambiguous

"__Data package__" can mean the JSON descriptors or the directory tree containing the descriptor, the data and associated file. I find this can be confusing

### Datapackage versioning ?

One thing I stumbled upon whilst uploading the Mexican Federal budget is the problem of iteration. You have your dataset. Map it. Feel unsatisfied. Map it again. This raises the issue of do we just overwrite the `datapackage.json` or do we want to introduce versionning.  

### What the thinking behind 'goodtables' ?

I would like to understand the rationale for doing detailed / low-level validation of CSV files. Obviously you want to give the user as much feedback as possible. But doing it row by row seems to set a limit on the number of rows that can be put through the pipeline, which seems to imply validation false positives.

### Ooh how about messytables ?

Who uses it ?

### Code design

Okay so we have a `DataPackage` class in the datapackage-py repo. But do we have a `FiscalDataPackage` subclass defined somewhere? 

### DataPackage command

A command line utility already exists. Could the logical thing to do be extending it?

## Notes on OS-conductor

### Secrets 

The way secrets and keys are handled is really confusing. First: name things properly: some are for S3 and some for google. Next don't put google stuff into seperate files. Google OAuth info comes in a neat JSON file which has everything in it. 