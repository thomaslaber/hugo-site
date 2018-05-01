+++
date = "2018-04-29"
title = "SQL Server for Data Scientists"
+++

SQL is not the sexiest language on the block and many/most data scientists I know prefer to stick to R and/or Python. Some common complains I hear about SQL are:

- It is hard to read and as a consequence
- large SQL statements are hard to debug.
- Version control with databases often requires additional tooling to work with git.

However, since SQL was designed to work with and process data, I think that every data scientist should have at least a basic understanding of how to write queries efficiently. While I am no expert in SQL myself, maybe you still find something interesting that can help you in your work.

There are a lot of SQL flavors out there, but in this post I will focus on Microsoft's T-SQL. It should be easy enough to translate T-SQL to your SQL flavor of choice.

## Setup
Before we get started, I would recommend you download SQL Server Management Studio or VS Code with the MSSQL extension. To follow along, you will need access to a SQL Server instance. If you don't you can either:

- [Install SQL Server 2017 locally on either Windows or Linux](https://www.microsoft.com/en-us/sql-server/sql-server-downloads-free-trial)
- [Install SQL Server 2017 using Docker](https://hub.docker.com/r/microsoft/mssql-server-linux/) or
- [spin up an SQL Server instance in the cloud using Azure](https://azure.microsoft.com/en-us/free/sql-on-azure/) or AWS

Once you are connected to a SQL Server instance, you are ready to go on:

## Creating a database and a table
First, let's create a database and a table:
```
CREATE DATABASE test_db

-- select database to use for current connection:
USE test_db

CREATE TABLE test_table
( id INT NOT NULL,
  last_name VARCHAR(50) NOT NULL,
  first_name VARCHAR(50),
  age INT
);
```
You can access the online help for a specific command by highlighting a command and pressing F1.

## Database schemas
A schema is basically a namespace in a database and is also closely tied to permission settings. The default schema is `[dbo]` and is specified immediately after the database name:
```
SELECT *
FROM [test_db].[dbo].[test_table]
```

This is the fully qualified name of a table, starting with the database [test_db], followed by the schema [dbo] and the table [test_table] at the very end. Most of the time you will probably not bother to write the fully qualified name, especially if you use the default [dbo] schema, since no explicit schema implies [dbo].

## Loading data
Now wer are going to add some data to our table:
```
INSERT INTO dbo.test_table (id, last_name, first_name, age)  
VALUES ('1', 'Phu', 'Winnie', NULL),  
       ('2', 'Pan', 'Peter', '17')
```

Importing data from one SQL Server into another is also easy. You can either write a query:
```
INSERT INTO [your_schema_here].[your_table_here]
   SELECT *
   FROM [source_server].[source_database].[source_schema].[your_table_here]
```
or use 'SQL Server Import and Export Data' tool, which is pretty much self-explanatory to use.

## Manipulating data
There are only a couple of commands that cover probably 99% of all cases you are going to encounter:

```
-- * selects all columns
SELECT *
FROM [test_db].[dbo].[test_table]

-- select specific columns by name
SELECT 
    id
    ,last_name
    ,first_name
    ,age
FROM [test_db].[dbo].[test_table]
```

> Note: SQL is NOT case sensitive, so LAST_NAME, last_name and Last_Name and every other variation is seen as the same variable.

Some names are reserved key words and need to be put in square brackets if you want to use them as table or column names, like so:
`[test_db].[dbo].[test_table]`.

The following commands always need to be specified in this order: `WHERE` > `GROUP BY` > `ORDER BY`
```
SELECT *
FROM test_table
WHERE id = '1' AND id = '2'
GROUP BY id, last_name, first_name
ORDER BY id, last_name
```

## Temporary tables and Common Table Expressions
There are basically two ways to work with temporary data in SQL:

1. Create temporary tables or
2. use common table expressions, aka CTEs

Creating a temporary table is easy, all you need to do is add `into #temp_table_name` to your select statement like so:
```
-- without sub-select:
SELECT *
INTO #temp_table
FROM dbo.test_table

-- with sub-select:
SELECT *
INTO #temp_table 
FROM (
    SELECT *
    FROM dbo.test_table
)
```
Personally, I prefer the option with sub-select as it is easier to see that data is written to a temporary table. As you can see, all you need to do to create a temporary table is prefix your table with one hashtag `#`. This temporary table is available as long as the current connection from the current user is open. You can also create a temp table using two hashtags `##`. This temp table is available to any user by any connection and is only deleted if all open connections are closed.

Usually, you do not open and close connections very often (at least I do not) so you might want to drop your temp table on each re-run. You can do that like so:
```
-- drop table if it exists:
if object_id('tempdb..#temp_table') is not null 
	DROP table #temp_table

SELECT *
INTO #temp_table
FROM dbo.test_table
```

With a common-table expression you do not need to bother with cleaning up temp tables as a CTE is only available at run time of its query. So in a sense, a CTE is a temp table that is not persisted after the query finishes and thus you do not need to drop it. A major benefit in my opinion is also that CTEs allow you to structure your query in a more readable manner:
```
WITH name_of_CTE_1 
AS (
    SELECT *
    FROM dbo.test_table
), 
name_of_CTE_2 as (
    SELECT *
    FROM dbo.test_table
) 

SELECT 
    a.id
    a.last_name
    b.first_name
FROM name_of_CTE_1 a
LEFT JOIN name_of_CTE_2 b on a.id = b.id
```

## UPDATE


## Machine Learning Services


## Further stuff
- GO operator
