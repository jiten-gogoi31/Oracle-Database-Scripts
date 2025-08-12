## To check Active sessions in database
```
select SID,SERIAL#,USERNAME,STATUS,OSUSER,module,MACHINE,program,sql_id,to_char(LOGON_TIME,'DD-MON-YYYY HH24:MI:SS'),event from v$session where status='ACTIVE' and username is not null;
```

## Long Running operations/sessions:
```
SELECT opname,sid, CONTEXT,TARGET, SOFAR, TOTALWORK,  ROUND(SOFAR/TOTALWORK*100,2) "%_COMPLETE", time_remaining/3600, elapsed_seconds/3600 FROM V$SESSION_LONGOPS    WHERE  TOTALWORK != 0  AND SOFAR <> TOTALWORK and sid in (select sid from v$session );
```


## To check sql_text in the running sessions:
```
select distinct b.SID,b.SERIAL#,b.USERNAME,b.STATUS,b.OSUSER,b.MACHINE,b.program,b.sql_id,to_char(b.LOGON_TIME,'DD-MON-YYYY HH24:MI:SS') logontime,a.HASH_VALUE, a.sql_text
from v$sql a, v$session b
where b.sql_address = a.address
and b.sql_hash_value = a. hash_value
and b.status ='ACTIVE' ;
```

## Query to find Blocking Sessions:

```
select s1.username || '@' || s1.machine
|| ' ( SID=' || s1.sid || ' )  is blocking '
|| s2.username || '@' || s2.machine || ' ( SID=' || s2.sid || ' ) ' AS blocking_status
from v$lock l1, v$session s1, v$lock l2, v$session s2
where s1.sid=l1.sid and s2.sid=l2.sid
and l1.BLOCK=1 and l2.request > 0
and l1.id1 = l2.id1
and l2.id2 = l2.id2 ;
```

## Find Blocked Sessions.

```
select a.SID "Blocking Session", b.SID "Blocked Session"  
from v$lock a, v$lock b 
where a.SID != b.SID and a.ID1 = b.ID1  and a.ID2 = b.ID2 and 
b.request > 0 and a.block = 1;
```


## Find Lock Wait Time.

```
SELECT 
  blocking_session "BLOCKING_SESSION",
  sid "BLOCKED_SESSION",
  serial# "BLOCKED_SERIAL#", 
  seconds_in_wait/60 "WAIT_TIME(MINUTES)"
FROM v$session
WHERE blocking_session is not NULL
ORDER BY blocking_session;
```


## Find Blocked SQL.

```
SELECT SES.SID, SES.SERIAL# SER#, SES.PROCESS OS_ID, SES.STATUS, SQL.SQL_FULLTEXT
FROM V$SESSION SES, V$SQL SQL, V$PROCESS PRC
WHERE
   SES.SQL_ID=SQL.SQL_ID AND
   SES.SQL_HASH_VALUE=SQL.HASH_VALUE AND 
   SES.PADDR=PRC.ADDR AND
   SES.SID=&Enter_blocked_session_SID;
```

## script to get the sql stmts involved in blocking sessions :

```
select distinct b.SID,b.SERIAL#,b.USERNAME,b.STATUS,b.OSUSER,b.MACHINE,b.program,b.sql_id,to_char(b.LOGON_TIME,'DD-MON-YYYY HH24:MI:SS') logontime,a.HASH_VALUE, a.sql_text
from v$sql a, v$session b
where b.sql_address = a.address
and b.sql_hash_value = a. hash_value
and b.sid in(select s1.sid
from v$lock l1, v$session s1, v$lock l2, v$session s2
where s1.sid=l1.sid and s2.sid=l2.sid
and l1.BLOCK=1 and l2.request > 0
and l1.id1 = l2.id1
and l2.id2 = l2.id2 
union
select s2.sid 
from v$lock l1, v$session s1, v$lock l2, v$session s2
where s1.sid=l1.sid and s2.sid=l2.sid
and l1.BLOCK=1 and l2.request > 0
and l1.id1 = l2.id1
and l2.id2 = l2.id2 ) order by logontime;
```

## Query to find object locks.

```
select s.sid, s.serial#,p.pid,s.machine, p.spid vprocess_spid,
    to_char(logon_time,'MM/DD/YY HH24:MI:SS') time,
    s.status sstatus, s.username susername,d.object_name
    from   dba_objects d, v$locked_object l, v$session s,
    v$process p
    where  (l.session_id=s.sid)
    and (p.addr = s.paddr)
    and (d.object_id = l.object_id)
    and s.sid in (select sid from v$lock);
```

## query to check instance uptime.

```
 select INSTANCE_NAME,HOST_NAME,to_char(STARTUP_TIME,'DD/MON/YYYY HH24:MI:SS') from v$instance;
```
 
## Check the startup history timing of Oracle Instance.

```
COL INSTANCE_NAME FOR A10
SELECT INSTANCE_NAME,TO_CHAR(STARTUP_TIME, 'HH24:MI DD-MON-YY') FROM DBA_HIST_DATABASE_INSTANCE ORDER BY STARTUP_TIME DESC;
```
 
## RDS EXPORT AND IMPORT LOGS.

** To kill single session in RDS:**

 cmd:
 ```
   exec rdsadmin.rdsadmin_util.kill(Sid, Serial# ,'IMMEDIATE');
```
**   example:**
```
  exec rdsadmin.rdsadmin_util.kill(1448, 5 ,'IMMEDIATE');
```
  
## To kill multiple sessions in RDS based on condition.
  
  ```
   SELECT
       s.sid,
       s.serial#,
       p.spid,
       S.Username,
       S.Program,
       'exec rdsadmin.rdsadmin_util.kill('||s.sid||','||s.serial#||','||'''immediate'');' as Execute_command
FROM   v$session s
       Join v$process P On P.Addr = S.Paddr 
Where  S.Type != 'BACKGROUND'
  AND s.machine='esobatch-01.as.ifxgp.prod.va' and s.status='ACTIVE';
```
  
  
## To check the list of files and sizes in a database directory:
   
```
  select filename,type,round(filesize/1024/1024/1024,2) GB,mtime from table(RDSADMIN.RDS_FILE_UTIL.LISTDIR('DATA_PUMP_DIR')) order by mtime desc; 
```
  
## To check and read the contents of log files in the RDS.

```
  select * from table(RDSADMIN.RDS_FILE_UTIL.READ_TEXT_FILE('DATA_PUMP_DIR','so_pax_041217_testimp.log'));
```
  
## To remove the dump & log files from the database directories.

```
  exec utl_file.fremove('DATA_PUMP_DIR','sample_copied.dmp');
```

## To check the list of files and sizes in a database directory.
```
  select filename,type,round(filesize/1024/1024/1024,2) GB,mtime from table(RDSADMIN.RDS_FILE_UTIL.LISTDIR('DATA_PUMP_DIR')) order by mtime desc;
```
  

## To get Query_Plan using sql_id.

```
SELECT *   FROM TABLE(DBMS_XPLAN.DISPLAY_CURSOR('47kzwsj0q9t43'));
```

## To get full sql text using sql_id.
```
select a.sql_text from v$sqltext_with_newlines a where sql_id='6333f20psxrr5' ORDER BY a.piece;
```

## Another query to get sql full text.
```
select SQL_FULLTEXT from v$sql where sql_id='c0rcw540b2azp';
```

## To check the archivelog retention configuration. 
```
exec rdsadmin.rdsadmin_util.show_configuration;
```

## To check the archivelogs in the archive log directory.
```
select sum(FILESIZE)/1024/1024/1024 ARCH_GB from table (rdsadmin.rds_file_util.listdir(p_directory => 'ARCHIVELOG_DIR')) order by mtime;
```

## To set the archivelog retention hours. 
```
begin
    rdsadmin.rdsadmin_util.set_configuration(
        name  => 'archivelog retention hours',
        value => '6');
end;
/
```

## Monitoring scheduler jobs:

## To list all scheduler jobs in the database:
```
SELECT owner,job_name,job_action,start_date FROM dba_scheduler_jobs;
```

## query to find the jobs currently running.
```
SELECT job_name, session_id, running_instance, elapsed_time, cpu_used FROM dba_scheduler_running_jobs;
```

## Query to find the history details of job that has run.
```
SELECT job_name, log_date, status, actual_start_date, run_duration, cpu_used FROM dba_scheduler_job_run_details;
```

## To find the jobs that haven’t succeeded.
```
SELECT job_name, log_date, status, run_duration, cpu_used,additional_info FROM dba_scheduler_job_run_details where status='FAILED';
```


## Queries to find database size.

**Total DB size**

```
   select owner,sum(BYTES/1024/1024/1024) SIZE_GB from dba_segments group by owner order by 2 desc;
```
   
**Each Schema wise**

```
select owner,sum(BYTES/1024/1024/1024) SIZE_GB from dba_segments group by owner order by 2 desc;
```
	 
**One particular schema**
```
select owner,segment_name,sum(BYTES/1024/1024/1024) SIZE_GB from dba_segments where owner='GPPROD' group by owner,segment_name order by 3 desc;
```
		  
**query to know one particular database object size**
 ``` 
 select owner,segment_name,sum(BYTES/1024/1024/1024) SIZE_GB from dba_segments where owner='ESSPROD2' and segment_name='FSS_DAILY_FLT_SERVICE' group by owner,segment_name order by 3 desc;
```
 

## Query to check Tablespace sizes
```
SELECT * FROM (
SELECT c.tablespace_name,
ROUND(a.bytes/1048576,2) MB_Allocated,
ROUND(b.bytes/1048576,2) MB_Free,
ROUND((a.bytes-b.bytes)/1048576,2) MB_Used,
ROUND(b.bytes/a.bytes * 100,2) tot_Pct_Free,
ROUND((a.bytes-b.bytes)/a.bytes,2) * 100 tot_Pct_Used
FROM (SELECT tablespace_name,
SUM(a.bytes) bytes
FROM sys.DBA_DATA_FILES a
GROUP BY tablespace_name) a,
(SELECT a.tablespace_name,
NVL(SUM(b.bytes),0) bytes
FROM sys.DBA_DATA_FILES a,
sys.DBA_FREE_SPACE b
WHERE a.tablespace_name = b.tablespace_name (+)
AND a.file_id = b.file_id (+)
GROUP BY a.tablespace_name) b,
sys.DBA_TABLESPACES c
WHERE a.tablespace_name = b.tablespace_name(+)
AND a.tablespace_name = c.tablespace_name
) WHERE tot_Pct_Used >=0
ORDER BY 2 desc;
```

## Query to check temp Tablespace size.

```
select a.tablespace_name tablespace,
         d.TEMP_TOTAL_MB,
         sum (a.used_blocks * d.block_size) / 1024 / 1024 TEMP_USED_MB,
         d.TEMP_TOTAL_MB - sum (a.used_blocks * d.block_size) / 1024 / 1024 TEMP_FREE_MB
from v$sort_segment a,
        (
          select   b.name, c.block_size, sum (c.bytes) / 1024 / 1024 TEMP_TOTAL_MB
          from     v$tablespace b, v$tempfile c
          where    b.ts#= c.ts#
          group by b.name, c.block_size
        ) d
where    a.tablespace_name = d.name
group by a.tablespace_name, d.TEMP_TOTAL_MB;
```

## Query to check Temp tablespace usage per session wise.

```
SELECT   S.sid || ',' || S.serial# sid_serial, S.username, S.osuser, P.spid, S.module,
P.program, SUM (T.blocks) * TBS.block_size / 1024 / 1024 mb_used, T.tablespace,
COUNT(*) statements
FROM     v$sort_usage T, v$session S, dba_tablespaces TBS, v$process P
WHERE    T.session_addr = S.saddr
AND      S.paddr = P.addr
AND      T.tablespace = TBS.tablespace_name
GROUP BY S.sid, S.serial#, S.username, S.osuser, P.spid, S.module,
P.program, TBS.block_size, T.tablespace
ORDER BY mb_used;
```

## To check the table fragmentation.
```
SELECT T.OWNER,

           T.TABLE_NAME,

           ROUND((T.BLOCKS * 8),2) AS "SIZE" , 

           ROUND((T.NUM_ROWS * T.AVG_ROW_LEN / 1024),2) "ACTUAL",

          (ROUND((T.BLOCKS * 8),2) - ROUND((T.NUM_ROWS * T.AVG_ROW_LEN / 1024), 2)) "WASTED",

           ROUND((T.NUM_ROWS * T.AVG_ROW_LEN / 1024) / (BLOCKS * 8) * 100, 1)  AS "OCCUPANCY"

      FROM DBA_TABLES T

     WHERE (ROUND((T.BLOCKS * 8),2) > ROUND((T.NUM_ROWS * T.AVG_ROW_LEN / 1024), 2))

       -------------------------------------------------------------------------

  --     AND T.OWNER IN ('IFXPROD2')

   --   AND T.TABLE_NAME in('TBL_LOG')

       AND ROUND((BLOCKS * 8),2) > 10000

       AND ROUND((T.NUM_ROWS * T.AVG_ROW_LEN / 1024) / (BLOCKS * 8) * 100, 1) < 75

       -------------------------------------------------------------------------

     ORDER BY 5 DESC;
 ```

