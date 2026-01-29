# Manual de Despliegue del Agente de Base de Datos AppDynamics (Database Visibility)

> **Nota:** Este manual se basa en la documentación oficial de Splunk AppDynamics 24.x (Database Visibility), que puede ser retirada. Se recomienda conservar esta guía como referencia interna.
>
> **Fuente:** [Configure the Agent Settings for Monitoring Database](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/configure-the-agent-settings-for-monitoring-database) y secciones relacionadas de la documentación AppDynamics 24.x.

---

## 1. Introducción

Database Visibility en Splunk AppDynamics proporciona visibilidad del rendimiento de bases de datos, ayuda a diagnosticar problemas como tiempos de respuesta lentos y carga excesiva, y ofrece métricas sobre actividades como:

- Sentencias SQL o procedimientos almacenados que consumen más recursos
- Estadísticas de procedimientos, sentencias SQL y planes de consulta
- Tiempo en fetch, ordenación o espera de bloqueos
- Actividad del día, semana o mes anterior
- Utilización de disco/CPU/memoria y disponibilidad de la base de datos

Este documento describe los **pre-requisitos**, la **conectividad**, los requisitos de **Java y TLS**, y los pasos para instalar, configurar y desplegar el **Database Agent**.

---

## 2. Pre-requisitos

### 2.1 Conectividad de red

- La máquina donde se ejecuta el Database Agent debe poder alcanzar:
  - El **Controller** de AppDynamics (puerto HTTP/HTTPS).
  - Cada **base de datos** que se quiera monitorear (puerto del listener de la base de datos).
- Si las bases de datos están detrás de un **firewall**, debe permitirse el acceso desde la máquina del agente a los puertos del listener de la base de datos (y opcionalmente SSH o WMI para monitoreo de hardware).
- El ancho de banda aproximado entre agente y controller es de **~300 KB/minuto por colector** (referencia: base de datos grande con ~200 clientes, ~50 esquemas, ~10.000 consultas/minuto). Los valores reales dependen del tipo de base de datos, número de esquemas y consultas únicas.

**Resumen de conectividad:**

| Origen              | Destino     | Puerto / Observación                                      |
|---------------------|------------|-----------------------------------------------------------|
| Database Agent      | Controller | HTTP 8090 (on‑prem) o 80 (SaaS); HTTPS 8181 (on‑prem) o 443 (SaaS) |
| Database Agent      | Bases de datos | Puerto del listener de cada BD (ej. 1521 Oracle, 1433 SQL Server, etc.) |
| Database Agent      | Events Service | Solo entornos **on‑premises**: el Events Service debe estar iniciado antes que el Database Agent |

### 2.2 Java y versión de JVM

- El Database Agent se ejecuta sobre la **Java Virtual Machine (JVM)**.
- **Versiones soportadas:** **Java 1.8 hasta Java 17** (según [Database Visibility System Requirements](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/database-visibility-system-requirements)).
- Se recomienda usar una **JRE/JDK 64 bits** cuando el sistema operativo sea 64 bits para evitar errores en métricas.
- Para **Java &gt; 15** es necesario añadir en la línea de arranque del agente:
  ```text
  --add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/java.security=ALL-UNNAMED
  ```

### 2.3 TLS y compatibilidad con versiones antiguas

- **Requisito actual:** AppDynamics exige **TLS 1.2 o superior** para las comunicaciones seguras. A partir del **1 de abril de 2024**, AppDynamics **deja de aceptar** conexiones con TLS 1.0 y 1.1.
- **Controllers on‑premises ≥ 23.11:** solo aceptan TLS ≥ 1.2. Los agentes que **solo** soporten TLS &lt; 1.2 **dejarán de reportar** al subir el Controller a versiones ≥ 23.11.
- **Si necesitas seguir usando TLS antiguo (1.0/1.1):**
  - Deberás mantener un **Controller en una versión anterior** a 23.11 y un **agente de base de datos compatible** con esa versión.
  - En ese caso, la documentación y los agentes de versiones antiguas son la referencia; este manual se centra en el despliegue actual (TLS 1.2+).
- **Recomendación:** usar **JDK 8 o superior** para garantizar soporte TLS 1.2. **JDK 6 no soporta TLS 1.2** y debe actualizarse.
- **Comprobar que TLS 1.0/1.1 no estén habilitados** en el JVM que usa el agente. Puedes revisar:
  ```bash
  cat $JAVA_HOME/conf/security/java.security | grep "^[^#;]*jdk.tls.disabledAlgorithms"
  # o en instalaciones antiguas:
  cat $JAVA_HOME/jre/lib/security/java.security | grep "^[^#;]*jdk.tls.disabledAlgorithms"
  ```
  Si en la configuración del agente o en variables de entorno se fuerza el uso de TLS 1.0 o 1.1, la JVM podría permitirlos; evita esas sobreescrituras para cumplir con TLS 1.2+.

**Referencia:** [End of Support Notice: Disabling TLS 1.0 and 1.1](https://docs.appdynamics.com/paa/appdynamics-end-of-support-notices/end-of-support-notice-disabling-tls-1-0-and-1-1).

---

## 3. Requisitos de hardware y software

### 3.1 Hardware (máquina del Database Agent)

- **Memoria heap:** al menos **1 GB** de heap para el agente más **512 MB** por instancia de base de datos monitoreada (en bases menos cargadas se puede bajar a 256 MB por instancia).
- **CPU:** 2 GHz o superior.

Ejemplos de tamaño de heap:

| Instancias de BD monitoreadas | Heap sugerido        |
|------------------------------|----------------------|
| 5                            | 1024 + 5×512 = 3.584 MB |
| 20                           | 1024 + 20×512 = 11.264 MB |
| 100                          | 1024 + 100×512 = 52.224 MB |

*Instancia = nodo en Oracle RAC, MongoDB, Couchbase, standalone-collector o sub-colector.*

### 3.2 Software

- **JVM:** Java 1.8 – 17 (ver sección 2.2).
- **Sistema operativo:** consultar [Database Visibility Supported Environments](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/database-visibility-supported-environments). Desde la versión 20.5, **32 bits no está soportado**; se recomienda 64 bits.

### 3.3 Controller y Events Service (solo on‑premises)

- En instalaciones **on‑premises**, el Database Agent requiere el **Events Service**. Debe **iniciarse el Events Service antes** que el Database Agent.
- Espacio en disco del Controller: ~500 MB por colector por día; Events Service ~500 MB por día (por defecto retención 10 días).

---

## 4. Instalación del agente

### 4.1 Descarga y permisos

1. Descargar el paquete adecuado para tu sistema operativo desde el [Cisco AppDynamics Downloads Portal](https://accounts.appdynamics.com/downloads).
2. Descomprimir el ZIP en el directorio de instalación. **No usar espacios** en la ruta de instalación del agente.
3. Iniciar sesión como **administrador** en la máquina donde se instalará el agente. Para evitar problemas de permisos, instalar con el **mismo usuario** que ejecutará el agente (o como administrador). El usuario que ejecute el agente debe tener **permisos de escritura** en el directorio de logs y en el directorio `conf` de la instalación.

### 4.2 Descompresión

**Windows:**  
- Descomprimir `db-agent-x.x.x.zip` en la ruta deseada. Si el sistema bloquea el ZIP: clic derecho → Propiedades → desbloquear.  
- Existen versiones 32 y 64 bits (desde 4.5.2); elegir según el sistema operativo.

**Linux:**  
```bash
unzip db-agent-x.x.x.zip -d <directorio_destino>
```

### 4.3 Múltiples agentes en la misma máquina

Se pueden ejecutar **varios Database Agents** en la misma máquina (cada uno con su propio directorio y, si aplica, propiedades de sistema distintas).

### 4.4 Licencia (solo on‑premises)

Obtener un archivo `license.lic` con licencia de Database Monitoring y colocarlo en el directorio del Controller. El Controller puede tardar uno o dos minutos en detectar la licencia; reiniciar el Controller fuerza la detección.

---

## 5. Configuración básica del agente

Las propiedades se pueden definir en:

- El archivo **`controller-info.xml`** en `<db_agent_home>/conf/`
- O mediante **propiedades de sistema** (`-D`) en el comando de arranque. Las propiedades de sistema **sobrescriben** las de `controller-info.xml`. Las propiedades son **sensibles a mayúsculas/minúsculas**.

### 5.1 Propiedades obligatorias

| Propiedad              | controller-info.xml      | Propiedad de sistema                    | Descripción |
|------------------------|--------------------------|-----------------------------------------|-------------|
| Controller Host        | `<controller-host>`      | `-Dappdynamics.controller.hostName`    | Host o IP del Controller (el mismo que se usa para la UI). |
| Controller Port        | `<controller-port>`      | `-Dappdynamics.controller.port`        | Puerto HTTP o HTTPS del Controller (ver tabla siguiente). |
| Account Access Key     | `<account-access-key>`   | `-Dappdynamics.agent.accountAccessKey` | Clave de acceso de la cuenta para autenticación. |

**Puertos por defecto:**

- **On‑premises:** HTTP 8090, HTTPS 8181.
- **SaaS:** HTTP 80, HTTPS 443.

### 5.2 Ejemplo de `controller-info.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<controller-info>
  <controller-host>192.10.10.10</controller-host>
  <controller-port>8090</controller-port>
  <account-access-key>165e65645-95c1-40e3-9576-6a1424de9625</account-access-key>
  <controller-ssl-enabled>false</controller-ssl-enabled>
  <!-- Para SaaS: descomentar y rellenar account-name -->
  <!-- <account-name></account-name> -->
</controller-info>
```

### 5.3 Propiedades opcionales útiles

- **SSL:** `<controller-ssl-enabled>true</controller-ssl-enabled>` y puerto HTTPS (ver sección 7).
- **Nombre del agente** (necesario si hay varios agentes): `-Ddbagent.name="Nombre del agente"` (por defecto: "Default Database Agent").
- **Unique Host ID:** `-Dappdynamics.agent.uniqueHostId` para particionar lógicamente el host (debe ser único; si hay Machine Agent en el mismo host, debe ser distinto).
- **Multi-tenant / SaaS:** `<account-name>` en `controller-info.xml` o `-Dappdynamics.agent.accountName`.
- **Proxy:** `-Dappdynamics.http.proxyHost`, `-Dappdynamics.http.proxyPort`, `-Dappdynamics.http.proxyUser`, `-Dappdynamics.http.proxyPasswordFile`.

---

## 6. Propiedades del agente para monitoreo de bases de datos

Estas propiedades se configuran en **`controller-info.xml`** en `<db_agent_home>/conf/` (o según documentación de tu versión). Sirven para ajustar el comportamiento del monitoreo.

### 6.1 Bases de datos relacionales (y MongoDB/Couchbase para sampling)

| Nombre de propiedad | Base de datos | Propósito | Valor por defecto |
|---------------------|--------------|-----------|-------------------|
| `dbagent.disable.sybase.ase.system.monitoring` | Sybase | Desactiva el monitoreo de sistema Sybase (p. ej. si ya usas otras herramientas). Si está activo, deja de ejecutar `sp_sysmon` y métricas como Calls per Minute, Server Statistic y Load pueden no ser fiables. | false |
| `dbagent.sampling.interval` | Todas las relacionales; también MongoDB y Couchbase | Intervalo (en segundos) con el que se muestrean consultas: 1, 2, 5, 10, 20 o 30. Elapsed Time se muestra en múltiplos de este valor (ej. si es 2: 2s, 4s, 6s…). | 1 |
| `dbagent.postgresql.database.size.fetch.timeout.in.hrs` | PostgreSQL | Frecuencia (en horas) para recoger estadísticas de tamaño de base de datos. Si &lt; 1, el colector no obtiene la métrica de tamaño. Útil en entornos con muchos objetos. | 1 |

### 6.2 Cassandra

| Nombre de propiedad | Propósito | Valor por defecto |
|---------------------|-----------|-------------------|
| `dbagent.cassandra.sampling.interval.seconds` | Intervalo de muestreo de consultas (1–60 s). El umbral de muestreo se ajusta entre 300 ms y 1000 ms según el intervalo. | 10 |
| `dbagent.cassandra.max.query.execution.time.seconds` | Tiempo máximo de ejecución; las consultas que superen este tiempo no se muestrean. | 300 |
| `dbagent.cassandra.query.sampling.limit` | Número máximo de consultas a muestrear en un periodo. Si es 0, el muestreo de consultas está desactivado. | 100.000 |
| `dbagent.cassandra.dse.skip.slow.query.table.for.sampling` | Usar muestreo desde query traces en lugar de la tabla node_slow_log en DSE Cassandra. | false |
| `dbagent.cassandra.apache.sampling.threshold.in.milliseconds` | Umbral fijo (ms) para Apache Cassandra: se muestrean consultas que superen este tiempo. Valor negativo = umbral dinámico según el intervalo de muestreo. | -1 |

---

## 7. SSL/TLS y SSH

### 7.1 Habilitar SSL entre agente y Controller

- Necesitas el **puerto SSL** del Controller (SaaS: 443; on‑premises: por defecto 8181).
- Crear un **truststore** del agente con la cadena de confianza del certificado del Controller (CA pública, CA interna o certificado autofirmado según tu caso).

Pasos resumidos:

1. Obtener el certificado raíz de la CA que firmó el certificado del Controller. Para **SaaS** se suelen usar certificados DigiCert e IdenTrust (consultar documentación oficial).
2. Crear el truststore con `keytool`:
   ```bash
   keytool -import -alias rootCA -file <certificado_raiz> -keystore cacerts.jks -storepass <contraseña>
   ```
3. Colocar el truststore (p. ej. `cacerts.jks`) en `<db_agent_home>/conf/`.
4. En `controller-info.xml` (o propiedades equivalentes):
   - Puerto del Controller = puerto SSL (443 para SaaS).
   - `<controller-ssl-enabled>true</controller-ssl-enabled>`
   - Contraseña del truststore y ruta del archivo (p. ej. `../../conf/cacerts.jks` si se referencia desde otro contexto).

Proteger el archivo del truststore y de `controller-info.xml` con permisos de sistema de archivos adecuados (solo lectura para el usuario del agente, escritura solo para usuarios privilegiados).

**Utilidad Keystore Certificate Extractor:** en instalaciones on‑premises existe una utilidad para exportar certificados del keystore del Controller sin copiarlo a la máquina del agente; suele estar en `<db_agent_home>/utils/keystorereader/kr.jar`. Consultar la documentación de tu versión para el uso exacto.

### 7.2 SSH (solo Linux, para monitoreo de hardware)

Para que el Database Agent monitoree hardware en hosts remotos (distinto del local), en Linux se puede configurar autenticación SSH (clave RSA/DSA o certificado PEM). Las claves privadas se colocan en `<db_agent_home>/keys/` y en el servidor monitoreado la clave pública en `~/.ssh/authorized_keys`. El puerto SSH del colector por defecto es 22 y puede cambiarse en la configuración del colector (Monitoring Hardware).

---

## 8. Inicio y parada del agente

### 8.1 Events Service (solo on‑premises)

Antes de iniciar el Database Agent en entornos on‑premises, iniciar el Events Service:

- **Linux:** `bin/platform-admin.sh start-events-service`
- **Windows:** `bin\platform-admin.exe start-events-service`

### 8.2 Iniciar el agente

Asegurar que en `controller-info.xml` (o en propiedades de sistema) estén definidos al menos: Controller Host, Controller Port y Account Access Key.

**Con script (recomendado):**

- **Linux:** `./start-dbagent -Xms<min_heap> -Xmx<max_heap> &`
- **Windows:** `start-dbagent.bat -Xms<min_heap> -Xmx<max_heap>`

**Con comando Java (Linux):**

- Java ≤15:
  ```bash
  java -XX:+HeapDumpOnOutOfMemoryError -XX:OnOutOfMemoryError="kill -9 %p" \
    -DLog4jContextSelector=org.apache.logging.log4j.core.async.AsyncLoggerContextSelector \
    -jar db-agent.jar
  ```
- Java &gt;15: añadir `--add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/java.security=ALL-UNNAMED`.

**Windows (64 bits)** con ruta de bibliotecas:
```bat
java -Djava.library.path="<db_agent_home>\auth\x64" -Ddbagent.name="Nombre del agente" -DLog4jContextSelector=org.apache.logging.log4j.core.async.AsyncLoggerContextSelector -jar <db_agent_home>\db-agent.jar
```

Para **aumentar memoria JVM** (varias bases de datos), usar por ejemplo `-Xms1536m -Xmx1536m` (o los valores calculados en la sección 3.1).

### 8.3 Parada del agente

- **Consola:** si el proceso está en primer plano, Ctrl+C. Si está en segundo plano, usar `kill <PID>`.
- **Windows (servicio):** en Servicios de Windows, seleccionar "AppDynamics Database Agent" y detenerlo.

---

## 9. Actualización del agente

- Detener el Database Agent antes de instalar la nueva versión. Todos los agentes que usen la misma ruta de instalación deben estar detenidos.
- Hacer **copia de seguridad** del directorio de instalación y del archivo `conf/controller-info.xml`.
- Descargar el paquete correspondiente desde el [Cisco AppDynamics Downloads Portal](https://accounts.appdynamics.com/downloads).
- Instalar según la guía de instalación; copiar `controller-info.xml` de la copia de seguridad al nuevo `conf/`.
- Si el script de arranque tenía propiedades personalizadas, reañadirlas al nuevo script.
- Para **actualización sin downtime**: si ambas versiones (antigua y nueva) son ≥ 4.2, se puede instalar el nuevo agente en otra ruta, arrancarlo y, cuando aparezca como ACTIVE en el Controller, detener el agente antiguo.

---

## 10. Database Visibility Supported Environments (bases de datos soportadas)

Esta sección resume los entornos y versiones soportados por el Database Visibility Agent según la documentación 24.x. Para permisos concretos por base de datos, ver [Database Visibility Supported Environments](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/database-visibility-supported-environments).

### 10.1 Tabla de bases de datos y versiones

| Base de datos | Soporte Amazon RDS | Versiones |
|---------------|--------------------|-----------|
| **Oracle, Oracle RAC** | Sí | 10g (≥ 10.2), 11g, 12c, 18c, 19c |
| **Microsoft SQL Server** | Sí | 2005, 2008, 2012, 2014, 2016, 2017, 2019, 2022, SQL Azure |
| **Microsoft SQL Server on Linux** | — | Preview; no recomendado para producción |
| **MySQL** | Sí | Todas, incl. 8.0, Percona, MariaDB, Aurora |
| **PostgreSQL** | Sí | Todas, incl. Azure Database for PostgreSQL, Aurora |
| **SAP HANA** | — | ≥ 2.0 SPS 05, ≥ QRC 3/2023 |
| **Sybase ASE** | — | ≥ 15 |
| **Sybase IQ** | — | Todas las versiones |
| **IBM DB2 LUW** | — | 9.x, 10.x, 11.x |
| **MongoDB, MongoDB cluster** | Sí | ≥ 2.6 |
| **Apache Cassandra** | — | ≥ 3.11.4 |
| **Datastax Enterprise (DSE) Cassandra** | — | 5.1, ≥ 6.7.3 |
| **Couchbase** | — | ≥ 4.5 |

**Nota:** Para monitoreo de hardware en MySQL y PostgreSQL en RDS, el vCPU se obtiene según [Amazon RDS Instance Types](https://aws.amazon.com/rds/instance-types/) (modelo vs vCPU) para el modelo de licencia IBL.

### 10.2 Sistemas operativos soportados

- **Windows:** 64 bits (desde 20.5, 32 bits no soportado).
- **Linux:** 32 y 64 bits (recomendado 64 bits con JRE 64 bits).
- **Solaris:** todas las versiones.
- **AIX:** ≥ 6.1.
- **Amazon RDS:** todas las versiones.

Se recomienda usar JRE 64 bits en SO 64 bits para evitar errores en métricas.

---

## 11. Agregar y configurar colectores (Add Database Collectors)

Para que el Database Agent recoja datos, debe añadirse y configurarse un **Database Collector** por cada base de datos a monitorear. El colector es el proceso dentro del agente que recopila métricas de instancias y servidores de base de datos.

**Documentación original (puede ser retirada):** [Add Database Collectors](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors).

### 11.1 Permisos necesarios

- **En la base de datos:** los permisos requeridos dependen del tipo de BD; ver [Database Visibility Supported Environments](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/database-visibility-supported-environments).
- **En Splunk AppDynamics:** el rol del usuario debe tener:
  - Can Create Collectors
  - Can Edit All Collectors
  - Can Delete All Collectors  
  Ver [Database Permissions](https://docs.appdynamics.com/accounts/en/splunk-appdynamics-saas-user-management/manage-custom-roles-for-splunk-appdynamics/database-permissions).

### 11.2 Cómo agregar un colector

1. En la UI del Controller: **Configuration** → **Collectors** → **Add** (+).
2. Completar cada campo del cuadro **Create New Collector** (tipo de BD, agente, nombre del colector, host, puerto, usuario, contraseña, etc.).
3. Si se desea monitorear un **sub-colector** (solo bases relacionales): **Advanced** >> **+Add sub-collector** e indicar host y puerto.
4. Pulsar **OK**.

**Importante:** Al crear el colector, la contraseña se cifra y se guarda; una vez guardada **no puede recuperarse**. Splunk AppDynamics usa la biblioteca interna `secure-credential-store-api` y el algoritmo PKCS5 para el cifrado.

Después de añadir el colector se puede ajustar su configuración y, si aplica, [vincular una base de datos en los mapas de flujo de aplicación](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/monitor-databases-and-database-servers/access-database-visibility-from-application-monitoring-views) a una instancia monitoreada por Database Visibility.

- **Monitoreo de hardware:** [Configure the Database Agent to Monitor Server Hardware](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-the-database-agent-to-monitor-server-hardware).
- **Uso e interpretación:** [Monitor Databases and Database Servers](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/monitor-databases-and-database-servers).

### 11.3 Verificación del colector

Cuando el colector esté en ejecución, en poco tiempo podrás ver datos de actividad histórica. El cuadro de configuración del colector mostrará un icono de colector; desde el menú **Databases** se ve la vista general de todos los colectores. Puede tardar unos minutos en reportar métricas.

### 11.4 Editar, eliminar y exportar

- **Editar:** desde el panel Collectors se pueden editar todos los detalles del colector **excepto** el tipo de plataforma de base de datos. Para los campos específicos de cada BD, ver las páginas de configuración por base de datos (enlaces en la sección 12).
- **Eliminar:** desde el panel Collectors se puede eliminar un colector.
- **Exportar:** **Actions** > **Export Data** genera un CSV con: Collector name, Host name, Port number, Username, Database type, Agent name.

### 11.5 Logs y diagnóstico del agente

- **Agent Dashboard:** clic en el nombre del agente bajo **Agent Name** en la página Collectors.
- Desde ahí se pueden solicitar logs (**Request Agent Log files**) o thread dumps (**Request Agent Thread Dumps**). Ver [Agent Log Files](https://docs.appdynamics.com/appd/24.x/latest/en/application-monitoring/administer-app-server-agents/agent-log-files) y [Request Agent Log Files](https://docs.appdynamics.com/appd/24.x/latest/en/application-monitoring/administer-app-server-agents/request-agent-log-files).

### 11.6 Resolución de problemas

Los colectores mal configurados o que no pueden conectar a la base de datos muestran un error en la página **Databases** y en los dashboards individuales; al pasar el ratón sobre el icono de error se muestra el motivo. Si el colector no reporta métricas tras unos minutos y la base de datos está activa, revisar el panel **Events**. Los [Agent Diagnostic Events](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/monitor-databases-and-database-servers/database-agent-events-reference) pueden indicar contraseña incorrecta o errores de comunicación. Comprobar la configuración del colector y que el Database Agent tenga conectividad y permisos según [Database Monitoring Requirements and Supported Environments](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/database-visibility-supported-environments).

---

## 12. Configuración de colectores por base de datos

Para cada tipo de base de datos existen campos y opciones específicos. La documentación de configuración de cada colector está guardada **en este repositorio** (descargada de la documentación 24.x que puede ser retirada). **Oracle** se detalla un poco más abajo por ser uno de los más usados.

**Índice de todos los colectores (local):** [docs/collectors/README.md](docs/collectors/README.md)

### 12.1 Documentación local “Configure … Collectors”

| Base de datos | Documento local (en el repo) |
|---------------|------------------------------|
| **Oracle** | [Configure Oracle Collectors](docs/collectors/configure-oracle-collectors.md) |
| **Cassandra** | [Configure Cassandra Collectors](docs/collectors/configure-cassandra-collectors.md) |
| **Couchbase** | [Configure Couchbase Collectors](docs/collectors/configure-couchbase-collectors.md) |
| **IBM DB2** | [Configure IBM DB2 Collectors](docs/collectors/configure-ibm-db2-collectors.md) |
| **Microsoft SQL Server** | [Configure Microsoft SQL Server Collectors](docs/collectors/configure-microsoft-sql-server-collectors.md) |
| **Microsoft Azure** | [Configure Microsoft Azure Collectors](docs/collectors/configure-microsoft-azure-collectors.md) |
| **MongoDB** | [Configure MongoDB Collectors](docs/collectors/configure-mongodb-collectors.md) |
| **MySQL** | [Configure MySQL Collectors](docs/collectors/configure-mysql-collectors.md) |
| **PostgreSQL** | [Configure PostgreSQL Collectors](docs/collectors/configure-postgresql-collectors.md) |
| **SAP HANA** | [Configure SAP HANA Collectors](docs/collectors/configure-sap-hana-collectors.md) |
| **Sybase ASE** | [Configure Sybase Collectors](docs/collectors/configure-sybase-collectors.md) |
| **Sybase IQ** | [Configure Sybase IQ Collectors](docs/collectors/configure-sybase-iq-collectors.md) |

Cada archivo incluye al inicio la URL original (24.x) por si la documentación en línea es retirada.

### 12.2 Oracle: resumen de configuración

- **Versiones:** Oracle 10g o superior. Para **Oracle RAC** basta con configurar **un** colector para todo el clúster; se elige cualquier nodo para conectar y el clúster se detecta automáticamente.
- **Oracle 12c PDB:** la consulta de métricas de sistema está disponible desde Oracle Database 12c Release 2 (12.2.0.1).
- **Connection Details:**
  - **SID o Service Name** de la instancia a monitorear.
  - **CDB:** indicar el service name del CDB. **PDB:** indicar el service name del PDB; el service name debe existir en todos los nodos que tengan el PDB.
  - **Custom JDBC Connection String** (opcional). Para Kerberos, usar la opción LDAP/Kerberos en Advanced Options.
- **Sub-colectores:** hasta 29 sub-colectores (30 bases en un clúster personalizado). Parámetros distintos por sub-colector solo vía [Create Collector API](https://docs.appdynamics.com/appd/24.x/latest/en/extend-splunk-appdynamics/splunk-appdynamics-apis/database-visibility-api). No se puede convertir un clúster personalizado en colector standalone.
- **Usuario/contraseña:** el usuario debe tener los permisos indicados en “User Permissions for Oracle” (ver más abajo). Opción **CyberArk** para obtener credenciales.
- **Opciones avanzadas:** Exclude Schemas, SSL Connection (Truststore/Keystore), LDAP/Kerberos, Monitor Operating System (hardware).

**Permisos de usuario para Oracle (10g y posteriores):**

- SELECT_CATALOG_ROLE  
- CREATE SESSION  

Para Oracle 12c+ con Multitenant (CDB/PDB), los mismos roles aplican en el contexto correspondiente. Consultar la documentación para el SQL exacto de creación de usuario.

**Explain plans:** Para generar planes de ejecución desde la ventana de drill-down SQL hace falta una **plan table** en el esquema del usuario de monitoreo, con privilegios INSERT y SELECT. El usuario también necesita privilegios de acceso sobre las tablas/vistas de la consulta que se explica.

**Kerberos:** [Monitor Oracle Databases Using Kerberos Authentication](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-oracle-collectors/monitor-oracle-databases-using-kerberos-authentication).

---

## 13. Referencias a documentación original (24.x)

Estos enlaces corresponden a la documentación que puede ser retirada; se incluyen para trazabilidad y consulta histórica.

### 13.1 Database Visibility y agente

- [Database Visibility (índice)](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/)
- [Configure the Agent Settings for Monitoring Database](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/configure-the-agent-settings-for-monitoring-database)
- [Database Agent Configuration Properties](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/administer-the-database-agent/configure-the-database-agent/database-agent-configuration-properties)
- [Install the Database Agent](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/administer-the-database-agent/install-the-database-agent)
- [Start and Stop the Database Agent](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/administer-the-database-agent/start-and-stop-the-database-agent)
- [Enable SSL and SSH for Database Agent Communications](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/administer-the-database-agent/configure-the-database-agent/enable-ssl-and-ssh-for-database-agent-communications)
- [Database Visibility System Requirements](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/database-visibility-system-requirements)
- [Database Visibility Supported Environments](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/database-visibility-supported-environments)
- [Add Database Collectors](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors)
- [Upgrade the Database Agent](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/administer-the-database-agent/upgrade-the-database-agent)

### 13.2 Colectores por base de datos (Configure … Collectors)

**Documentación local en el repo:** [docs/collectors/README.md](docs/collectors/README.md) — cada “Configure … Collectors” está guardado en `docs/collectors/configure-*-collectors.md`.

Enlaces externos (24.x, pueden ser retirados):
- [Configure Cassandra Collectors](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-cassandra-collectors)
- [Configure Couchbase Collectors](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-couchbase-collectors)
- [Configure IBM DB2 Collectors](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-ibm-db2-collectors)
- [Configure Microsoft SQL Server Collectors](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-microsoft-sql-server-collectors)
- [Configure Microsoft Azure Collectors](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-microsoft-azure-collectors)
- [Configure MongoDB Collectors](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-mongodb-collectors)
- [Configure MySQL Collectors](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-mysql-collectors)
- [Configure Oracle Collectors](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-oracle-collectors)
- [Configure PostgreSQL Collectors](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-postgresql-collectors)
- [Configure SAP HANA Collectors](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-sap-hana-collectors)
- [Configure Sybase Collectors](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-sybase-collectors)
- [Configure Sybase IQ Collectors](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-sybase-iq-collectors)

### 13.3 Otros enlaces útiles

- [Configure the Database Agent to Monitor Server Hardware](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-the-database-agent-to-monitor-server-hardware)
- [Monitor Oracle Databases Using Kerberos Authentication](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/add-database-collectors/configure-oracle-collectors/monitor-oracle-databases-using-kerberos-authentication)
- [Access Database Visibility from Application Monitoring Views](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/monitor-databases-and-database-servers/access-database-visibility-from-application-monitoring-views)
- [Monitor Databases and Database Servers](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/monitor-databases-and-database-servers)
- [Database Agent Events Reference](https://docs.appdynamics.com/appd/24.x/latest/en/database-visibility/monitor-databases-and-database-servers/database-agent-events-reference)
- [Database Permissions (roles)](https://docs.appdynamics.com/accounts/en/splunk-appdynamics-saas-user-management/manage-custom-roles-for-splunk-appdynamics/database-permissions)
- [Database Visibility API / Create Collector API](https://docs.appdynamics.com/appd/24.x/latest/en/extend-splunk-appdynamics/splunk-appdynamics-apis/database-visibility-api)
- [Agent Log Files](https://docs.appdynamics.com/appd/24.x/latest/en/application-monitoring/administer-app-server-agents/agent-log-files)
- [Request Agent Log Files](https://docs.appdynamics.com/appd/24.x/latest/en/application-monitoring/administer-app-server-agents/request-agent-log-files)
- [End of Support Notice: Disabling TLS 1.0 and 1.1](https://docs.appdynamics.com/paa/appdynamics-end-of-support-notices/end-of-support-notice-disabling-tls-1-0-and-1-1)

---

*Documento generado a partir de la documentación oficial de Splunk AppDynamics 24.x (Database Visibility). Si la documentación en línea es retirada, este manual sirve como referencia de despliegue y configuración del Database Agent.*
