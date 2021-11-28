---
layout: default
title: Flashback
parent: Databases
nav_order: 4
---
# Flashback stuff 

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Enable / Disable 
```
alter database flashback on;
alter database flashback off;
```

## Initialization Parameters 

### Setting the location of the flashback 

```
recovery area	db_recovery_file_dest=/oracle/flash_recovery_area
```

### Setting the size of the flashback 
```
recovery area	db_recovery_file_dest_size=2147483648
```

### Setting the retention time for flashback files (in minutes)	-- 2 days 

```
db_flashback_retention_target=2880
```

## Flashback history 

```
select current_scn
from v$database;

select oldest_flashback_scn,
oldest_flashback_time
from gv$flashback_database_log;
```

## Perform a flashback 

```
shutdown immediate;

startup mount exclusive;

-- be sure to substitute your scn
flashback database to scn 19513917;

flashback database to restore point BEF_DAMAGE;

flashback database to timestamp (sysdate-1/24);

flashback database 
to timestamp to_timestamp('2002-11-11 16:00:00', 'yyyy-mm-dd hh24:mi:ss');

alter database open resetlogs;
```

## Restore points 

```
create restore point <restore_point_name>
[as of <timestamp | scn> <timestamp_or_scn_value>]
[preserved];

-- normal restore point
create restore point before_damage;

— guaranteed restore point
create restore point before_damage guarantee flashback database;

drop restore point <restore_point_name>;

select scn, database_incarnation#, guarantee_flashback_database, storage_size, time, name
from gv$restore_point;
```

## Table level flashback 

```
flashback table scott.dept to restore point <restore_point_name>;
flashback table scott.dept to restore point <restore_point_name>
					  *
ERROR at line 1:
ORA-08189: cannot flashback the table because row movement is not enabled

SQL>alter table scott.dept enable row movement;
SQL>flashback table scott.dept to restore point <restore_point_name>;
Flashback complete.
```

## Find the content of the flashback logs 

```
column "Log NO" for 9,999
column "Thread No" for 99
column "Seq No" for 99
column name for a50
column "Size(GB)" for 999,999
column "First Chg No" for 999,999,999,999,999,999
 
alter session set nls_date_format='DD/MM/YYYY HH24:MI:SS'
/
 
select
	 log# as "log no", 
	 thread# as "thread no",
	 sequence# as "seq no",
	 name,
	 bytes/1024/1024/1024 as "size(gb)",
	 first_change# as "first chg no",
	 first_time
from  
   v$flashback_database_logfile
order by first_time
/
```