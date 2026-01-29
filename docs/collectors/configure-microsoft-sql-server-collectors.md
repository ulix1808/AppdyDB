# Configure Microsoft SQL Server Collectors

> **Fuente (documentación 24.x, puede ser retirada):**  
> https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-microsoft-sql-server-collectors

---

This page provides configuration details for Microsoft SQL Server collectors.

## Prerequisites

- Microsoft SQL Server 2005 or newer.
- For Azure SQL Managed Instance, follow the same configuration as for a Microsoft SQL Server Collector.

**Nota:** Database Agent does not work if you enable both [certain conflicting options] in the Microsoft SQL Server database (see [JDBC driver issue](https://github.com/Microsoft/mssql-jdbc/issues/963)).

## Connection Details

- **Hostname or IP Address:** For a cluster, use the listener hostname/IP. See Monitor the Microsoft SQL Server Cluster.
- **Listener Port:** For a cluster, specify the listener port. Enable cluster monitoring to discover all nodes. See Monitor the Microsoft SQL Server Cluster.
- **Custom JDBC Connection String:** Optional.
- **Sub-Collectors:** Up to 29 sub-collectors. Different parameters only via [Create Collector API](https://docs.appdynamics.com/appd/24.x/latest/en/extend-splunk-appdynamics/splunk-appdynamics-apis/database-visibility-api). Cannot convert custom cluster to standalone.
- **Exclude Databases:** Comma-separated.
- **Monitor Operating System:** See [Configure the Database Agent to Monitor Server Hardware](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-the-database-agent-to-monitor-server-hardware).

By default the Database Agent uses TLSv1.1 for SSL. For SQL Server 2019 and later, set TLSv1.2 or TLSv1.3 via connection parameters to avoid SSL version errors.

For Amazon RDS High Availability (Multi-AZ), set the `databaseName` JDBC connection string property to a user database.

## Monitor Microsoft SQL Server

You can use an existing user or create a new user with the relevant privileges.

### Monitor the Microsoft SQL Server Cluster

Database Visibility supports Always On cluster discovery for MSSQL >= 2012. Enable the appropriate property at Controller or agent level to monitor all nodes in the availability group. Read-only routing is supported.

### Authentication Methods

- Azure Active Directory Integrated (for Azure SQL Managed Instance; install Microsoft ODBC driver)
- Azure Active Directory Password
- SQL Server authenticated account (Windows or Linux agent)
- Windows authenticated account (Windows agent)

**Windows Authentication:** Select Windows Authentication in Create New Collector; do not specify username/password. Set `-Djava.library.path` to `\<db_agent_home>\auth\x64` (64-bit) or `\auth\x86` (32-bit). Ensure the Windows account has privileges and, if using a Windows service, set the service logon to that account.

**Azure Active Directory Password:** Add the appropriate property in Custom JDBC Connection String or Connection Properties.

### Server Level Permissions for SQL Server Logon

Create a login and map the user to **master** and **msdb** (mandatory). Grant the documented privileges (substitute with the login name). Optional object permissions for Object Browser (Users, Storage, Job Status, Error Log, Database) are documented in the original AppDynamics page; roles such as securityAdmin and public may be required depending on the screen.
