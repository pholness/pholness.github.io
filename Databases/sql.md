---
layout: default
title: Useful SQL
parent: Databases
nav_order: 1
---
# Useful SQL Snippets

## Date format

```
alter session set nls_date_format='DD/MM/YYYY HH24:MI:SS';
```

## Get current SCN

```
select dbms_flashback.get_system_change_number from dual;
```

## Explain plan 

```
explain plan for <sql statement>

Explained.

select plan_table_output from table(dbms_xplan.display());
```