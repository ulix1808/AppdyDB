# Configure Sybase Collectors

> **Fuente (documentación 24.x, puede ser retirada):**  
> https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-sybase-collectors

---

To monitor Sybase with Database Visibility, you must run Sybase >= 15.

## Connection Details

- **Sub-Collectors:** Up to 29 sub-collectors. All parameters other than hostname/IP and port are the same as the main collector; different parameters only via [Create Collector API](https://docs.appdynamics.com/appd/24.x/latest/en/extend-splunk-appdynamics/splunk-appdynamics-apis/database-visibility-api). Cannot convert custom cluster to standalone.

| Section | Field | Description |
|---------|-------|-------------|
| Create New Collector | Database Type, Agent, Collector Name | As usual. |
| Connection Details | Hostname or IP Address, Listener Port, Custom JDBC Connection String | |
| Username and Password | Username, Password | User with permissions per User Permissions for Sybase. |
| | CyberArk | Enable CyberArk; download JavaPasswordSDK.jar, rename and copy to lib directory. |
| Advanced Options | Sub-Collectors, Connection Properties, Exclude Databases, Monitor Operating System | See [Configure the Database Agent to Monitor Server Hardware](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-the-database-agent-to-monitor-server-hardware). |

## User Permissions for Sybase

For full Database Visibility functionality, the monitoring user requires:

| Permission type | Permission |
|----------------|------------|
| Role permission | mon_role, sa_role |
| Select permission | master.dbo.monProcessProcedures, monProcessSQLText, monProcessLookup, monProcess, sysmonitors, sysconfigures, monWaitEventInfo |
| Execute permission | sp_sysmon |

To browse objects of a custom database, use the documented command. A sample user creation script is provided in the AppDynamics documentation (replace placeholder with a secure value; DBMon_Agent_User = user under which you run the Database Visibility Agent).

The following configuration parameters must be set to 1 (true) for Sybase ASE: the relevant monitoring flags; and one parameter set to at least 8192 (8kB). Example commands are in the documentation. If that parameter was previously less than 4096, increasing it may require restarting the Sybase ASE instance.

For Sybase >= 15.7 without the full sa_role permission, run the documented alternative commands. If you monitor using reduced permissions, you may encounter: timeslice error in mmap64 or mda_flush_iostats, and thread utilization incorrectly reported (see SAP/AppDynamics knowledge base references in the original documentation).
