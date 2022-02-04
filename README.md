# Introduction-to-Redis-Project
## Problem Statement

The task for this project is to write a Python script that imports data from a dummy
CSV file into a remote Redis database. 
> Find the project details here: [Project Brief](https://github.com/vtex-apps/login/blob/main/docs/README.md)

**Data**

The dummy CSV used contains data on terrorism and related offenses, scraped from websites and is saved in a csv file `dummy.csv`

The data file is updated on a daily basis. We will use a delta of 1 day to demonstrate ***updating on redis***. The delta data is saved in the `dummy_update.csv` file

`Upload the csv files`

```python
# Upload dummy csv files

from google.colab import files
files.upload()
```
**Redis Database**

The remote Redis database is created on the [Redis website](https://redis.com/try-free) using the free fixed plan which does not require payment

`Connect to the remote database`

```python
# Install redis-py
!pip install redis

# Connect to Redis
import redis

r = redis.StrictRedis(
    host='redis-12506.c257.us-east-1-3.ec2.cloud.redislabs.com',
    port=12506,
    password='passoword_string',
    db=0,
    decode_responses=True
)
```
### Tasks
#### 1. Import data from a dummy CSV file into the remote Redis database
```python
import csv

# Read the csv file
f = csv.reader(open("dummy.csv",encoding='ISO 8859-1'))
header = next(f)

# Create a list to hold the documents to be uploaded
doclist= []

# Append the docs to the list
for row in f:
  doc = dict(zip(header, row))
  doclist.append(doc)

# We will use json for the bulk upload since the `set` redis command overwrites data inserted individually
import json

# Define a variable for the redis documents key
key = "dummy"

# Insert all the documents to the database
r.set(key, json.dumps(doclist))
```
#### 2. Retrieve the data from Redis and store it in a Pandas data frame
```python
import pandas as pd

# Define a pandas dataframe variable
myoutput = pd.DataFrame()

# Download the data from the remote database and store in a variable 'data'
data = json.loads(r.get(key))

# Iterate over all the values in 'data' and insert in the dataframe
for item in data:
  myoutput = myoutput.append(item, ignore_index=True)
```
#### 3. Update the data on the Redis database
```python
# Read in the data from the delta csv file
f = csv.reader(open("dummy_update.csv",encoding='ISO 8859-1'))
header = next(f)

# Save the data in a new list
doclist_update= []

for row in f:
  doc = dict(zip(header, row))
  doclist_update.append(doc)

# Combine the previous list with the new
updated_list = doclist_update + doclist

# Upload the new list to the database
r.set(key, json.dumps(updated_list))
```
> ***The lists have to be combined otherwise importing only the new dataset will overwrite the existing data***

#### 4. Delete the data from the database
```python
# Delete the data
r.flushdb()

# Confirm deletion
print(r.keys())
```
