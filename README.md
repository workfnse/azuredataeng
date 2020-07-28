
Azure data engineering
DP-200
DP-201

### Synapse Analytics
Cheatsheet: https://docs.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/cheat-sheet

Table overview: https://docs.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-overview
(including the use of CTAS, create external table just creates a table in the memory without actually copying it anywhere)

Kinds of tables: Clustered index, Clustered columnstore (default), Heap
 - By default, SQL pool stores a table as a clustered columnstore index. This form of data storage achieves high data compression and query performance on large tables.
 - A heap table can be especially useful for loading transient data, such as a staging table which is transformed into a final table.

**Distributions**
 - Replicated: <2 GB, usually DIM tables
 - Hash: Best query performance for large tables 
 - Round Robin: Evenly distributed + ideal for staging tables

**Partitioning**
In 99 percent of cases, the partition key should be based on date.
 - Only empty partitions can be split in when a columnstore index exists on the table. Consider disabling the columnstore index before issuing the ALTER PARTITION statement, then rebuilding the columnstore index after ALTER PARTITION is complete.
 - When creating partitions on clustered columnstore tables, it is important to consider how many rows belong to each partition. For optimal compression and performance of clustered columnstore tables, a minimum of 1 million rows per distribution and partition is needed. Before partitions are created, Synapse SQL pool already divides each table into 60 distributed databases.
https://docs.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-partition

### Security

**Best practices**:

https://docs.microsoft.com/en-us/azure/security/fundamentals/data-encryption-best-practices

https://docs.microsoft.com/en-us/azure/azure-sql/database/security-best-practice

https://www.youtube.com/watch?v=mntOLLNejUo

**SQL/Synapse Analytics**
 - Column-level encryption in Synapse/SQL is non-deterministic; no option of deterministic? LINK?
 - Master key (SQL) is required for column encryption/decryption tasks.
 - [Always Encrypted](https://docs.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine?view=sql-server-2017) supports two types of encryption: randomized (more secure) and deterministic (querying/joins).
 - [Row level security](https://docs.microsoft.com/en-us/sql/relational-databases/security/row-level-security?view=sql-server-2017) (`CREATE SECURITY POLICY` - T-SQL) encrypts the WHOLE row; not useful where only one column like address has to be masked.

**Databricks**: https://docs.microsoft.com/en-us/learn/modules/describe-platform-architecture-security-data-protection-azure-databricks/


### Monitoring

 - [Dynamic Management Views](https://docs.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-manage-monitor) can be used to monitor Azure **Synapse Analytics**. In particular, `sys.dm_pdw_exec_sessions` stores login information for the last 10k logins. `sys.dm_pdw_exec_requests` logs the last 10k requests (their elapsed time and such).
 - Push notifications send alerts to Azure **mobile** app.
 - Azure [IT Service Management connector](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/itsmc-connections) allows you to connect Azure to non-Azure services. supports: *Cherwell, Provance, ServiceNow, System Center Service Manager*. It can be used in a bi-directional way to exchange information between Azure Monitor/Log Analytics and an external service.
