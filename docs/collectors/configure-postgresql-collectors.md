# Configure PostgreSQL Collectors

> **Fuente (documentación 24.x, puede ser retirada):**  
> https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-postgresql-collectors

---

You can monitor any version of PostgreSQL with Database Visibility.

## Connection Details

| Section | Field | Description |
|---------|-------|-------------|
| Create New Collector | Database Type, Agent, Collector Name | As usual. |
| Connection Details | Hostname or IP Address, Listener Port, Custom JDBC Connection String | e.g. `jdbc:postgresql://...`. For Azure Active Directory: `jdbc:sqlserver://:port_number;database=;authentication=ActiveDirectoryPassword;User=;Password=;encrypt=true;trustServerCertificate=false;` |
| Username and Password | Username, Password | User with permissions per User Permissions for PostgreSQL. |
| | CyberArk | Enable CyberArk; download JavaPasswordSDK.jar, rename to cyberark-sdk-9.5.jar, copy to lib directory. |
| Advanced Options | Sub-Collectors | Up to 29 sub-collectors. Different parameters only via Create Collector API. Cannot convert custom cluster to standalone. |
| | Connection Properties | Add or edit JDBC properties. For Azure AD: property `authentication`, property `database`. |
| | EnterpriseDB | Click if your PostgreSQL is an [EnterpriseDB](http://www.enterprisedb.com/) distribution. |
| | Exclude Databases | Comma-separated. |
| | Monitor Operating System | See [Configure the Database Agent to Monitor Server Hardware](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-the-database-agent-to-monitor-server-hardware). |

## Set up PostgreSQL for Monitoring

### User Permissions

You must be a superuser. Create a non-superuser and grant monitoring via SECURITY DEFINER functions so non-superusers can view `pg_stat_activity` and `pg_stat_statements`:

1. Create view for pg_stat_activity:

```sql
CREATE FUNCTION get_sa()
RETURNS SETOF pg_stat_activity LANGUAGE sql AS
$$ SELECT * FROM pg_catalog.pg_stat_activity; $$
VOLATILE SECURITY DEFINER;

CREATE VIEW pg_stat_activity_allusers AS SELECT * FROM get_sa();
GRANT SELECT ON pg_stat_activity_allusers TO public;
```

2. Create view for pg_stat_statements:

```sql
CREATE FUNCTION get_querystats()
RETURNS SETOF pg_stat_statements LANGUAGE sql AS
$$ SELECT * FROM pg_stat_statements; $$
VOLATILE SECURITY DEFINER;
CREATE VIEW pg_stat_statements_allusers AS SELECT * FROM get_querystats();
GRANT SELECT ON pg_stat_statements_allusers TO public;
```

The monitoring user must be able to connect remotely from the Database Agent machine.

### Enable pg_stat_statements

As superuser: `create extension pg_stat_statements`. Restart the database if creating the extension for the first time.

### Validate the Setup

As the monitoring user (e.g. appduser), run:

1. `SELECT * FROM pg_stat_activity_allusers`
2. `SELECT * FROM pg_stat_statements_allusers`

If both return output, the setup is successful.

## Set up pgvector for Monitoring

pgvector is an extension for vector storage and similarity search. Prerequisites: `pg_stat_statements` loaded (in `shared_preload_libraries` in postgresql.conf), vector extension installed, PostgreSQL >= 14.

Enable vector metrics when launching the Database Agent:

| Property | Description |
|----------|-------------|
| `dbagent.postgres.vector.extension.metrics.enabled` | Enables PostgreSQL vector metrics. |
| `dbagent.postgres.vector.tables` | Comma-separated list of vector tables to monitor. |
