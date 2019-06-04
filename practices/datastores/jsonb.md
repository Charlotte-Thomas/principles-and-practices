---
layout: page
title: Using JSONB columns responsibly
---

Postgres allows the use of JSON column types that allow us to get the same functionality as you might expect from a document store like MongoDB from a relational store.

It can be tempting to have a table with just a primary key, and a JSONB column and never worry about migrations again. Problems can occur with this approach though which means you need to be careful about how you use this datatype.

## Good reasons for using JSON column types

### Prototyping

If we're not sure what the best way to structure our data is and we are exploring what we might want to store and interact with then JSONB columns are a good idea.

### Infrequently stored data

Where we want to store data that is only relevant to small numbers of rows in a table (like artist's scene preferences or configuration options that only a handful of clients use) then the JSONB column is perfect because it avoids adding lots of columns to a table most of which are null.

This reduces the amount of "noise" there is on the table.

Queries on these mostly empty columns can be poor because the database has to handle all the empty columns when generating the query plan.

## Don't use JSON columns when...

### The column value is present for every row

If a data value occurs in virtually every row it is far better to make it a column and simplify the query pattern and make it easy to understand the kind of data the table contains.

If a column would ever be NOT NULL then it shouldn't be in a JSONB column.

### The datatype would be too generic

Where the equivalent database column type would have a stronger or clearer type then you should use a column.
For example dates and times are poorly represented in JSON as it has no native type for time-based values.

## Problems we've had

These are all actual problems that have occurred rather than theoretial issues.

### Overwriting data

As the handling of the columns is abstracted away by SQL Alchemy and Turbogears transactions (and sometimes Redux stores) it can be easy to create situations where the column value is updated incorrectly with a whole update instead of a path by path update.

In theory there is no difference between updating a path in a JSONB column and a regular column but we are not working at a low enough level to guarantee this.

### Opaque data structure

Where we have allowed free writing and reading of data from the JSONB column then we often fail to have a description of what we consider to be valid content of the column.

Developers struggle to understand what data a column contains and what the type of a value for a given path is. Compared to looking up a column definition in a database table the structure of the data is much more opaque.

This can be managed via use of JSON Schema or a Python dictionary schema library or mapping the JSON to a data object of some kind but the developer will need to have access to both the code and the database to understand what is happening.

### Query performance

While querying paths in a JSONB column is generally fast enough for our purposes certain queries will be much less efficient that the equivalent column-based query.

If you suspect you want to be able to JOIN on a data value then you should be thinking about making it a column.