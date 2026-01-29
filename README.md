# AppdyDB

Repositorio de documentación y manuales para el **Agente de Base de Datos (Database Agent)** de Splunk AppDynamics (Database Visibility).

## Contenido

- **[MANUAL_DESPLIEGUE_AGENTE_DATABASE_APPDYNAMICS.md](./MANUAL_DESPLIEGUE_AGENTE_DATABASE_APPDYNAMICS.md)**  
  Manual de despliegue del Database Agent con:
  - Pre-requisitos de **conectividad** (Controller, bases de datos, Events Service)
  - Requisitos de **Java** (1.8–17) y **TLS** (TLS 1.2 o superior; nota sobre TLS antiguo y agentes antiguos)
  - Requisitos de hardware y software
  - Instalación, configuración (`controller-info.xml` y propiedades de sistema)
  - Propiedades del agente para monitoreo (relacionales, Cassandra, PostgreSQL, Sybase, etc.)
  - Habilitación de **SSL/TLS** y **SSH** para el agente
  - Inicio, parada y actualización del agente
  - **Database Visibility Supported Environments:** tabla completa de bases de datos y versiones (Oracle, SQL Server, MySQL, PostgreSQL, SAP HANA, Sybase, DB2, MongoDB, Cassandra, Couchbase, etc.) y sistemas operativos
  - **Add Database Collectors:** pasos para agregar colectores, permisos (AppDynamics y BD), verificación, editar/eliminar/exportar, logs y resolución de problemas
  - **Configuración de colectores por base de datos:** documentación **local** en **[docs/collectors/](./docs/collectors/)** para las 12 bases; en el manual también hay enlaces a “Configure … Collectors” para **todas** las bases (Cassandra, Couchbase, IBM DB2, Microsoft SQL Server, Microsoft Azure, MongoDB, MySQL, **Oracle**, PostgreSQL, SAP HANA, Sybase ASE, Sybase IQ), con resumen específico para **Oracle** (RAC, CDB/PDB, SID/service name, sub-colectores, permisos, Explain plans, Kerberos)
  - **Referencias (sección 13):** enlaces completos a la documentación 24.x que puede ser retirada (Database Visibility, agente, colectores por BD, hardware, API, permisos, eventos, logs, TLS)

El manual se basa en la documentación oficial de AppDynamics 24.x (Database Visibility) y se conserva aquí como referencia ante la posible retirada de esa documentación.

## Documentación local de colectores (Configure … Collectors)

La configuración detallada de cada tipo de base de datos está guardada en el repo:

**[docs/collectors/](./docs/collectors/)** — [README con índice](./docs/collectors/README.md)

| Base de datos | Archivo |
|---------------|---------|
| Oracle | [configure-oracle-collectors.md](./docs/collectors/configure-oracle-collectors.md) |
| Cassandra | [configure-cassandra-collectors.md](./docs/collectors/configure-cassandra-collectors.md) |
| Couchbase | [configure-couchbase-collectors.md](./docs/collectors/configure-couchbase-collectors.md) |
| IBM DB2 | [configure-ibm-db2-collectors.md](./docs/collectors/configure-ibm-db2-collectors.md) |
| Microsoft SQL Server | [configure-microsoft-sql-server-collectors.md](./docs/collectors/configure-microsoft-sql-server-collectors.md) |
| Microsoft Azure | [configure-microsoft-azure-collectors.md](./docs/collectors/configure-microsoft-azure-collectors.md) |
| MongoDB | [configure-mongodb-collectors.md](./docs/collectors/configure-mongodb-collectors.md) |
| MySQL | [configure-mysql-collectors.md](./docs/collectors/configure-mysql-collectors.md) |
| PostgreSQL | [configure-postgresql-collectors.md](./docs/collectors/configure-postgresql-collectors.md) |
| SAP HANA | [configure-sap-hana-collectors.md](./docs/collectors/configure-sap-hana-collectors.md) |
| Sybase ASE | [configure-sybase-collectors.md](./docs/collectors/configure-sybase-collectors.md) |
| Sybase IQ | [configure-sybase-iq-collectors.md](./docs/collectors/configure-sybase-iq-collectors.md) |

## Referencias (documentación 24.x, puede ser retirada)

- [AppDynamics Database Visibility (24.x)](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/)
- [Add Database Collectors](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors)
- [Database Visibility Supported Environments](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/database-visibility-supported-environments)
- [Configure Oracle Collectors](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-oracle-collectors)
- [Configure the Agent Settings for Monitoring Database](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/configure-the-agent-settings-for-monitoring-database)
- [End of Support Notice: Disabling TLS 1.0 and 1.1](https://docs.appdynamics.com/paa/appdynamics-end-of-support-notices/end-of-support-notice-disabling-tls-1-0-and-1-1)

## Licencia

Documentación de referencia; consultar los términos de Splunk AppDynamics para el uso del producto.
