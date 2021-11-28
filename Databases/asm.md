---
layout: default
title: ASM Related
parent: Databases
nav_order: 3
---
# A selection of ASM related queries

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## ASM Dirs

List directories and sizes for ASM disk groups.

```
col short_path format a40 trunc
set pages 999 lines 132


select substr(full_alias_path,1,greatest(instr(full_alias_path,'/',1,3),instr(full_alias_path,'/',1,2))) short_path , sum(Mb)
from
( select concat(''||gname, sys_connect_by_path(aname, '/')) full_alias_path, Mb
from ( select b.name gname, a.parent_index pindex, a.name aname,
              a.reference_index rindex , a.system_created, a.alias_directory,
              c.type file_type, trunc(c.bytes/1024/1024) Mb
       from v$asm_alias a, v$asm_diskgroup b, v$asm_file c
       where a.group_number = b.group_number
             and a.group_number = c.group_number(+)
             and a.file_number = c.file_number(+)
             and a.file_incarnation = c.incarnation(+)
     )
start with (mod(pindex, power(2, 24))) = 0
            and rindex in
                ( select a.reference_index
                  from v$asm_alias a, v$asm_diskgroup b
                  where a.group_number = b.group_number
                        and (mod(a.parent_index, power(2, 24))) = 0
                )
connect by prior rindex = pindex)
where Mb > 0
group by substr(full_alias_path,1,greatest(instr(full_alias_path,'/',1,3),instr(full_alias_path,'/',1,2)))
UNION
select name||'/Free' , free_mb from v$ASM_DISKGROUP
/
```

## Find out what ASM is doing

During a rebalance for example:

```
select * from gv$asm_operation;

select ao.inst_id, dg.NAME, ao.OPERATION, ao.STATE,ao.SOFAR,ao.EST_WORK,ao.EST_MINUTES
from gv$asm_operation ao, gv$asm_diskgroup dg
where ao.GROUP_NUMBER = dg.GROUP_NUMBER;
```

## Looking at ASM disks 

New disks will show up with a header_status of "CANDIDATE"

```
SELECT * from v$asm_disks;
```

## Adding Disks to a Diskgroup

You can add disks to a diskgroup using the command:

```
ALTER DISKGROUP XXXX ADD DISK '/dev/name1' NAME BLAH1,
                              '/dev/name2' NAME BLAH2;
```
etc.

If you run into an error (disk not visible across all nodes in a cluster for example), you may find that the disks show in the v$asm_disks view have no name, but show as members of disk group 0.
You can get around this by  using  this command:

```
ALTER DISKGROUP XXXX ADD DISK '/dev/name1' NAME BLAH1 FORCE,
                              '/dev/name2' NAME BLAH2 FORCE;
```

## Rebalancing Disks

You can force a re-balance using this SQL. Note the power clause changes the parallelism of the operation (makes it quicker, at the expense of more resource usage)

```
ALTER DISKGROUP XXXX REBALANCE POWER 11;
```
