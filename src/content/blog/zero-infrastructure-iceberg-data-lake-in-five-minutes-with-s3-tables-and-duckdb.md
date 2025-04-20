---
title: 'Zero-Infrastructure Iceberg in 5 Minutes'
pubDate: 'Apr 10 2025'
description: 'Zero-Infrastructure Iceberg in 5 Minutes'
heroImage: '/img/blog/zero-infrastructure-iceberg.png'
---

Operating a low-tradeoff Iceberg data lake shouldn't be so ~hard~ impossible. And finally it is not.


But wait, don't you need Spark or Some Java Thing™? How about background compaction? Manifest rewrites? Sourcing a new vendor only to use their (mostly-closed) catalog?

...*anything?!?*


### Nope.

<br>

Thanks to [S3 Tables](https://aws.amazon.com/s3/features/tables/), [DuckDB](https://duckdb.org/), and [Arrow](https://arrow.apache.org/) (behind the scenes) you can build a low-cost, low-touch Iceberg data lake.

**In five minutes, with no infrastructure, using a single vendor that you probably already have (AWS).**


This post outlines the steps to create an Iceberg lake (using Python), register appropriate resources, use [DuckDB](https://duckdb.org/) to generate an Iceberg table for [2024 Stack Overflow Developer Survey](https://survey.stackoverflow.co/2024/) data, and read data from the Iceberg lake - again using DuckDB.


<br>

#### Step 1: Create the S3 Table Bucket

You'll need a recent version of [awscli](https://aws.amazon.com/cli/) for this:

```bash
    aws s3tables create-table-bucket --cli-input-json '{"name": "jakes-lake"}'


    {
        "arn": "arn:aws:s3tables:us-east-1:$YOUR-ACCOUNT:bucket/jakes-lake"
    }

```

<br>

#### Step 2: Load the catalog and create a namespace

Using [pyiceberg](https://py.iceberg.apache.org/) to load the `catalog` with table-bucket-specific `warehouse` is simple. It uses a combination of `glue` and `lake formation` behind the scenes.


```python
import duckdb
from pyiceberg.catalog import Catalog, load_catalog


catalog = load_catalog(
    name="s3tablescatalog",
    **{
        "type":"rest",
        "uri":"https://glue.us-east-1.amazonaws.com/iceberg",
        "warehouse": "$YOUR-ACCOUNT:s3tablescatalog/jakes-lake",
        "rest.sigv4-enabled": "true",
        "rest.signing-name": "glue",
        "rest.signing-region": "us-east-1",
    }
)

catalog.create_namespace("stack_overflow", {"description": "Stack Overflow data"})
```

<br>

#### Step 3: Load data from a Cloudflare-hosted CSV to a local DuckDB table

DuckDB will grab a remote csv and auto-discover its schema. Pretty neat:


```python
# Connect to the local DuckDB database
d_conn = duckdb.connect(database="so.db")

# Install prerequisites
install_sql = """
INSTALL aws;
INSTALL httpfs;
INSTALL iceberg;
"""
d_conn.execute(install_sql)

# Load data from a CSV file sitting in a CDN
d_conn.execute("create table results as select * from read_csv('survey-results.csv')")
```

<br>

#### Step 4: Use the local DuckDB table to create an Iceberg table in S3

CSV type discovery gets better when used to create an Iceberg table with a single LOC:

```python
# Get the pyarrow schema
schema = d_conn.execute("select * from results limit 1").fetch_arrow_table().schema

# Use the schema to create an Iceberg table
catalog.create_table(
    identifier="stack_overflow.survey_results",
    schema=schema,
)

# Ensure the table exists
catalog.list_tables(namespace="stack_overflow")

# Load the newly-created Iceberg table
iceberg_tbl = catalog.load_table("stack_overflow.survey_results")

```

<br>

#### Step 5: Append rows from local table to the remote Iceberg table

This will run the specified query before iterating over result record batches, coercing them to a pyarrow table, and appending the table to the `stack_overflow.survey_results` Iceberg table:

```python
for record_batch in d_conn.execute("select * from results").fetch_record_batch():
    batch = pa.Table.from_batches([record_batch])
    print("appending record batch to iceberg table...")
    iceberg_tbl.append(batch)

```

<br>

#### Step 6: Use DuckDB to attach the S3 Tables catalog and query data

This example uses the [duckdb cli](https://duckdb.org/docs/stable/clients/cli/overview.html) to showcase the flexibilty of these tools.

This is a database. Without the database infrastructure and cost.

```sql

-- Install prerequisites
INSTALL aws;
INSTALL httpfs;
FORCE INSTALL iceberg FROM core_nightly;

-- Create an AWS secret which uses the credentials chain
CREATE SECRET (
    TYPE s3,
    PROVIDER credentials_chain
);

-- Attach the S3 Tables catalog
ATTACH 'arn:aws:s3tables:us-east-1:878889713122:bucket/jakes-lake'
    AS s3_tables_db (
        TYPE iceberg,
        ENDPOINT_TYPE s3_tables
    );

```


