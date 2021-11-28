---
layout: default
title: Log Miner
parent: Databases
nav_order: 5
---

# Log Miner Tips 

LogMiner lets you look back through old archive logs, to see who did what, when.

You can extract the original SQL and data, and the corresponding undo SQL.

You can pull old backed up archive logs out of backup to mine, you can even study the logs on a completely different database, so long as you have the data dictionary from the source database.

[Link to Oracle Docs on Log Miner](http://docs.oracle.com/cd/E11882_01/server.112/e22490/logminer.htm#i1015913)

## To pull out the log files from a backup

```
RMAN> list backup summary;

```

Find the backup set you are interested in

```
RMAN> list backupset xxxxx

List of Backup Sets
===================


BS Key  Size       Device Type Elapsed Time Completion Time
------- ---------- ----------- ------------ ---------------
6760    2.04G      DISK        00:31:33     06-MAR-2012
        BP Key: 6760   Status: AVAILABLE  Compressed: YES  Tag: TAG20120306T083603
        Piece Name: /u999/oraback/PWCMPUBL/arc/PWCMPUBL_arc1_20120306_jbn567bm_1_1_6763

  List of Archived Logs in backup set 6760
  Thrd Seq     Low SCN    Low Time    Next SCN   Next Time
  ---- ------- ---------- ----------- ---------- ---------
  1    13752   479003136  06-MAR-2012 479072703  06-MAR-2012
  1    13753   479072703  06-MAR-2012 479078468  06-MAR-2012
  1    13754   479078468  06-MAR-2012 479083233  06-MAR-2012
  1    13755   479083233  06-MAR-2012 479088067  06-MAR-2012
  1    13756   479088067  06-MAR-2012 479093288  06-MAR-2012
  1    13757   479093288  06-MAR-2012 479098752  06-MAR-2012
  1    13758   479098752  06-MAR-2012 479104341  06-MAR-2012

RMAN> RUN
{
RESTORE ARCHIVELOG FROM SEQUENCE 13753 UNTIL SEQUENCE 13755;
}
```

Out of RMAN, move these logs somewhere safe, and then cross check to stop RMAN looking for them. This stops scheduled backup jobs deleting them again.

In sqlplus

```
EXECUTE DBMS_LOGMNR.ADD_LOGFILE(LOGFILENAME => '/u201/temp/o1_mf_1_13989_7oj3w7hm_.arc', OPTIONS => DBMS_LOGMNR.NEW);
EXECUTE DBMS_LOGMNR.ADD_LOGFILE(LOGFILENAME => '/u201/temp/o1_mf_1_13990_7oj3w7k4_.arc', OPTIONS => DBMS_LOGMNR.ADDFILE);
EXECUTE DBMS_LOGMNR.ADD_LOGFILE(LOGFILENAME => '/u201/temp/o1_mf_1_13991_7oj43dnx_.arc', OPTIONS => DBMS_LOGMNR.ADDFILE);
EXECUTE DBMS_LOGMNR.ADD_LOGFILE(LOGFILENAME => '/u201/temp/o1_mf_1_13992_7oj400dh_.arc', OPTIONS => DBMS_LOGMNR.ADDFILE);
EXECUTE DBMS_LOGMNR.ADD_LOGFILE(LOGFILENAME => '/u201/temp/o1_mf_1_13993_7oj40vjv_.arc', OPTIONS => DBMS_LOGMNR.ADDFILE);
EXECUTE DBMS_LOGMNR.ADD_LOGFILE(LOGFILENAME => '/u201/temp/o1_mf_1_13994_7oj3w7nj_.arc', OPTIONS => DBMS_LOGMNR.ADDFILE);
EXECUTE DBMS_LOGMNR.ADD_LOGFILE(LOGFILENAME => '/u201/temp/o1_mf_1_13995_7oj41p6g_.arc', OPTIONS => DBMS_LOGMNR.ADDFILE);
EXECUTE DBMS_LOGMNR.ADD_LOGFILE(LOGFILENAME => '/u201/temp/o1_mf_1_13996_7oj3w7qg_.arc', OPTIONS => DBMS_LOGMNR.ADDFILE);
EXECUTE DBMS_LOGMNR.ADD_LOGFILE(LOGFILENAME => '/u201/temp/o1_mf_1_13997_7oj400fx_.arc', OPTIONS => DBMS_LOGMNR.ADDFILE);
EXECUTE DBMS_LOGMNR.ADD_LOGFILE(LOGFILENAME => '/u201/temp/o1_mf_1_13998_7oj3w7qr_.arc', OPTIONS => DBMS_LOGMNR.ADDFILE);
EXECUTE DBMS_LOGMNR.ADD_LOGFILE(LOGFILENAME => '/u201/temp/o1_mf_1_13999_7oj3twd0_.arc', OPTIONS => DBMS_LOGMNR.ADDFILE);

EXECUTE DBMS_LOGMNR.START_LOGMNR(OPTIONS => DBMS_LOGMNR.DICT_FROM_ONLINE_CATALOG);

SQL> select min(timestamp), max(timestamp) from v$logmnr_contents;

MIN(TIMESTAMP)             MAX(TIMESTAMP)
-------------------------- --------------------------
06-MAR-2012 05:19:05       06-MAR-2012 05:30:59

 col OPERATION format a15
 col TABLE_NAME format a50
 select OPERATION,TABLE_NAME,count(*) num from  v$logmnr_contents
    group by OPERATION,TABLE_NAME
    order by num desc
   /

OPERATION       TABLE_NAME                  NUM
--------------- -------------------- ----------
INSERT          INJSTATS_ALLYEAR         440050
INTERNAL                                  75170
COMMIT                                    12693
START                                     12693
UNSUPPORTED     SEG$                       1252
UPDATE          COL_USAGE$                   50
UNSUPPORTED                                  49
UPDATE          MON_MODS$                    10
UNSUPPORTED     SMON_SCN_TIME                 3
UPDATE          SYS_FBA_BARRIERSCN            2
UPDATE          HTMLCONVERSIONS               2

11 rows selected.
```
