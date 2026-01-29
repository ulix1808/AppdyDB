# Configure Cassandra Collectors

> **Fuente (documentación 24.x, puede ser retirada):**  
> https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-cassandra-collectors

---

This page provides configuration details for Cassandra collectors.

## Prerequisites

- Apache Cassandra >= 3.11.4
- Datastax Enterprise (DSE) Cassandra 5.1, >= 6.7.3

## Connection Details

| Field | Description |
|-------|-------------|
| Database Type | The database type that you want to monitor. |
| Agent | The Database Agent that manages the collector. |
| Collector Name | The name you want to identify the collector by. |
| Hostname or IP Address | The hostname or IP address of the machine that your database is running on. |
| Listener Port | The TCP/IP port on which your database communicates with the Database Agent. |
| JMX Port | Port for remote JMX connection (optional for DSE Cassandra). See JMX Configurations. |
| JMX Username / JMX Password | JMX credentials. See JMX configurations. |
| Username / Password | Database user with permissions per [User Permissions for Apache Cassandra](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-cassandra-collectors/query-capture-configurations-for-apache-cassandra) and [User Permissions for DSE Cassandra](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-cassandra-collectors/query-capture-configurations-for-dse-cassandra). |
| CyberArk | Enable CyberArk; download JavaPasswordSDK.jar, rename to cyberark-sdk-9.5.jar, copy to agent lib directory. |
| SSL Connection | Truststore Location, Type (PKCS12 default, SSO), Password. Optional client auth: Keystore Location, Type, Password. Context protocol via `dbagent.cassandra.ssl.context.protocol`; default TLSv1.3, fallback TLSv1.2. |
| Monitor Operating System | See [Configure the Database Agent to Monitor Server Hardware](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-the-database-agent-to-monitor-server-hardware). |

## Cassandra Configurations

### Query Capture Configurations

- [Apache Cassandra](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-cassandra-collectors/query-capture-configurations-for-apache-cassandra)
- [DSE Cassandra](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-cassandra-collectors/query-capture-configurations-for-dse-cassandra)

### JMX Configurations for Apache/DSE Cassandra

1. Open `cassandra-env.sh`.
2. To enable JMX authentication: under `if [ "$LOCAL_JMX" = "yes" ]; then` (local) or the corresponding else (remote), set:
   ```bash
   JVM_OPTS="$JVM_OPTS -Dcom.sun.management.jmxremote.authenticate=true"
   ```
3. Set path to credentials file:
   ```bash
   JVM_OPTS="$JVM_OPTS -Dcom.sun.management.jmxremote.password.file=PATH TO FILE"
   ```
4. In `jmxremote.password`, set credentials (same as collector config): `username password`

For ReadOnly JMX access, use `jmxremote.access` (see [Enabling JMX authentication and authorization](https://docs.datastax.com/en/cassandra/3.0/cassandra/configuration/secureJmxAuthentication.html)).

If DB and JMX use different SSL keystores: use DB SSL in Controller collector config, and system properties for JMX SSL:

```bash
-Djavax.net.ssl.keyStore="..." -Djavax.net.ssl.keyStorePassword=... -Djavax.net.ssl.trustStore="..." -Djavax.net.ssl.trustStorePassword=...
```

- JMX SSL disabled: `-Dcassandra.jmx.ssl.enabled=false`
- JMX SSL enabled: `-Dcassandra.jmx.ssl.enabled=true -Djavax.net.ssl.trustStore="..." -Djavax.net.ssl.trustStorePassword=...`

For multiple Cassandra clusters with different root CAs: create one truststore containing all root CAs and Controller certificates (e.g. multiple `keytool -import`). See [Enable SSL and SSH for Database Agent Communications](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/administer-the-database-agent/configure-the-database-agent/enable-ssl-and-ssh-for-database-agent-communications). Otherwise use different Database Agents per cluster.
