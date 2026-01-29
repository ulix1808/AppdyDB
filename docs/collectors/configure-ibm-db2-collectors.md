# Configure IBM DB2 Collectors

> **Fuente (documentación 24.x, puede ser retirada):**  
> https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-ibm-db2-collectors

---

This page describes configuration details for IBM DB2 collectors.

## Prerequisites

DB2 >= 9.x

## Connection Details

- **Database:** The name of the database instance. If the instance hosts multiple databases, create a separate collector for each database.
- **Custom JDBC Connection String:** Optional. For Kerberos, select the Kerberos option under Advanced Options.
- **Sub-Collectors:** Up to 29 sub-collectors (30 databases in a custom cluster). Parameters other than hostname/IP and port are the same as the main collector; different parameters only via [Create Collector API](https://docs.appdynamics.com/appd/24.x/latest/en/extend-splunk-appdynamics/splunk-appdynamics-apis/database-visibility-api). Cannot convert custom cluster collector to standalone.
- **Connection Properties:** Add or edit JDBC connection properties. For Kerberos: [Monitor IBM DB2 Databases Using Kerberos Authentication](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-ibm-db2-collectors/monitor-ibm-db2-databases-using-kerberos-authentication).
- **Monitor Operating System:** See [Configure the Database Agent to Monitor Server Hardware](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-the-database-agent-to-monitor-server-hardware).

| Section | Field | Description |
|---------|-------|-------------|
| Create New Collector | Database Type, Agent, Collector Name | As usual. |
| Connection Details | Hostname or IP Address, Database, Listener Port, Custom JDBC Connection String | |
| Username and Password | Username, Password | User with permissions per User Permissions for IBM DB2 LUW. |
| | CyberArk | Enable CyberArk; download and rename JAR, copy to agent lib directory. |
| Advanced Options | Sub-Collectors, Connection Properties, Kerberos, Monitor Operating System | |

## User Permissions for IBM DB2 LUW

The monitoring user needs **SYSMON** authority and connection privileges, and must be part of the **sysmon_group**.

### DB2 >= 9.7

Enable monitoring switch "TIMESTAMP" for full Database Visibility functionality. Grant the required privileges (replace with the user name under which you run the Database Visibility Agent).

### DB2 9.5

Enable monitoring switches "STATEMENT" and "TIMESTAMP". Grant the required privileges.

### User Permissions When restrict_access is Set to YES

Grant the additional privileges as documented for your DB2 version.

## Generate Execution Plans

The monitoring user ID must have access to the explain_* tables. Create the explain tables (e.g. via DB2 tools under SYSTOOLS schema or another schema). Explain plans require SELECT on every table accessed in the SQL plus the necessary explain_* tables.
