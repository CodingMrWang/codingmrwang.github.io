---
layout:     post
title:      Hive operations
subtitle:   Create, Delete, Alter
date:       2018-08-06
author:     CodingMrWang
header-img: img/post-bg-swift2.jpg
catalog: true
tags:
    - Hive
---


> This post is first created by [CodingMrWang](http://codingmrwang.github.io), 作者 [@Zexian Wang](http://github.com/codingmrwang) ,please keep the original link if you want to repost it.

Hive

> Apache Hive is a data warehouse software project built on top of Apache Hadoop for providing data summarization, query and analysis. Hive gives an SQL-like interface to query data stored in various databases and file systems that integrate with Hadoop. 

Create:

```
CREATE TABLE content.post_dict (
  post_id	STRING COMMENT 'This is primary key',
  url	STRING,
  mp_id	BIGINT,
  title	STRING,
  content	STRING,
  publish_time	STRING,
  update_time	STRING,
) PARTITIONED BY (date STRING) ROW FORMAT SERDE 'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe' STORED AS INPUTFORMAT 'org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat' OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat'
```
It is really important to add format constraint. The default separator is '\u0001' in hive, so if your content contains this separator, it will become a mess in your hive table. I suffered this before when I transfer data from abase to hive.

Delete:

```
DROP TABLE IF EXISTS [db.name].[table.name];
```

Alter:

```
ALTER TABLE [old_db_name.]old_table_name RENAME TO [new_db_name.]new_table_name

ALTER TABLE name ADD COLUMNS (col_spec[, col_spec ...])
ALTER TABLE name DROP [COLUMN] column_name
ALTER TABLE name CHANGE column_name col_spec
ALTER TABLE name REPLACE COLUMNS (col_spec[, col_spec ...])

ALTER TABLE name { ADD [IF NOT EXISTS] | DROP [IF EXISTS] } PARTITION (partition_spec) [PURGE]
ALTER TABLE name RECOVER PARTITIONS
```