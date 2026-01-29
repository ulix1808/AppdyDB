# Configure SAP HANA Collectors

> **Fuente (documentación 24.x, puede ser retirada):**  
> https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-sap-hana-collectors

---

To monitor SAP HANA with Database Visibility, you must run:

- Controller >= 24.5
- Database Agent >= 24.5
- SAP HANA >= 2.0 SPS 05 or >= QRC 3/2023

## Prerequisite

Download the JDBC driver JAR from the SAP website or from the HANA installation package. Rename the file as required and copy the JAR to the Database Agent directory (e.g. lib directory).

## Connection Details

| Section | Field | Description |
|---------|-------|-------------|
| Create New Collector | Database Type, Agent, Collector Name | As usual. |
| Connection Details | Hostname or IP Address, Listener Port, Custom JDBC Connection String | |
| Username and Password | Username, Password | User for connecting and monitoring. |
| Advanced Options | Sub-Collectors | Up to 29 sub-collectors. Different parameters only via [Create Collector API](https://docs.appdynamics.com/appd/24.x/latest/en/extend-splunk-appdynamics/splunk-appdynamics-apis/database-visibility-api). Cannot convert custom cluster to standalone. |
| | Connection Properties | Add or edit JDBC connection properties. |
| | Exclude Databases | Comma-separated. |
| | Monitor Operating System | See [Configure the Database Agent to Monitor Server Hardware](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-the-database-agent-to-monitor-server-hardware). |

## User Permissions for SAP HANA

Create a monitoring user and grant the permissions required to monitor the database (see AppDynamics documentation for the exact SQL/commands for your HANA version).
