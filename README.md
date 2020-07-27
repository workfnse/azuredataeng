# azuredataeng
Azure data engineering
DP-200
DP-201

### Security

**Best practices**:

https://docs.microsoft.com/en-us/azure/security/fundamentals/data-encryption-best-practices

https://docs.microsoft.com/en-us/azure/azure-sql/database/security-best-practice

https://www.youtube.com/watch?v=mntOLLNejUo

 - Column-level encryption in Synapse/SQL is non-deterministic; no option of deterministic.
 - Master key (SQL) is required for column encryption/decryption tasks.
 - [Always Encrypted](https://docs.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine?view=sql-server-2017) supports two types of encryption: randomized (more secure) and deterministic (querying/joins).
 - [Row level security](https://docs.microsoft.com/en-us/sql/relational-databases/security/row-level-security?view=sql-server-2017) (`CREATE SECURITY POLICY` - T-SQL) encrypts the WHOLE row; not useful where only one column like address has to be masked.

**Databricks**: https://docs.microsoft.com/en-us/learn/modules/describe-platform-architecture-security-data-protection-azure-databricks/


### Monitoring

 - [Dynamic Management Views](https://docs.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-manage-monitor) can be used to monitor Azure Synapse Analytics. In particular, `sys.dm_pdw_exec_sessions` stores login information for the last 10k logins. `sys.dm_pdw_exec_requests` logs the last 10k requests (their elapsed time and such).
 - Push notifications send alerts to Azure **mobile** app.
 - Azure [IT Service Management connector](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/itsmc-connections) allows you to connect Azure to non-Azure services. supports: Cherwell, Provance, ServiceNow, System Center Service Manager. It can be used in a bi-directional way to exchange information between Azure Monitor/Log Analytics and an external service.
