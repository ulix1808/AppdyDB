# Configure Sybase IQ Collectors

> **Fuente (documentación 24.x, puede ser retirada):**  
> https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-sybase-iq-collectors

---

You can monitor any version of Sybase IQ with Database Visibility.

## Connection Details

- **Sub-Collectors:** Up to 29 sub-collectors. All parameters other than hostname/IP and port are the same as the main collector; different parameters only via [Create Collector API](https://docs.appdynamics.com/appd/24.x/latest/en/extend-splunk-appdynamics/splunk-appdynamics-apis/database-visibility-api). Cannot convert custom cluster to standalone.

| Section | Field | Description |
|---------|-------|-------------|
| Create New Collector | Database Type, Agent, Collector Name | As usual. |
| Connection Details | Hostname or IP Address, Listener Port, Custom JDBC Connection String | |
| Username and Password | Username, Password | User with permissions per User Permissions for Sybase IQ. |
| | CyberArk | Enable CyberArk; download file from CyberArk, rename and copy to agent lib directory. |
| Advanced Options | Sub-Collectors, Connection Properties, Exclude Databases, Monitor Operating System | See [Configure the Database Agent to Monitor Server Hardware](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-the-database-agent-to-monitor-server-hardware). |

## User Permissions for Sybase IQ

To monitor with a **non-DBA** account:

- Grant execution privileges on the required procedures (see AppDynamics documentation for the exact GRANT statements; substitute with the user name under which you run the Database Visibility Agent).
- Grant **DBA** authority to the user, or use a user account with **SERVER OPERATOR** system privilege.
