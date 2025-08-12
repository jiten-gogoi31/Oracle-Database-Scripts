**About Oracle Scheduler Job:**
Oracle Database includes Oracle Scheduler, a job scheduler to help you simplify the scheduling of hundreds or even thousands of tasks. Oracle Scheduler (the Scheduler) is implemented by the procedures and functions in the DBMS_SCHEDULER PL/SQL package.
You can run program units, that is, PL/SQL anonymous blocks, PL/SQL stored procedures, and Java stored procedures on the local database or on one or more remote Oracle databases.
Run external executables, (executables that are external to the database)



****Creating a scheduler job in Oracle RDS database**
Parameters required:
•	Frequency of the job to be run.
•	Package or procedure name to execute.
•	From when we must start the job.
•	Reason for job creation.

To check existing scheduler jobs in database other than oracle pre-defined job.

SELECT OWNER,JOB_NAME,JOB_ACTION,REPEAT_INTERVAL FROM DBA_SCHEDULER_JOBS WHERE OWNER NOT IN ('SYS','ORACLE_OCM') ORDER BY OWNER,JOB_NAME;

Statement to Create a scheduler job:

BEGIN
DBMS_SCHEDULER.CREATE_JOB (
    JOB_NAME        => 'SSIM_MONITOR_JOB', -- Name of the Job
    JOB_TYPE       	 => 'PLSQL_BLOCK', -- Type of the Job
    JOB_ACTION      => 'BEGIN AIGIFX.IFX4_REPORTS.REGULAR_MONITERING_REPORT; END;', --Sql block to executed.
    START_DATE      => '1-FEB-23 12.30.00 AM +00:00', -- Start date of the job
    REPEAT_INTERVAL => 'FREQ=HOURLY; BYDAY=WED; INTERVAL=2;', --- Frequency of the job
    ENABLED         	     => TRUE,
    COMMENTS        => 'IF SSIM IS NOT BEEN PROCESSED); --- comments
END;
/


Check currently running scheduler jobs in Database:

SELECT JOB_NAME, SESSION_ID, RUNNING_INSTANCE, ELAPSED_TIME, CPU_USED FROM DBA_SCHEDULER_RUNNING_JOBS;

Disable a job:
execute dbms_scheduler.disable('owner.job');

Drop a job:
execute DBMS_SCHEDULER.DROP_JOB('EGATEORACLE1.KA_DELETE_REPORT_JOB');

Stop a job:

exec DBMS_SCHEDULER.STOP_JOB(job_name =>'GP4QA.DBMS_JOB$_208663',force => TRUE);

Run a job:

begin
dbms_scheduler.run_job('IFXPRODORACLE.REGULAR_MONITORING_REPORT_JOB');
end;
/

