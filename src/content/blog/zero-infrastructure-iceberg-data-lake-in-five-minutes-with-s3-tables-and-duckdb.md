---
title: 'Creating a Zero-Infrastructure Iceberg Data Lake in 5 Minutes'
pubDate: 'Apr 10 2025'
description: 'Zero-Infrastructure Iceberg in 5 Minutes'
heroImage: '/img/blog/zero-infrastructure-iceberg.png'
---

Operating a low-tradeoff Iceberg data lake shouldn't be so ~hard~ impossible. And finally it is not.


But wait, don't you need Spark or Some Java Thing™? How about background compaction? Manifest rewrites? Sourcing a new vendor only to use their (mostly-closed) catalog?

...*anything?!?*


### Nope.

<br>

Thanks to [AWS S3 Tables](https://aws.amazon.com/s3/features/tables/), [DuckDB](https://duckdb.org/), and [Arrow](https://arrow.apache.org/) you can build a low-cost, low-touch Iceberg data lake.

**In five minutes, with no infrastructure, using a single vendor that you probably already have (AWS).**


This post outlines the steps to create an Iceberg lake (using Python), register appropriate resources, use [DuckDB](https://duckdb.org/) to generate an Iceberg table for [2024 Stack Overflow Developer Survey](https://survey.stackoverflow.co/2024/) data, and read data from the Iceberg lake - again using DuckDB.


<br>

#### Step 1: Create the S3 Table Bucket

You'll need a recent version of [awscli](https://aws.amazon.com/cli/) for this:

```bash
    aws s3tables create-table-bucket --cli-input-json '{"name": "$YOUR-BUCKET"}'


    {
        "arn": "arn:aws:s3tables:us-east-1:$YOUR-ACCOUNT:bucket/$YOUR-BUCKET"
    }

```

<br>

#### Step 2: Load the catalog and create a namespace

Loading the `catalog` and table-bucket-specific `warehouse` is simple with [pyiceberg](https://py.iceberg.apache.org/). It uses a combination of `glue` and `lake formation` behind the scenes.


```python
import duckdb
from pyiceberg.catalog import load_catalog
import pyarrow as pa


catalog = load_catalog(
    name="s3tablescatalog",
    **{
        "type":"rest",
        "uri":"https://glue.us-east-1.amazonaws.com/iceberg",
        "warehouse": "$YOUR-ACCOUNT:s3tablescatalog/$YOUR-BUCKET",
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
d_conn.execute("create table results as select * from read_csv('https://assets.jakthom.dev/zero-infrastructure-lake/survey-results.csv.gz');")
```

The resulting table looks like this (using DuckDB cli):

```
D desc results;
┌──────────────────────┬─────────────┬─────────┬─────────┬─────────┬─────────┐
│     column_name      │ column_type │  null   │   key   │ default │  extra  │
│       varchar        │   varchar   │ varchar │ varchar │ varchar │ varchar │
├──────────────────────┼─────────────┼─────────┼─────────┼─────────┼─────────┤
│ responseid           │ BIGINT      │ YES     │ NULL    │ NULL    │ NULL    │
│ mainbranch           │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ age                  │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ employment           │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ remotework           │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ check                │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ codingactivities     │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ edlevel              │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ learncode            │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ learncodeonline      │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ techdoc              │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ yearscode            │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ yearscodepro         │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ devtype              │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ orgsize              │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ purchaseinfluence    │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ buynewtool           │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ buildvsbuy           │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ techendorse          │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ country              │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│    ·                 │    ·        │  ·      │  ·      │  ·      │  ·      │
│    ·                 │    ·        │  ·      │  ·      │  ·      │  ·      │
│    ·                 │    ·        │  ·      │  ·      │  ·      │  ·      │
│ timesearching        │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ timeanswering        │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ frustration          │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ professionaltech     │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ professionalcloud    │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ professionalquestion │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ industry             │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsatpoints_1       │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsatpoints_4       │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsatpoints_5       │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsatpoints_6       │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsatpoints_7       │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsatpoints_8       │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsatpoints_9       │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsatpoints_10      │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsatpoints_11      │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ surveylength         │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ surveyease           │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ convertedcompyearly  │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsat               │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
├──────────────────────┴─────────────┴─────────┴─────────┴─────────┴─────────┤
│ 114 rows (40 shown)                                              6 columns │
└────────────────────────────────────────────────────────────────────────────┘
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

This works because by using DuckDB's [arrow](https://duckdb.org/2021/12/03/duck-arrow.html) integration, we are able to retain table structure and metadata:

```python
In [9]:  tbl = d_conn.execute("select * from results limit 1").fetch_arrow_table()

In [10]: tbl
Out[10]:
pyarrow.Table
responseid: int64
mainbranch: string
age: string
employment: string
remotework: string
check: string
codingactivities: string
edlevel: string
learncode: string
learncodeonline: string
techdoc: string
yearscode: string
yearscodepro: string
devtype: string
orgsize: string
purchaseinfluence: string
buynewtool: string
buildvsbuy: string
techendorse: string
country: string
currency: string
comptotal: string
languagehaveworkedwith: string
languagewanttoworkwith: string
languageadmired: string
databasehaveworkedwith: string
databasewanttoworkwith: string
databaseadmired: string
platformhaveworkedwith: string
platformwanttoworkwith: string
platformadmired: string
webframehaveworkedwith: string
webframewanttoworkwith: string
webframeadmired: string
embeddedhaveworkedwith: string
embeddedwanttoworkwith: string
embeddedadmired: string
misctechhaveworkedwith: string
misctechworktoworkwith: string
misctechadmired: string
toolstechhaveworkedwith: string
toolstechwanttoworkwith: string
toolstechadmired: string
newcollabtoolshaveworkedwith: string
newcollabtoolswanttoworkwith: string
newcollabtoolsadmired: string
opsyspersonaluse: string
opsysprofessionaluse: string
officestackasynchronoususe: string
officestackasyncwanttoworkwith: string
officestackasyncadmired: string
officestacksynchaveworkedwith: string
officestacksyncwanttoworkwith: string
officestacksyncadmired: string
aisearchdevhaveworkedwith: string
aisearchdevwanttoworkwith: string
aisearchdevadmired: string
newsosites: string
sovisitfreq: string
soaccount: string
sopartfreq: string
sohow: string
socomm: string
aiselect: string
aisent: string
aiben: string
aiacc: string
aicomplex: string
aitoolcurrentlyusing: string
aitoolinterestedinusing: string
aitoolnotinterestedinusing: string
ainextmuchmoreintegrated: string
ainextnochange: string
ainextmoreintegrated: string
ainextlessintegrated: string
ainextmuchlessintegrated: string
aithreat: string
aiethics: string
aichallenges: string
tbranch: string
icorpm: string
workexp: string
knowledge_1: string
knowledge_2: string
knowledge_3: string
knowledge_4: string
knowledge_5: string
knowledge_6: string
knowledge_7: string
knowledge_8: string
knowledge_9: string
frequency_1: string
frequency_2: string
frequency_3: string
timesearching: string
timeanswering: string
frustration: string
professionaltech: string
professionalcloud: string
professionalquestion: string
industry: string
jobsatpoints_1: string
jobsatpoints_4: string
jobsatpoints_5: string
jobsatpoints_6: string
jobsatpoints_7: string
jobsatpoints_8: string
jobsatpoints_9: string
jobsatpoints_10: string
jobsatpoints_11: string
surveylength: string
surveyease: string
convertedcompyearly: string
jobsat: string
```

Which can then be used directly, to create the Iceberg table:

```python
catalog.create_table(
    identifier="stack_overflow.survey_results",
    schema=schema # <---------- me me me look at me
)
```

<br>

#### Step 5: Append rows from local table to the remote Iceberg table

This part is about as easy as it gets.

1. Get all records from the origin table
2. Iterate over record batches and coercing them to a pyarrow table
3. Append pyarrow tbl to the `stack_overflow.survey_results` Iceberg table

Like this:

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
FORCE INSTALL iceberg FROM core_nightly; -- This will change to FORCE INSTALL iceberg at some point. IDK when.

-- Create an AWS secret which uses the credentials chain
CREATE SECRET (
    TYPE s3,
    PROVIDER credential_chain
);

-- Attach the S3 Tables catalog
ATTACH 'arn:aws:s3tables:us-east-1:$YOUR-ACCOUNT:bucket/$YOUR-BUCKET'
    AS jakes_lake (
        TYPE iceberg,
        ENDPOINT_TYPE s3_tables
    );

```

Once the catalog has been attached, it is ready to use.

**Set the search path and check the table out:**

```sql
D use jakes_lake.stack_overflow;
D show tables;
┌────────────────┐
│      name      │
│    varchar     │
├────────────────┤
│ survey_results │
└────────────────┘
D desc table survey_results;
┌──────────────────────┬─────────────┬─────────┬─────────┬─────────┬─────────┐
│     column_name      │ column_type │  null   │   key   │ default │  extra  │
│       varchar        │   varchar   │ varchar │ varchar │ varchar │ varchar │
├──────────────────────┼─────────────┼─────────┼─────────┼─────────┼─────────┤
│ responseid           │ BIGINT      │ YES     │ NULL    │ NULL    │ NULL    │
│ mainbranch           │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ age                  │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ employment           │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ remotework           │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ check                │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ codingactivities     │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ edlevel              │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ learncode            │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ learncodeonline      │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ techdoc              │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ yearscode            │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ yearscodepro         │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ devtype              │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ orgsize              │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ purchaseinfluence    │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ buynewtool           │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ buildvsbuy           │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ techendorse          │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ country              │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│    ·                 │    ·        │  ·      │  ·      │  ·      │  ·      │
│    ·                 │    ·        │  ·      │  ·      │  ·      │  ·      │
│    ·                 │    ·        │  ·      │  ·      │  ·      │  ·      │
│ timesearching        │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ timeanswering        │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ frustration          │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ professionaltech     │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ professionalcloud    │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ professionalquestion │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ industry             │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsatpoints_1       │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsatpoints_4       │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsatpoints_5       │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsatpoints_6       │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsatpoints_7       │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsatpoints_8       │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsatpoints_9       │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsatpoints_10      │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsatpoints_11      │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ surveylength         │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ surveyease           │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ convertedcompyearly  │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ jobsat               │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
├──────────────────────┴─────────────┴─────────┴─────────┴─────────┴─────────┤
│ 114 rows (40 shown)                                              6 columns │
└────────────────────────────────────────────────────────────────────────────┘
```


**Query data from the Iceberg table in S3:**

```
D select responseid, age, employment, country from survey_results limit 25;
100% ▕████████████████████████████████████████████████████████████▏
┌────────────┬────────────────────┬──────────────────────────────────────────────────────────────────────────────────────────────┬──────────────────────────────────────────────────────┐
│ responseid │        age         │                                          employment                                          │                       country                        │
│   int64    │      varchar       │                                           varchar                                            │                       varchar                        │
├────────────┼────────────────────┼──────────────────────────────────────────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┤
│         91 │ 18-24 years old    │ I prefer not to say                                                                          │ Russian Federation                                   │
│         92 │ Under 18 years old │ Student, full-time                                                                           │ Poland                                               │
│         93 │ 55-64 years old    │ Employed, full-time                                                                          │ United States of America                             │
│         94 │ 25-34 years old    │ Employed, full-time                                                                          │ Ireland                                              │
│         95 │ Under 18 years old │ Student, part-time                                                                           │ Iran, Islamic Republic of...                         │
│         96 │ 25-34 years old    │ Employed, full-time                                                                          │ Greece                                               │
│         97 │ 25-34 years old    │ Employed, full-time                                                                          │ India                                                │
│         98 │ 55-64 years old    │ Employed, full-time                                                                          │ United States of America                             │
│         99 │ 18-24 years old    │ Employed, full-time                                                                          │ Italy                                                │
│        100 │ Under 18 years old │ Student, full-time                                                                           │ Israel                                               │
│         81 │ 35-44 years old    │ Employed, full-time                                                                          │ Germany                                              │
│         82 │ 25-34 years old    │ Employed, full-time;Independent contractor, freelancer, or self-employed                     │ Finland                                              │
│         83 │ 35-44 years old    │ Employed, full-time;Independent contractor, freelancer, or self-employed                     │ Slovakia                                             │
│         84 │ 25-34 years old    │ Employed, full-time                                                                          │ Poland                                               │
│         85 │ 25-34 years old    │ Not employed, but looking for work                                                           │ United States of America                             │
│         86 │ 18-24 years old    │ Not employed, and not looking for work                                                       │ United States of America                             │
│         87 │ 45-54 years old    │ Employed, full-time                                                                          │ United Kingdom of Great Britain and Northern Ireland │
│         88 │ 25-34 years old    │ Employed, full-time                                                                          │ Germany                                              │
│         89 │ 35-44 years old    │ Independent contractor, freelancer, or self-employed                                         │ United Kingdom of Great Britain and Northern Ireland │
│         90 │ 25-34 years old    │ Employed, full-time                                                                          │ India                                                │
│         71 │ 35-44 years old    │ Not employed, and not looking for work                                                       │ Australia                                            │
│         72 │ 45-54 years old    │ Employed, full-time                                                                          │ Sweden                                               │
│         73 │ 18-24 years old    │ Employed, full-time;Student, full-time;Independent contractor, freelancer, or self-employe…  │ Pakistan                                             │
│         74 │ 35-44 years old    │ Employed, full-time                                                                          │ Czech Republic                                       │
│         75 │ 55-64 years old    │ Employed, full-time                                                                          │ Switzerland                                          │
├────────────┴────────────────────┴──────────────────────────────────────────────────────────────────────────────────────────────┴──────────────────────────────────────────────────────┤
│ 25 rows                                                                                                                                                                     4 columns │
└───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

## In Conclusion

I've been very slow to embrace Iceberg adoption due to too many system/org/vendor/functionality tradeoffs. Adoption has seemed vendor-locked at the catalog level, getting data in/out of a lake has seemed overly complex or bloated, and multi-engine or multi-lang support has seemed fragmented at best.

This exploration has changed that for me.

#### And I think it's finally time.

