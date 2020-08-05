 
Azure data engineering
DP-200
DP-201


| Key | Datastore | Use case | Levels |
|---|---|---|---|
| Role Based Access Control (Active Directory) | All resources | All kinds of roles | Expires, needs to be renewed periodically |
|---|---|---|---|
| Shared Access Signatures | Azure Storage (BLOB, ...) | 3rd party app/Restricted access | account, resource, container, ... |
| Shared Key | Azure Storage (BLOB, ...) | Admin/Full access |  |
| Resource Token | Cosmos DB | 3rd party app/Restricted access |  |
| Master Key | Cosmos DB | Admin/Full access |  |
| Access Control List | Data Lake Gen2 | Granular/Restricted access | Directory, file, ... |



### Azure Data Lake Gen2
[Overview](https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction), 
[Best Practices](https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-best-practices), 
[Summary - 3rd party](https://www.blue-granite.com/blog/10-things-to-know-about-azure-data-lake-storage-gen2)
 - Multi-modal (file system, BLOB)
 - Increased granularity for security (POSIX/Access Control List), data engineering and science (databricks).
 - Renaming/delete operations much faster as opposed to wasb-BLOB.
 - `abfs[s]://[container].dfs.core.windows.net/` (ADLS Gen2) vs `wasb[s]://[container].blob.core.windows.net/` (BLOB storage)
 - Container -> Directory -> File as opposed to BLOB storage: Container -> Virtual Directory -> blob.
 
 [**Optimization**](https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-performance-tuning-guidance)
 - Large files are better than small files: `256MB to 100GB`
 - Folder organization: time series/streaming: `\DataSet\YYYY\MM\DD\HH\mm\datafile_YYYY_MM_DD_HH_mm.tsv`.
 Keeping the smallest unit on the right end makes sure that the analytics engine/reading service only has to read a fraction of the data instead of reading entire folders.
 - Read-Write operations with `4MB-16MB` size.
 
 **Tiers**: Hot (frequent access, ex: daily), Cool (infrequent access, ex: monthly), Archive (rare access, ex: yearly)
 
 **Costs**: Storage cost the same as BLOB storage BUT **transaction** cost is higher.

### Cosmos DB

Two main problems with Cosmos DB: 
 - Expensive
 - Can handle mostly **simple queries**

Everything is stored as ARS = Atomic Record Sequence.

| API | Database | Containers | Items |
|---|---|---|---|
| SQL API | Database | Container | Document  |
| MongoDB API | Database | Collection | Document |
| Gremlin API | Database | Graph | Node/Edge |
| Table API |  | Table | Item |
| Cassandra API | Keyspace | Table | Row |

**Partition key**: Defined at provision time -- cannot be changed later
 - High cardinality (**WRITE** optimized, example: vehicalid + date; extreme cardinality bad for READ operations)
 - Evenly distribute requests (avoid data skewness)
 - Evenly distribute storage (20 GB max). That is, avoid **hot partition** problem.
 - For **READ** operations, find a partition key that is even and does not have high cardinality.


**Time To Live**: https://docs.microsoft.com/en-us/azure/cosmos-db/time-to-live
 - Container: null
   - Item: null, -1, n (seconds) -- item will never expire (default)
 - Container: -1
   - Item: 
     - null or -1: item will never expire
     - n (seconds): item will expire after n seconds
 - Container: n
   - Item:
     - null: item will expire after n seconds (inherits from container)
     - -1: item will never expire even though the container will expire after n seconds
     - n' seconds: item will expire after n' seconds

**RU**: Throughput cost = CPU + I/O + Memory
 - Request Units **per second** depend on size, query size, indexing, properties, indexed properties, consistency level (strong ~ 2 * eventual for read operations)
 - Storage costs are separate
 - Higher than provisioned RU ==> rate-limited. Azure CLI: `--throughput`
 - **Total RU** count = total RUs consumed, useful for monitoring/optimizing.



### Synapse Analytics

[Best practices](https://docs.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-best-practices), [Cheatsheet](https://docs.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/cheat-sheet), [SQL Tables overview](https://docs.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-overview)

(including the use of CTAS, create external table just creates a table in the memory without actually copying it anywhere)

**ELT**: Extract, Load (in parallel) and Transform (using many nodes) as opposed to ETL in Databricks -- that is traditional ETL

Kinds of tables: Clustered index, Clustered columnstore (default), Heap
 - By default, SQL pool stores a table as a clustered columnstore index. This form of data storage achieves high data compression and query performance on large tables.
 - A heap table can be especially useful for loading transient data, such as a staging table which is transformed into a final table.

**Distributions**
 - Replicated: <2 GB, usually dimension tables
 - Hash: 
   - Large tables: Best query performance
   - Small tables/Dimension tables with a common hash key to a fact table with frequent join operations can be hash distributed.
 - Round Robin: Evenly distributed 
   - ideal for staging tables
   - tables (both large/small) with NO JOINS

**Partitioning**
In 99 percent of cases, the partition key should be based on `date`.
 - Only empty partitions can be split in when a columnstore index exists on the table. Consider disabling the columnstore index before issuing the ALTER PARTITION statement, then rebuilding the columnstore index after ALTER PARTITION is complete.
 - When creating partitions on clustered columnstore tables, it is important to consider how many rows belong to each partition. For optimal compression and performance of clustered columnstore tables, a minimum of 1 million rows per distribution and partition is needed. Before partitions are created, Synapse SQL pool already divides each table into 60 distributed databases.
https://docs.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-partition

**Units**: Increase DWU for better performance.
 - `DWU` = CPU + Memory + IO.
 - Storage: Costs are separate.
 - `PAUSE` option available.
 
**Optimization**
 - Result-set cache: repetitive queries with infrequent data changes are stored in a cache.
 - Workload groups: Prioritize some requests over others, limit resources for unimportant requests.

**Architecture**: Control node + 60 compute nodes.

**Polybase**:
1. Extract files into txt/csv.
2. Load into BLOB or Data Lake Gen2.
3. Prepare data for loading.
4. Import data into temp/staging table (round-robin distribution) in Synapse Analytics.
5. Transform.
6. Insert into production tables.

More fine grained:
 - Create MASTER key.
 - Create scoped credential.
 - External Data Source.
 - External File Format (SCHEMA).
 - External Table (view only -- stays in memory, not on disk)
 - Table (physical disk) using `CTAS` = Create Table as Select.

### Security

**Best practices**:

https://docs.microsoft.com/en-us/azure/security/fundamentals/data-encryption-best-practices

https://docs.microsoft.com/en-us/azure/azure-sql/database/security-best-practice

https://www.youtube.com/watch?v=mntOLLNejUo

**Access/Authorization**

**Data Lake Gen2**
https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-access-control
 - RBAC (Role Based Access Control; defined in **Active Directory**) Granular access BUT expires unless renewed.
 - Access Control Lists (file/directory; available in Data Lake Gen2, not in BLOB): 
**Read**=4, **Write**=2, **Execute**=1
 - Shared Key/Access Key: Full access including ability to modify ACLs.
 - [Shared Access Signatures](https://docs.microsoft.com/en-us/azure/storage/common/storage-sas-overview): Restricted access (3rd party APPs).

**BLOB**: https://docs.microsoft.com/en-us/azure/storage/blobs/security-recommendations

**SQL/Synapse Analytics**
 - **Advanced Data Security**: Discovering, classifying, and labeling columns that contain sensitive data in your database. Sensitive Data dashboard.
https://docs.microsoft.com/en-us/azure/azure-sql/database/data-discovery-and-classification-overview
 - Auditing: enable database auditing.
 - [Row Level Security](https://docs.microsoft.com/en-us/sql/relational-databases/security/row-level-security?view=sql-server-2017) using predicates/functions for read-only access or other kinds.
 - Master key (SQL) is required for column encryption/decryption tasks.
 - [Always Encrypted](https://docs.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine?view=sql-server-2017) supports two types of encryption: randomized (more secure) and deterministic (querying/joins). It does **column-level** encryption.
 - [Row level security](https://docs.microsoft.com/en-us/sql/relational-databases/security/row-level-security?view=sql-server-2017) (`CREATE SECURITY POLICY` - T-SQL) encrypts the WHOLE row; not useful where only one column like address has to be masked.

**Databricks**: https://docs.microsoft.com/en-us/learn/modules/describe-platform-architecture-security-data-protection-azure-databricks/

**Private peering/VNets/ExpressRoutes**: https://docs.microsoft.com/en-us/azure/storage/common/storage-private-endpoints

### Monitoring

 - [Dynamic Management Views](https://docs.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-manage-monitor) can be used to monitor Azure **Synapse Analytics**. In particular, `sys.dm_pdw_exec_sessions` stores login information for the last 10k logins. `sys.dm_pdw_exec_requests` logs the last 10k requests (their elapsed time and such).
 - Push notifications send alerts to Azure **mobile** app.
 - Azure [IT Service Management connector](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/itsmc-connections) allows you to connect Azure to non-Azure services. supports: *Cherwell, Provance, ServiceNow, System Center Service Manager*. It can be used in a bi-directional way to exchange information between Azure Monitor/Log Analytics and an external service.
 
 **Log queries/analytics**
 - Starting January 11,2019, creating or modifying log alert rules that use `search`, or `union` operators will **not** be supported the in Azure portal. 
 - [Log alerts](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/alerts-log): KQL queries for alerts should not contain a **time filter**. Time filters are set by `period` in the alert setup.
