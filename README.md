# Collibra Data Quality Web (DQ Web) Docker Installation Guide

## Prerequisites

- Docker installed
- Postgres instance (This example uses a Docker container, not recommended for production)
- Docker pull access (refer to documentation for details on how to obtain)

### Docker Login

```bash
cat service-account.json | docker login -u _json_key --password-stdin https://gcr.io/owl-hadoop-cdh
```

### (Optional) Docker Pull

```bash
docker pull gcr.io/owl-hadoop-cdh/dq-web:2024.11.1-ABDGCSHILM-4213
```

## Steps

### 1. Postgres (Only if not using a pre-existing Postgres instance)

```bash
mkdir -p /path/to/your/postgres/data # Replace with your desired data directory path

docker run -d \
-p 5432:5432 \
-it --memory="4g" \
-e POSTGRES_PASSWORD=password \
-v /path/to/your/postgres/folder:/var/lib/postgresql/data \
postgres
```

### 2. Run DQ Web

```bash
docker run -p 9005:9005 \
-d -it \
--memory="4g" \
--env-file=env-demo.txt \
--add-host=host.docker.internal:host-gateway \
-v /path/to/your/mount/folder/:/tmp/scratch \
-e OWL_VERSION=2024.11.1-ABDGCSHILM-4213 \
gcr.io/owl-hadoop-cdh/dq-web:2024.11.1-ABDGCSHILM-4213
```

### Environment Variables

> **Note:** Environment variables can be passed via the Docker run command or a file (e.g., `env-demo.txt` in this example).  See example env-demo.txt (below). 

### Volume

> **Note:** This step is optional for persisting artifacts (like keys and JKS files). Refer to the documentation for more details. If you do not use a JKS file or need to persist artifacts, you can omit the -v volume section and comment the env variables, if you're not using this volume. 

### License

> **Note:** A license is required for full functionality. Refer to the documentation for details on obtaining a license.

### First Fresh Install

> **Note:** Refer to the documentation for detailed instructions on the first fresh install, including:
> - Providing a license name
> - Creating a tenant
> - Adding a user
> - Adding a connection
> - Adding a license key

## Resources

- [Versions](https://productresources.collibra.com/docs/collibra/latest/Content/DataQuality/Builds.htm)
- [Product Page](https://www.collibra.com/us/en/products/data-quality-and-observability)
- [Release Notes](https://productresources.collibra.com/docs/collibra/latest/Content/DataQuality/release-notes.htm)
- [Supported Deployments](https://productresources.collibra.com/docs/collibra/latest/Content/DataQuality/Installation/to_installation.htm)

## Additional Notes

- For production deployments, it is strongly recommended to use a Postgres service instead of a containerized version.
- The Postgres host accessibility note is particularly important for Mac users (host.docker.internal:host-gateway). Ensure the Postgres host is reachable by the container using the correct hostname or IP address.

## Example env-file
```bash
# Version (or pass in docker run -e OWL_VERSION=), use the -e equivalent in the docker run command
#OWL_VERSION=2024.11.1-ABDGCSHILM-4213

# License
# LICENSE_KEY=<replace-with-your-license>

# Metastore (change accordingly for your postgres instance)
SPRING_DATASOURCE_DRIVER_CLASS_NAME=org.postgresql.Driver
SPRING_DATASOURCE_URL=jdbc:postgresql://host.docker.internal:5432/postgres
SPRING_DATASOURCE_USERNAME=postgres
SPRING_DATASOURCE_PASSWORD=password

# Multi-tenant
MULTITENANTMODE=TRUE
LEGACY_INTEGRATION=FALSE
multiTenantSchemaHub=owlhub

# AI (Mounted Volume Path /tmp/scratch for key, optional to coment when not using AI functionality)  
#GAI_VERTEX_KEY=/tmp/scratch/ai.json
#GAI_VERTEX_LOCATION=us-central1
#GAI_VERTEX_PROJECTID=your-project-id

# SSL (Mounted Volume Path /tmp/scratch for jks, optional to comment this section when SERVER_HTTP_ENABLED=TRUE)
SERVER_HTTP_ENABLED=TRUE
#SERVER_HTTPS_ENABLED=TRUE
#SERVER_SSL_KEY_STORE=/tmp/scratch/keystore.jks
#SERVER_REQUIRE_SSL=TRUE
#SERVER_SSL_KEY_TYPE=PKCS12
#SERVER_SSL_KEY_PASS=<keystore-password>
#SERVER_SSL_KEY_ALIAS=<keystore-alias>

# Connection Pool
SPRING_DATASOURCE_POOL_MAX_WAIT=10000
SPRING_DATASOURCE_POOL_MAX_SIZE=30
SPRING_DATASOURCE_POOL_INITIAL_SIZE=5
SPRING_DATASOURCE_POOL_MAX_IDLE=10
SPRING_DATASOURCE_TOMCAT_MAXIDLE=10
SPRING_DATASOURCE_TOMCAT_MAXACTIVE=20
SPRING_DATASOURCE_TOMCAT_MAXWAIT=10000 
SPRING_DATASOURCE_TOMCAT_INITIALSIZE=5

# Misc Defaults
JAVA_OPTS=-Xms1g -Xmx5g
SPARK_SUBMIT_MODE=native
INSTALL_PATH=/opt/owl
SPARK_HOME=/opt/spark
OWL_BASE=/opt
PACKAGE_MANAGER=apt
INSTALL_INTERACTIVE="DEBIAN_FRONTEND=noninteractive"
EXTRA_JVM_OPTIONS=-Djava.system.class.loader=com.owl.web.config.DQClassLoader
SPRING_FLYWAY_ENABLED=false

# Disable Agent
DISABLENOAGENT=true
```
