## meta data files
lookup csv for column delimeter per table , column names per table (this will be in the repo but deployed to dbfs)

## configuration files
1. location of drop zone
2. location of staging area
3. folder for metadata
4. folder for csv 


## fake data generator
1. create csv zip files based on schema, full tables. look up column del and column names.
2. data is stored in dbfs as the drop zone.
3. will run every 5 minutes

## data-pipeline trunk/load
1. is triggered via rest.

## triggering data pipeline
1. a separate java app that calls the rest api for each table in metadata lookup.
2. the fake data generator calls rest api 
3. both won't be run at the same time, but user's have the option to use both.

## delta updates
1. fake data generator calls via rest api