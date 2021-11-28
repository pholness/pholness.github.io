---
layout: default
title: RMAN Tips
parent: Databases
nav_order: 6
---

# RMAN Tips

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Check config 
RMAN> show all;

As a rule, ensure control file autobackup is on. This can save your bacon if you lose all copies of the controlfile.

```
CONFIGURE CONTROLFILE AUTOBACKUP ON;
CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '%F'; # default
```

default to using compression

```
CONFIGURE DEVICE TYPE DISK BACKUP TYPE TO COMPRESSED BACKUPSET; 
```

## Full backup 
```
backup incremental level 0 database plus archivelog
```

## Incremental 

```
backup incremental level 1 database plus archivelog
```

## Compressed 
```
backup as compressed backupset incremental level 0 database plus archivelog
backup as compressed backupset incremental level 1 database plus archivelog
```


## Restore 
Make sure database is mounted
```
RMAN> restore database;
RMAN> recover database;
```


## Fix advisor 
```
RMAN> LIST FAILURE
RMAN> ADVISE FAILURE
RMAN> REPAIR FAILURE
```

e.g:
```
RMAN> list failure;

using target database control file instead of recovery catalog
List of Database Failures
=========================

Failure ID Priority Status    Time Detected Summary
---------- -------- --------- ------------- -------
62         HIGH     OPEN      28-FEB-12     One or more non-system datafiles are missing
660        HIGH     OPEN      28-FEB-12     One or more non-system datafiles need media recovery

RMAN> advise failure;

List of Database Failures
=========================

Failure ID Priority Status    Time Detected Summary
---------- -------- --------- ------------- -------
62         HIGH     OPEN      28-FEB-12     One or more non-system datafiles are missing
660        HIGH     OPEN      28-FEB-12     One or more non-system datafiles need media recovery

analyzing automatic repair options; this may take some time
allocated channel: ORA_DISK_1
channel ORA_DISK_1: SID=28 device type=DISK
analyzing automatic repair options complete

Mandatory Manual Actions
========================
no manual actions available

Optional Manual Actions
=======================
1. If file +DATA/orcl/datafile/undotbs1.258.774110591 was unintentionally renamed or moved, restore it
2. If you restored the wrong version of data file +DATA/orcl/datafile/undotbs1.258.774110591, then replace it with the correct one

Automated Repair Options
========================
Option Repair Description
------ ------------------
1      Restore and recover datafile 3  
  Strategy: The repair includes complete media recovery with no data loss
  Repair script: /u01/app/oracle/diag/rdbms/orcl/PETER/hm/reco_2853012380.hm

RMAN> repair failure;

Strategy: The repair includes complete media recovery with no data loss
Repair script: /u01/app/oracle/diag/rdbms/orcl/PETER/hm/reco_2853012380.hm

contents of repair script:
   # restore and recover datafile
   restore datafile 3;
   recover datafile 3;
   sql 'alter database datafile 3 online';

Do you really want to execute the above repair (enter YES or NO)? y
executing repair script

Starting restore at 28-FEB-12
using channel ORA_DISK_1

channel ORA_DISK_1: starting datafile backup set restore
channel ORA_DISK_1: specifying datafile(s) to restore from backup set
channel ORA_DISK_1: restoring datafile 00003 to +DATA/orcl/datafile/undotbs1.258.774110591
channel ORA_DISK_1: reading from backup piece +FRA/orcl/backupset/2012_02_28/nnndn0_tag20120228t105332_0.266.776429613
channel ORA_DISK_1: piece handle=+FRA/orcl/backupset/2012_02_28/nnndn0_tag20120228t105332_0.266.776429613 tag=TAG20120228T105332
channel ORA_DISK_1: restored backup piece 1
channel ORA_DISK_1: restore complete, elapsed time: 00:00:07
Finished restore at 28-FEB-12

Starting recover at 28-FEB-12
using channel ORA_DISK_1
channel ORA_DISK_1: starting incremental datafile backup set restore
channel ORA_DISK_1: specifying datafile(s) to restore from backup set
destination for restore of datafile 00003: +DATA/orcl/datafile/undotbs1.258.776433735
channel ORA_DISK_1: reading from backup piece +FRA/orcl/backupset/2012_02_28/nnndn1_tag20120228t114006_0.260.776432407
channel ORA_DISK_1: piece handle=+FRA/orcl/backupset/2012_02_28/nnndn1_tag20120228t114006_0.260.776432407 tag=TAG20120228T114006
channel ORA_DISK_1: restored backup piece 1
channel ORA_DISK_1: restore complete, elapsed time: 00:00:01

starting media recovery
media recovery complete, elapsed time: 00:00:00

Finished recover at 28-FEB-12

sql statement: alter database datafile 3 online
repair failure complete

Do you want to open the database (enter YES or NO)? y
database opened

RMAN> 
```

## Simple restore of missing non-system datafiles 

Do the above restore / recover

## Point in time recovery 

If you know you are going to do something dangerous, make sure you have backups, then switch log files, get the SCN, and you can always roll back. Here's how:

```
SQL> select dbms_flashback.get_system_change_number from dual;
SQL> alter system switch logfile;
```


To recover to this point in time:

```
$ rman target /
RMAN> shutdown immediate;
RMAN> startup mount;
RMAN> run {
 set until SCN <scn number from above>;
restore database;
recover database;
}
RMAN> alter database open resetlogs;
```

## Restore of single missing control file 

## Restore of all control files 
If you haven't got control file autobackup on, your life just got harder, but it is not impossible.
The control file will be included in backups anyway. You can see this in a healthy database by issuing list backup:

```
RMAN> list backup;
....

BS Key  Type LV Size       Device Type Elapsed Time Completion Time
------- ---- -- ---------- ----------- ------------ ---------------
27      Incr 1  1.05M      DISK        00:00:00     28-FEB-12      
		BP Key: 27   Status: AVAILABLE  Compressed: YES  Tag: TAG20120228T140216
		Piece Name: +FRA/orcl/backupset/2012_02_28/ncsnn1_tag20120228t140216_0.277.776440953
  SPFILE Included: Modification time: 28-FEB-12
  SPFILE db_unique_name: ORCL
  Control File Included: Ckp SCN: 1111892      Ckp time: 28-FEB-12

BS Key  Size       Device Type Elapsed Time Completion Time
------- ---------- ----------- ------------ ---------------
28      13.50K     DISK        00:00:00     28-FEB-12      
		BP Key: 28   Status: AVAILABLE  Compressed: YES  Tag: TAG20120228T140238
		Piece Name: +FRA/orcl/backupset/2012_02_28/annnf0_tag20120228t140238_0.279.776440959

  List of Archived Logs in backup set 28
  Thrd Seq     Low SCN    Low Time  Next SCN   Next Time
  ---- ------- ---------- --------- ---------- ---------
  1    9       1111873    28-FEB-12 1111900    28-FEB-12
```

However, once the control files are gone, you can no longer list the backups.
You can probably work out the latest backup by looking at the filenames, and timestamps. The control file is generally backed up in the last file in a backup set.

List / advise / repair failure will not help you here.

You need to restore the control file.
You can do this as follows:

```
RMAN>restore controlfile from autobackup;
```

(Make sure the database name / DBID is set correctly using your DBID and the format specified when backups taken - normally %F)
```
RMAN> SET DBID 320066378;
RMAN> RUN {
	SET CONTROLFILE AUTOBACKUP FORMAT 
		  FOR DEVICE TYPE DISK TO 'autobackup_format';
	RESTORE CONTROLFILE FROM AUTOBACKUP;
	}
```

If autobackups are not on, this will fail. 

You can look through the backupset files and find the most recent database backup, and try this:

```
RMAN> restore controlfile from '+FRA/orcl/backupset/2012_02_28/ncsnn1_tag20120228t140216_0.277.776440953';
```

Once you have the control files back, still need to restore and recover to get all the logs in sync.

```  
RMAN> restore database;
RMAN> recover database;
RMAN> alter database open resetlogs;
```
* Note, after open resetlogs, need to delete old archive logs.


# Queries on v$ views for performance monitoring 

## Past and current RMAN jobs 
```
COL STATUS FORMAT a9
COL hrs FORMAT 999.99
COL start_time format a18
COL end_time format a18
set lines 120
set pages 40

SELECT SESSION_KEY, INPUT_TYPE, STATUS,
TO_CHAR(START_TIME,'mm/dd/yy hh24:mi:ss') start_time,
TO_CHAR(END_TIME,'mm/dd/yy hh24:mi:ss') end_time,
ELAPSED_SECONDS/3600 hrs
FROM V$RMAN_BACKUP_JOB_DETAILS
ORDER BY SESSION_KEY;
```


## Data Rate 
```
COL in_per_sec FORMAT a10
COL out_per_sec FORMAT a11
COL TIME_TAKEN_DISPLAY FORMAT a10

SELECT SESSION_KEY,
OPTIMIZED,
COMPRESSION_RATIO,
INPUT_BYTES_PER_SEC_DISPLAY in_per_sec,
OUTPUT_BYTES_PER_SEC_DISPLAY out_per_sec,
TIME_TAKEN_DISPLAY
FROM V$RMAN_BACKUP_JOB_DETAILS
ORDER BY SESSION_KEY;
```


## Data Size 
```
COL in_size FORMAT a10
COL out_size FORMAT a10

SELECT SESSION_KEY,
INPUT_TYPE,
COMPRESSION_RATIO,
INPUT_BYTES_DISPLAY in_size,
OUTPUT_BYTES_DISPLAY out_size
FROM V$RMAN_BACKUP_JOB_DETAILS
ORDER BY SESSION_KEY;
```


## Combined Rate and Size 
```
COL in_per_sec FORMAT a10
COL out_per_sec FORMAT a11
COL in_size FORMAT a10
COL out_size FORMAT a10
COL TIME_TAKEN_DISPLAY FORMAT a10
COL input_type FORMAT a12
set lines 140
set pages 40

SELECT SESSION_KEY,
INPUT_TYPE,
OPTIMIZED,
COMPRESSION_RATIO,
INPUT_BYTES_PER_SEC_DISPLAY in_per_sec,
OUTPUT_BYTES_PER_SEC_DISPLAY out_per_sec,
TIME_TAKEN_DISPLAY,
INPUT_BYTES_DISPLAY in_size,
OUTPUT_BYTES_DISPLAY out_size
FROM V$RMAN_BACKUP_JOB_DETAILS
ORDER BY SESSION_KEY;

```
## Long Running RMAN jobs - Monitor progress 

```
col opname format a20
set lines 140
alter session set nls_date_format='dd/mm/yy hh24:mi:ss';

SELECT SID, OPNAME, CONTEXT, SOFAR, TOTALWORK,
ROUND(SOFAR/TOTALWORK*100,2) "%_COMPLETE",
sysdate + TIME_REMAINING/3600/24 END_AT
FROM V$SESSION_LONGOPS
WHERE OPNAME LIKE 'RMAN%'
AND OPNAME NOT LIKE '%aggregate%'
AND TOTALWORK != 0
AND SOFAR <> TOTALWORK
ORDER BY 7;
```


## List of event names for SBT Channels 
```
SELECT NAME
FROM V$EVENT_NAME
WHERE NAME LIKE '%MML%';
```


## Time waiting for media manager - Look for high seconds_in_wait 
```
COLUMN EVENT FORMAT a50
COLUMN SECONDS_IN_WAIT FORMAT 999
COLUMN STATE FORMAT a15
COLUMN CLIENT_INFO FORMAT a30
COLUMN SPID FORMAT a8

SELECT p.SPID, sw.EVENT, sw.SECONDS_IN_WAIT AS SEC_WAIT,
sw.STATE, CLIENT_INFO
FROM V$SESSION_WAIT sw, V$SESSION s, V$PROCESS p
WHERE sw.EVENT LIKE '%MML%'
AND s.SID=sw.SID
AND s.PADDR=p.ADDR;
```


