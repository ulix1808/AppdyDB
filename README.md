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
  - Agregar colectores (bases de datos a monitorear)
  - Referencias a la documentación oficial 24.x (que puede ser retirada)

El manual se basa en la documentación oficial de AppDynamics 24.x (Database Visibility) y se conserva aquí como referencia ante la posible retirada de esa documentación.

## Referencias

- [AppDynamics Database Visibility (24.x)](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/)
- [Configure the Agent Settings for Monitoring Database](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/configure-the-agent-settings-for-monitoring-database)
- [End of Support Notice: Disabling TLS 1.0 and 1.1](https://docs.appdynamics.com/paa/appdynamics-end-of-support-notices/end-of-support-notice-disabling-tls-1-0-and-1-1)

## Licencia

Documentación de referencia; consultar los términos de Splunk AppDynamics para el uso del producto.
