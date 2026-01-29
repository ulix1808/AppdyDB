# Configure MongoDB Collectors

> **Fuente (documentación 24.x, puede ser retirada):**  
> https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-mongodb-collectors

---

To monitor MongoDB with Database Visibility, you must run MongoDB >= 2.6.

If you are configuring a collector for a **MongoDB cluster**, you only need to configure one collector for the entire cluster. You can choose any node to connect to, and the entire cluster will be detected automatically.

## Connection Details

- **Custom Connection String:** You can specify a custom connection string. Do not put username and password in the connection string (security risk).
- **SRV Record:** Enable to use SRV record in the connection string.
- **SSL Connection:** Truststore Type (SSO, JKS, PKCS12 default), Keystore/Truststore Location, Password. Optional Enable SSL Client Authentication. Context protocol can be set via system variable; default TLSv1.3, fallback TLSv1.2. TLSv1.3 supported on Oracle JDK >= 8u261-b12, AdoptOpenJDK >= 8u262-b10, Azul Platform Prime >= 20.07.0.0.
- **Exclude Databases:** Comma-separated.
- **Monitor Operating System:** See [Configure the Database Agent to Monitor Server Hardware](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-the-database-agent-to-monitor-server-hardware).

When configuring the collector, encode username and password if they contain special characters.

| Section | Field | Description |
|---------|-------|-------------|
| Create New Collector | Database Type, Agent, Collector Name | As usual. |
| Connection Details | Hostname or IP Address, Listener Port, Custom Connection String, SRV Record | |
| Username and Password | Username, Password | User with permissions per User Permissions for MongoDB. |
| | CyberArk | Enable CyberArk; download and copy JAR to agent lib directory. |
| Advanced Options | SSL Connection, Exclude Databases, Monitor Operating System | |

## User Permissions for MongoDB

- **MongoDB < 2.6.x:** readAnyDatabase and ClusterMonitor built-in roles. For shared clusters, the monitoring user must have access to all shards.
- **MongoDB >= 2.6:** Built-in role **read** in addition to the above as required.
- If you create a new user for monitoring, the user must be created in the **admin** database.
- Configure user roles as shown in the AppDynamics documentation (sample query for roles).
