# Configure MySQL Collectors

> **Fuente (documentación 24.x, puede ser retirada):**  
> https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-mysql-collectors

---

To monitor MySQL with Database Visibility, you must run MySQL >= 5.7.

## Connection Details

| Section | Field | Description |
|---------|-------|-------------|
| Create New Collector | Database Type, Agent, Collector Name | As usual. |
| Connection Details | Hostname or IP Address, Listener Port, Custom JDBC Connection String | e.g. `jdbc:mysql://...` |
| Username and Password | Username, Password | User with permissions per User Permissions for MySQL. |
| | CyberArk | Enable CyberArk; download JavaPasswordSDK.jar, rename to cyberark-sdk-9.5.jar, copy to lib directory. |
| Advanced Options | Sub-Collectors | Up to 29 sub-collectors. Different parameters only via [Create Collector API](https://docs.appdynamics.com/appd/24.x/latest/en/extend-splunk-appdynamics/splunk-appdynamics-apis/database-visibility-api). Cannot convert custom cluster to standalone. |
| | Connection Properties | Add or edit JDBC connection properties. |
| | Exclude Databases | Comma-separated. |
| | Monitor Operating System | See [Configure the Database Agent to Monitor Server Hardware](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-the-database-agent-to-monitor-server-hardware). |

## User Permissions for MySQL

The MySQL user must have **SELECT**, **PROCESS**, and **SHOW DATABASES** on all databases.

For metrics Slave_io_running, Slave_sql_running, Seconds_behind_master, SQL_Delay: **REPLICATION CLIENT** privilege.

Example (substitute DBMon_Agent_User, host, password):

```sql
CREATE USER 'DBMon_Agent_User'@'host' identified by 'password';
GRANT SELECT, PROCESS, SHOW DATABASES on *.* to 'DBMon_Agent_User'@'host' identified by 'password';
GRANT REPLICATION CLIENT ON *.* to 'DBMon_Agent_User'@'host';
FLUSH privileges;
```

Set `max_allowed_packet` to 1073741824 on the server.
