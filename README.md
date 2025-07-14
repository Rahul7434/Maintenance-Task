# Maintenance-Task

##### Vacuum
- PostgreSQL VACUUM Overview:
      - PostgreSQL uses the MVCC (Multi-Version Concurrency Control) system for handling transactions, which allows multiple versions of rows in a table. When rows are updated or deleted, old row versions (or "dead tuples") stay in the table until they are cleaned up. The VACUUM process helps in cleaning these dead tuples to free up space and maintain database performance.

- There are three types of vacuum types: 
    1. Regular VACUUM:
	      Cleans up dead tuples but does not reclaim disk space. Instead, it marks the space as reusable.
    2. VACUUM FULL:
	     This fully rewrites the table to remove dead space and return unused disk space to the operating system. However, it locks the table during the operation.
- Important Parameters of VACUUM:
  ```
    1. autovacuum:
	      A background process that automatically runs VACUUM to maintain the database without manual intervention. Configurable via parameters in the postgresql.conf file.
  ```

- The visibility of pages determines whether or not each page has dead tuples. Vacuum processing can skip a page that does not have dead tuples by using the corresponding visibility map (VM).
###### Workflow of VACUUM in PostgreSQL:
1. Marking Dead Tuples:
```
	When a row is updated or deleted, the old version of the row is marked as "dead."
	VACUUM identifies these dead tuples and marks them for removal.
	VACUUM scans the entire table to identify and clean up dead tuples.
	It reads data pages to determine if any dead tuples are present.
```
3. Freeing Space:
```
	Once dead tuples are found, they are removed, and the space is marked as free.
	This space can now be reused for new data inserts or updates.
```
5. Updating Visibility Map:
```
VACUUM updates the visibility map to mark which pages contain only visible (live) tuples, helping future scans avoid unnecessary reads.
```
7. Transaction IDs Wraparound Prevention:
```
Regular VACUUMing also prevents transaction ID wraparound by updating the pg_class and pg_stat_activity to maintain healthy transaction counters.
```
2. vacuum_cost_delay:
```
	Specifies the delay between VACUUM operations to reduce I/O load on the system.
```
4. vacuum_freeze_min_age:
```
	Controls the minimum transaction age at which tuples are frozen to avoid transaction ID wraparound.
```
6. vacuum_defer_cleanup_age:
```
	This sets the number of transactions that can defer tuple cleanup to help reduce conflicts with long-running transactions.
```
8. vacuum_cost_limit:
```
	Determines the limit of I/O operations a VACUUM process can perform before pausing to avoid system overload.
```
-----------------------------------------------------------------------------------------------------------------------------------------
##### Internal Working of VACUUM:
1. Visibility Map and Free Space Map:
	VACUUM updates the Visibility Map, allowing PostgreSQL to track pages with only live tuples for future optimizations.
	The Free Space Map (FSM) tracks which pages have space available for inserts, reducing the need for scanning the entire table.
2. Multi-Version Concurrency Control (MVCC):
	When rows are updated or deleted, MVCC keeps the older versions of the data for transaction consistency. VACUUM removes these old versions that are no longer needed.

4. Importance of autovacuum: 
	Highlight the role of the automatic autovacuum process in preventing performance degradation and wraparound issues without manual intervention.
5. Parameters:
	Mention key parameters that can be tuned for better performance, like vacuum_cost_delay (to reduce I/O impact) and vacuum_freeze_min_age (for freezing old tuples).
----------------------------------------------------------------------------------------------------------------------------
- Syntax:- 
1. Basic VACUUM
```
VACUUM; , VACUUM <table_name>;
```
2. VACUUM FULL
```
VACUUM FULL;  , VACUUM FULL <table_name>;
```
3. VACUUM with ANALYZE
```
VACUUM ANALYZE; , VACUUM ANALYZE <table_name>;
```
4. VACUUM with VERBOSE
```
VACUUM VERBOSE; , VACUUM VERBOSE <table_name>;
```
5. VACUUM FREEZE
```
VACUUM FREEZE; , VACUUM FREEZE <table_name>;
```
6. VACUUM FULL with ANALYZE
```
VACUUM FULL ANALYZE;  VACUUM FULL ANALYZE <table_name>;
```
7. VACUUM with Specific Columns for ANALYZE
```
VACUUM ANALYZE <table_name> (<column1>, <column2>, ...);
```
8. VACUUM for a Specific Schema
```
VACUUM <schema_name>.<table_name>;
```
9. Autovacuum Configuration in postgresql.conf (for background VACUUM)
```
Enable/Disable autovacuum:
autovacuum = on
autovacuum_naptime = 60  # (time in seconds)
autovacuum_vacuum_threshold = 50
autovacuum_analyze_threshold = 50
autovacuum_vacuum_scale_factor = 0.2
autovacuum_analyze_scale_factor = 0.1
vacuum_cost_delay = 20  # (in milliseconds)
vacuum_cost_limit = 200  # (I/O operations before the process pauses)
```
 -------------------------------------------------------------------------------------------------------
- Post-VACUUM Commands:
```
Analyze Table (after VACUUM):
This updates the table statistics for optimized query planning:
ANALYZE <table_name>;
Running VACUUM on Multiple Tables or Databases:
Vacuum all tables in a database:
VACUUM;
Vacuum a specific set of tables:
VACUUM <table_name1>, <table_name2>;
Note on Privileges:
Only the table owner or a superuser can run VACUUM on a table.
By using these syntaxes, you can perform all types of vacuum-related maintenance tasks in PostgreSQL, from basic cleaning to advanced disk space reclamation and wraparound prevention.
```
