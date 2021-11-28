---
layout: default
title: Useful SQL
parent: Databases
nav_order: 2
---
# Useful SQL Snippets

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

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