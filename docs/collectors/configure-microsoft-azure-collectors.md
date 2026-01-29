# Configure Microsoft Azure Collectors

> **Fuente (documentación 24.x, puede ser retirada):**  
> https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-microsoft-azure-collectors

---

To monitor Microsoft Azure with Database Visibility, you must run Microsoft Azure >= 2008.

- **Azure SQL Managed Instance:** Follow [Configure Microsoft SQL Server Collectors](./configure-microsoft-sql-server-collectors.md).
- This page is specific to **Azure SQL Database** (not Managed Instance). Hardware monitoring is not supported for Microsoft Azure SQL databases.

## Connection Details

| Section | Field | Description |
|---------|-------|-------------|
| Create New Collector | Database Type, Agent, Collector Name | As usual. |
| Connection Details | Hostname or IP Address, Database, Listener Port, Custom JDBC Connection String | |
| Username and Password | Username, Password | User with permissions per [User Permissions for Microsoft Azure](https://docs.appdynamics.com/docs.appdynamics.com#id-.ConfigureMicrosoftAzureCollectorsv24.3-permis). |
| | CyberArk | Enable CyberArk; download JavaPasswordSDK.jar, rename to cyberark-sdk-9.5.jar, copy to lib directory. |
| Advanced Options | Sub-Collectors, Connection Properties | Same rules as other relational DBs (up to 29 sub-collectors, API for different params). |

## Authentication Methods

- Windows authenticated account (Database Agent on Windows)
- SQL Server authenticated account (Windows or Linux)
- [Azure Active Directory Password](https://docs.appdynamics.com/docs.appdynamics.com#id-.ConfigureMicrosoftAzureCollectorsv24.3-azuread)

**Windows Authentication:** Select Windows Authentication in Create New Collector; do not specify username/password. Set `-Djava.library.path` to `C:\dbagent_install_dir\auth\x64` (64-bit) or `\auth\x86` (32-bit). Ensure the Windows account has appropriate privileges; if using a Windows service, set the service logon to that account.

**Azure Active Directory Password:** Add property `authentication` = `ActiveDirectoryPassword` in Custom JDBC Connection String or Connection Properties.

## User Permissions for Microsoft Azure

The monitoring user can be an SQL Server authenticated account. Minimum permissions:

1. Create login: `CREATE LOGIN DBMon_Agent_User WITH PASSWORD = 'Password123'`
2. Create user in Azure SQL database: `CREATE USER DBMon_Agent_User FOR LOGIN DBMon_Agent_User WITH DEFAULT_SCHEMA = dbo`
3. Grant: `grant VIEW DATABASE STATE to DBMon_Agent_User`
4. Read-only access to master (or custom database): `USE master; EXEC sp_addrolemember 'db_datareader', '(your_user_name)'`
