---
title: 'Zero-Infrastructure Iceberg in 5 Minutes'
description: 'Zero-Infrastructure Iceberg in 5 Minutes'
pubDate: 'Apr 10 2025'
heroImage: '/img/blog/zero-infrastructure-iceberg.png'
---

Operating an Iceberg-based data lake shouldn't be so hard. And (finally) it is not.


But wait... don't you need Spark or Some Java Thing™? How about background compaction? Manifest rewrites? Sourcing a new vendor only to use their (mostly-closed) catalog?

...***anything?!?***


### Nope.

Thanks to [S3 Tables](https://aws.amazon.com/s3/features/tables/), [DuckDB](https://duckdb.org/), and [Arrow](https://arrow.apache.org/) (behind the scenes) you can build a low-cost, low-touch data lake on Iceberg.

**In five minutes, with no infrastructure, using a single vendor (AWS) that you probably already have.**


And it's dead-simple.

#### Step 1: Create a S3 Table bucket, namespace, and table.

This example uses `awscli` create a `lake` bucket, a `stack overflow` namespace, and a `developer survey` table, to then play around with [Stack Overflow's 2024 Developer Survey](https://survey.stackoverflow.co/2024/) data.

**The table bucket:**

```
    aws s3tables create-table-bucket --cli-input-json '{"name": "jakes-lake"}'


    {
        "arn": "arn:aws:s3tables:us-east-1:$YOUR-ACCOUNT:bucket/jakes-lake"
    }

```

**The namespace:**

```
    aws s3tables create-namespace --cli-input-json '{"tableBucketARN": "arn:aws:s3tables:us-east-1:$YOUR-ACCOUNT$:bucket/jakes-lake","namespace": ["stack_overflow"]}'

    {
        "tableBucketARN": "arn:aws:s3tables:us-east-1:$YOUR-ACCOUNT$:bucket/jakes-lake",
        "namespace": [
            "stack_overflow"
        ]
    }
```

**The table:**

```
    aws s3tables create-table --cli-input-json '{ "tableBucketARN": "arn:aws:s3tables:us-east-1:$YOUR_ACCOUNT$:bucket/jakes-lake", "namespace": "stack_overflow", "name": "survey_results", "format": "ICEBERG", "metadata": { "iceberg": { "schema": { "fields": [ { "name": "ResponseId", "type": "int" }, { "name": "MainBranch", "type": "string" }, { "name": "Age", "type": "string" }, { "name": "Employment", "type": "string" }, { "name": "RemoteWork", "type": "string" }, { "name": "Check", "type": "string" }, { "name": "CodingActivities", "type": "string" }, { "name": "EdLevel", "type": "string" }, { "name": "LearnCode", "type": "string" }, { "name": "LearnCodeOnline", "type": "string" }, { "name": "TechDoc", "type": "string" }, { "name": "YearsCode", "type": "string" }, { "name": "YearsCodePro", "type": "string" }, { "name": "DevType", "type": "string" }, { "name": "OrgSize", "type": "string" }, { "name": "PurchaseInfluence", "type": "string" }, { "name": "BuyNewTool", "type": "string" }, { "name": "BuildvsBuy", "type": "string" }, { "name": "TechEndorse", "type": "string" }, { "name": "Country", "type": "string" }, { "name": "Currency", "type": "string" }, { "name": "CompTotal", "type": "string" }, { "name": "LanguageHaveWorkedWith", "type": "string" }, { "name": "LanguageWantToWorkWith", "type": "string" }, { "name": "LanguageAdmired", "type": "string" }, { "name": "DatabaseHaveWorkedWith", "type": "string" }, { "name": "DatabaseWantToWorkWith", "type": "string" }, { "name": "DatabaseAdmired", "type": "string" }, { "name": "PlatformHaveWorkedWith", "type": "string" }, { "name": "PlatformWantToWorkWith", "type": "string" }, { "name": "PlatformAdmired", "type": "string" }, { "name": "WebframeHaveWorkedWith", "type": "string" }, { "name": "WebframeWantToWorkWith", "type": "string" }, { "name": "WebframeAdmired", "type": "string" }, { "name": "EmbeddedHaveWorkedWith", "type": "string" }, { "name": "EmbeddedWantToWorkWith", "type": "string" }, { "name": "EmbeddedAdmired", "type": "string" }, { "name": "MiscTechHaveWorkedWith", "type": "string" }, { "name": "MiscTechWantToWorkWith", "type": "string" }, { "name": "MiscTechAdmired", "type": "string" }, { "name": "ToolsTechHaveWorkedWith", "type": "string" }, { "name": "ToolsTechWantToWorkWith", "type": "string" }, { "name": "ToolsTechAdmired", "type": "string" }, { "name": "NEWCollabToolsHaveWorkedWith", "type": "string" }, { "name": "NEWCollabToolsWantToWorkWith", "type": "string" }, { "name": "NEWCollabToolsAdmired", "type": "string" }, { "name": "OpSysPersonal", "type": "string" }, { "name": "OpSysProfessional", "type": "string" }, { "name": "OfficeStackAsyncHaveWorkedWith", "type": "string" }, { "name": "OfficeStackAsyncWantToWorkWith", "type": "string" }, { "name": "OfficeStackAsyncAdmired", "type": "string" }, { "name": "OfficeStackSyncHaveWorkedWith", "type": "string" }, { "name": "OfficeStackSyncWantToWorkWith", "type": "string" }, { "name": "OfficeStackSyncAdmired", "type": "string" }, { "name": "AISearchDevHaveWorkedWith", "type": "string" }, { "name": "AISearchDevWantToWorkWith", "type": "string" }, { "name": "AISearchDevAdmired", "type": "string" }, { "name": "NEWSOSites", "type": "string" }, { "name": "SOVisitFreq", "type": "string" }, { "name": "SOAccount", "type": "string" }, { "name": "SOPartFreq", "type": "string" }, { "name": "SOHow", "type": "string" }, { "name": "SOComm", "type": "string" }, { "name": "AISelect", "type": "string" }, { "name": "AISent", "type": "string" }, { "name": "AIBen", "type": "string" }, { "name": "AIAcc", "type": "string" }, { "name": "AIComplex", "type": "string" }, { "name": "AIToolCurrently", "type": "string" }, { "name": "AIToolInterested", "type": "string" }, { "name": "AIToolNot", "type": "string" }, { "name": "AINextMuch", "type": "string" }, { "name": "AINextNo", "type": "string" }, { "name": "AINextMore", "type": "string" }, { "name": "AINextLess", "type": "string" }, { "name": "AIThreat", "type": "string" }, { "name": "AIEthics", "type": "string" }, { "name": "AIChallenges", "type": "string" }, { "name": "TBranch", "type": "string" }, { "name": "ICorPM", "type": "string" }, { "name": "WorkExp", "type": "string" }, { "name": "Knowledge_1", "type": "string" }, { "name": "Knowledge_2", "type": "string" }, { "name": "Knowledge_3", "type": "string" }, { "name": "Knowledge_4", "type": "string" }, { "name": "Knowledge_5", "type": "string" }, { "name": "Knowledge_6", "type": "string" }, { "name": "Knowledge_7", "type": "string" }, { "name": "Knowledge_8", "type": "string" }, { "name": "Knowledge_9", "type": "string" }, { "name": "Frequency_1", "type": "string" }, { "name": "Frequency_2", "type": "string" }, { "name": "Frequency_3", "type": "string" }, { "name": "TimeSearching", "type": "string" }, { "name": "TimeAnswering", "type": "string" }, { "name": "Frustration", "type": "string" }, { "name": "ProfessionalTech", "type": "string" }, { "name": "ProfessionalCloud", "type": "string" }, { "name": "ProfessionalQuestion", "type": "string" }, { "name": "Industry", "type": "string" }, { "name": "JobSatPoints_1", "type": "string" }, { "name": "JobSatPoints_4", "type": "string" }, { "name": "JobSatPoints_5", "type": "string" }, { "name": "JobSatPoints_6", "type": "string" }, { "name": "JobSatPoints_7", "type": "string" }, { "name": "JobSatPoints_8", "type": "string" }, { "name": "JobSatPoints_9", "type": "string" }, { "name": "JobSatPoints_10", "type": "string" }, { "name": "JobSatPoints_11", "type": "string" }, { "name": "SurveyLength", "type": "string" }, { "name": "SurveyEase", "type": "string" }, { "name": "ConvertedCompYearly", "type": "string" }, { "name": "JobSat", "type": "string" } ] } } } }'


    {
    "tableARN": "arn:aws:s3tables:us-east-1:$YOUR_ACCOUNT:bucket/jakes-lake/table/de77f4ce-2b2a-4377-bedd-20668da01936",
    "versionToken": "042c378a97092aabf2f6"
}
```


### Step 2: Grab the data with DuckDB and load it into the S3.... table.

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