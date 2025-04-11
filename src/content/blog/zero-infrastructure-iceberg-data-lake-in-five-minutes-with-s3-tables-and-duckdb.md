---
title: 'Zero-Infrastructure Iceberg in 5 Minutes'
description: 'Zero-Infrastructure Iceberg in 5 Minutes'
pubDate: 'Apr 10 2025'
heroImage: '/img/blog/zero-infrastructure-iceberg.png'
---

### Operating an Iceberg-based data lake shouldn't be so hard.

### And (finally) it is not.


But wait... don't you need Spark or Some Java Thing™? How about background compaction? Manifest rewrites? Sourcing a new vendor only to use their (mostly-closed) catalog?

...***anything?!?***


### Nope.

Thanks to [S3 Tables](https://aws.amazon.com/s3/features/tables/), [DuckDB](https://duckdb.org/), and [Arrow](https://arrow.apache.org/) (behind the scenes) you can build a low-cost, low-touch data lake on Iceberg.

**In five minutes, with no infrastructure, using a single vendor (AWS) that you probably already have.**


And it's dead-simple.

#### Step 1: Create a S3 Table bucket, namespace, and table.

Using `awscli` we'll create a `lake` bucket, a `stack overflow` namespace, and a `developer survey`

**The table**

```
    aws s3tables create-table-bucket --name jakes-lake


    {
        "arn": "arn:aws:s3tables:us-east-1:$YOUR-ACCOUNT:bucket/jakes-lake"
    }

```

**The namespace**

```
    aws s3tables create-namespace --table-bucket-arn arn:aws:s3tables:us-east-1:$YOUR-ACCOUNT$:bucket/jakes-lake --namespace stack_overflow


    {
        "tableBucketARN": "arn:aws:s3tables:us-east-1:$YOUR-ACCOUNT$:bucket/jakes-lake",
        "namespace": [
            "stack_overflow"
        ]
    }
```

**The table**

```
    aws s3tables create-table

```


### Step 2: Grab some data with DuckDB and load it into the S3.... table.

Using `python` with two dependencies: `pyiceberg` and `duckdb`:

```

```

### Step 3: Attach the S3 Table Catalog with DuckDB and query your data.

Using `python`:

```

```

Using `duckdb cli`:

```

```


### That's it. It's really that easy.

But there's still some friction.

**Not all iceberg data types are cleanly mapped to DuckDB types (yet)...**

For example if you create a `uuid` column in Iceberg and try to read it with DuckDB, you will get this:

```
BADNESS HERE
```

**Bouncing between `awscli`, `pyiceberg`, and `duckdb` could be streamlined.**

For example... what if DuckDB could directly create a table bucket?

And what if a `create schema` created a table bucket namespace?

And `create table`.... (you get the idea)


**Table Bucket ACL's...**

Are definitely nice to have in the same policy management tooling as the rest of your infrastructure. Especially since table-level acl's exist and are easier that mentally mapping directories and paths to iam permissions.

But important governance mechanisms like row-level policies? Or 

**Views, oh sweet Views**

Don't yet exist in the implementation. And materialized views are [just not in the spec](https://github.com/apache/iceberg/issues/10043).

But you can create [DuckDB views](https://duckdb.org/docs/stable/sql/statements/create_view.html) using an attached [s3 table bucket](https://duckdb.org/2025/03/14/preview-amazon-s3-tables.html#reading-amazon-s3-tables-with-duckdb). So that's pretty damn cool.


**I still have no idea if retention and lifecycle policies exist**

If so, it would be pretty neat.