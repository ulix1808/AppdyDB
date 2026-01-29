# Configure Couchbase Collectors

> **Fuente (documentación 24.x, puede ser retirada):**  
> https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-couchbase-collectors

---

To monitor Couchbase with Database Visibility, you must have:

- Couchbase >= 4.5
- Database Agent >= 20.x

## Connection Details

| Section | Field | Description |
|---------|-------|-------------|
| Create New Collector | Database Type | The database type that you want to monitor. |
| | Agent | The Database Agent that manages the collector. |
| | Collector Name | The name you want to identify the collector by. |
| Connection Details | Hostname or IP Address | The hostname or IP address of the machine that your database is running on. |
| | Listener Port | The TCP/IP port on which your database communicates with the Database Agent. |
| Username and Password | Username | User with permissions described in User Permissions for Couchbase. |
| | Password | Password of the user. |
| | CyberArk | Enable CyberArk; download JavaPasswordSDK.jar, rename to cyberark-sdk-9.5.jar, copy to agent lib directory. |
| Advanced Options | SSL Connection | Truststore Location, Type (PKCS12 default, SSO), Password. Optional client auth: Keystore Location, Type, Password. Context protocol via `dbagent.couchbase.ssl.context.protocol`; default TLSv1.3. Hostname verification: `dbagent.couchbase.ssl.hostname.verification.enabled` (default false). TLSv1.3 supported on Oracle JDK >= 8u261-b12, AdoptOpenJDK >= 8u262-b10, Azul Platform Prime >= 20.07.0.0; otherwise AppDynamics uses TLSv1.2. |
| | Exclude Databases | Databases to exclude, separated by commas. |
| | Monitor Operating System | See [Configure the Database Agent to Monitor Server Hardware](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-the-database-agent-to-monitor-server-hardware). |

## User Permissions for Couchbase

The monitoring user must have **Full Administrator** or **Read-only Administrator** privileges. Alternatively (Couchbase >= 5), you can assign a combination of **Data Monitoring** and **Query System Catalog** privileges instead of Administrator.
