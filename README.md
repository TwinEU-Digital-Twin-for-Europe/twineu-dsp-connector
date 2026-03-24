# TwinEU DSP Connector (v2)

## Introduction
[<img src="images/TRUE_Connector_Logo.png" alt="True Connector" width="25%">](https://github.com/Engineering-Research-and-Development/dsp-true-connector)&nbsp;&nbsp;&nbsp;&nbsp;
[<img src="images/idsa-sign-component-certification-small.png" alt="IDS certified" width="5%">](https://internationaldataspaces.org/offers/certification/)

This project started from and extends the [Engineering DSP True Connector](https://github.com/Engineering-Research-and-Development/dsp-true-connector), a general purpose Data Space Connector and open-source project developed by ENG, supporting IDSA Data Space Protocol standard (current version 2024-1). The connector is an open-source solution, designed to enable self-determined data sharing while ensuring compliance with regulations such as GDPR. Initially focused on the manufacturing domain, the TRUE Connector has proven its versatility across diverse sectors including circular economy, energy, smart buildings, and agri-food domains. It has received IDS certification.

<br />

[<img src="images/OneNet.svg" alt="OneNet Project" width="15%">](https://github.com/european-dynamics-rnd/OneNet)&nbsp;&nbsp;&nbsp;&nbsp;
[<img src="images/Interstore-logo.svg" alt="Interstore Project" width="15%">](https://github.com/Horizont-Europe-Interstore/Data-Space-Connector)

Furthermore, the project builds upon the work carried out with the [OneNet connector](https://github.com/european-dynamics-rnd/OneNet), developed within the [OneNet project](https://www.onenet-project.eu/) and extended in [Interstore Project](https://github.com/Horizont-Europe-Interstore/Data-Space-Connector). Specifically, the following elements were adopted and extended from the OneNet connector: 

* the OneNet Middleware for centralized services such as Identity Management and Service Catalogue (extended)
* the Semantic Vocabulary with more than 60 standardized services for the energy domain
* [New open-source advanced GUI](https://github.com/TwinEU-Digital-Twin-for-Europe/onenet-dsp-api)
* Integration of External Service via REST APIs and Push mechanisms

## New Features 

Starting from version 2, the TwinEU DSP Connector includes support for the IDSA Data Space Protocol (current version: 2024-1).

## Prerequisites
The deployment process involves the use of Docker containers. The use of Docker guarantees not only an easy deployment process and total portability of the solution, but also a high level of scalability of the released applications.
The hardware and operating system prerequisites are:
* A 64bit 2-core processor
*	8GB RAM Memory
*	50GB of disk space or more

The software prerequisites include:
*	Linux or Windows (preferably Server edition) Operative System (OS);
*	docker and docker-compose;

TwinEU Connector software and its components are available via the Docker containers. Firstly, the Docker platform has to be downloaded and installed accordingly to the OS of the server to host the deployment.
For the correct installation of docker and docker-compose, please refer to the official guides: https://docs.docker.com/get-docker/

## TwinEU DSP Connector v2 installation on Docker
To proceed with the installation of TwinEU Connector, the user must use the docker folder of the github repository that contains all the necessary configuration.

1.	The first step is to clone this repository https://github.com/TwinEU-Digital-Twin-for-Europe/data-space-connector in a specific folder (e.g. *twineu-framework*), by typing:
```
mkdir twineu-framework
cd twineu-framework
git clone https://github.com/TwinEU-Digital-Twin-for-Europe/onenet-dsp-connector.git
```

2.	There is the *docker-compose.yml* file located under the docker folder that contains all the configuration of the TwinEU DSP Connector containers. Go to that folder by typing the command:
```
cd twineu-dsp-connector/docker
```

3.	Update and Start the containers with the below commands:
```
docker compose pull
docker compose up -d
```
The default configuration, recommended for connector testing, simulates a complete environment with 2 connectors within the same docker.
If, however, you want to use the connector in a real environment, or test it with 2 independent machines, please read section [Deploy a single connector instance](#deploy-a-single-connector-instance).

4.	To show logs use the command:
```
docker compose logs -f
```
Alternatively you can use dozzle UI to access the logs of each container. Open the following url on your browser :
```
http://localhost:8085
```

5.	If no errors are seen, this means that TwinEU DSP Connector was successfully deployed on your premises.

To stop all the containers use:
```
docker compose down
```

> [!WARNING]
> Setting up the connector on a server will expose ports to the internet; ensure this is done carefully to maintain security.

### Hints

#### Deploy a single connector instance
The configuration present within the docker-compose.yml simulates a complete environment with 2 connectors.
By starting this configuration both connectors are started within the same docker.

If, however, you want to use the connector in a real environment, or test it with 2 independent machines, you can deploy a single connector instance.
For this purpose, an additional docker-compose file (*docker-compose-single.yml*) is provided that launches a single connector instance.
You can start this configuration using the commands:
```
docker compose -f docker-compose-single.yml up -d
```
and stop with:
```
docker compose -f docker-compose-single.yml down
```
This configuration is recommended for installing a connector production node because it allows you to launch a single connector instance and save hardware resources.

---

#### Optional proxy configuration (using nginx)
Optionally you can use nginx as proxy.

The following example defines the nginx service to start (from the official image), the exposed ports and the volumes to mount.

```
services:
  nginx:
   image : nginx:latest
   ports :
       - "8081:8081"
       - "80:80"
       - "443:443"
   volumes:
       - ./ssl:/etc/nginx/ssl
       - ./nginx.conf:/etc/nginx/conf.d/default.conf
``` 

The *nginx.conf* contains header management, SSL configuration and location directives necessary for the correct functioning of the connector.

In particular, for each service to be exposed, a location type directive must be defined with the following configurations (please replace the _**path**_ and _**uri**_ placeholders with your own values):
```
  location /<path> {
    proxy_pass <uri>;
    proxy_redirect off;
    proxy_set_header    Upgrade     $http_upgrade;
    proxy_set_header    Connection  "upgrade";
    proxy_set_header Host $host:$server_port;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Ssl on;
    proxy_set_header  X-Forwarded-Proto  https;
    rewrite ^/<path>/(.*)$ /$1 break;
  }
``` 

For further information, refer to the [Official Nginx Guide](https://nginx.org/en/docs/).

---

#### Optional Configuration: Traefik as HTTPS Reverse Proxy in Docker
This section describes how to configure Traefik as an HTTPS reverse proxy in a Docker environment.

Enabling HTTPS in Traefik requires configuring TLS certificates. It is possible to use valid certificates issued by a Certification Authority (CA) by mounting them as a volume in Traefik. Alternatively, Traefik natively supports integration with Let's Encrypt for automatic TLS certificate generation and renewal. This guide adopts the latter approach.

For more information, please refer to the [Official Traefik Let's Encrypt Documentation](https://doc.traefik.io/traefik/reference/install-configuration/tls/certificate-resolvers/acme/).

##### Docker Compose Service Configuration
It is recommended to expose the DSP Connector services (connector-a, connector-b), the One-Net DSP API instances (onenet-dsp-api-a, onenet-dsp-api-b), the DSP Connector user Interface (dsp-connector-ui), and the monitoring tool (dozzle) via Traefik.

For simplicity, this guide demonstrates how to configure Traefik for a single connector instance. However, the same configuration principles can be easily extended to support multiple instances.

From the project folder, go into the */docker* directory and edit the *docker-compose-single.yml* file to add the necessary Traefik configuration and labels for the services intended to be routed.

###### 1. Traefik Configuration Using Dedicated HTTPS Ports
In this configuration, each internal service is exposed externally through Traefik on its own unique HTTPS port.

###### _1.1. Traefik generic configuration_
Add the following Traefik service definition to *docker-compose-single.yml*, under the services section:

```yaml
services:
  traefik:
    image: traefik:v3.4
    container_name: traefik
    restart: unless-stopped
    networks:
      - network-a
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - ./traefik/acme.json:/acme.json
    ports:
      - "80:80"
      - "443:443"
      - "8445:8445"  # onenet-dsp-api-a
      - "8446:8446"  # connector-a
      - "8447:8447"  # dsp-connector-ui
      - "8448:8448"  # dozzle
      - "9443:9443"  # minio
    command:
      - "--api.dashboard=true"
      - "--api.insecure=false"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--entrypoints.websecure.http.tls=true"
      - "--entrypoints.onenet-dsp-api-a.address=:8445"
      - "--entrypoints.onenet-dsp-api-a.http.tls=true"
      - "--entrypoints.connector-a.address=:8446"
      - "--entrypoints.connector-a.http.tls=true"
      - "--entrypoints.dsp-connector-ui.address=:8447"
      - "--entrypoints.dsp-connector-ui.http.tls=true"
      - "--entrypoints.dozzle.address=:8448"
      - "--entrypoints.dozzle.http.tls=true"
      - "--entrypoints.minio.address=:9443"
      - "--entrypoints.minio.http.tls=true"
      - "--certificatesresolvers.resolver.acme.httpchallenge=true"
      - "--certificatesresolvers.resolver.acme.httpchallenge.entrypoint=web"
      - "--certificatesresolvers.resolver.acme.email=<email>"
      - "--certificatesresolvers.resolver.acme.storage=/acme.json"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.entrypoints=websecure"
      - "traefik.http.routers.traefik.rule=Host(`<domain`)"
      - "traefik.http.routers.traefik.service=api@internal"
      - "traefik.http.routers.traefik.tls=true"
      - "traefik.http.routers.traefik.tls.certresolver=resolver"
      # Dashboard basic authentication with user "admin" and password "admin"
      - "traefik.http.routers.traefik.middlewares=auth"
      - "traefik.http.middlewares.auth.basicauth.users=admin:$$apr1$$CMWeiHUf$$TvQKxOv1dtRYaoh.5mH5o1"
      - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"
      - "traefik.http.routers.http-catchall.rule=HostRegexp(`{host:.+}`)"
      - "traefik.http.routers.http-catchall.entrypoints=web"
      - "traefik.http.routers.http-catchall.middlewares=redirect-to-https"
      - "traefik.http.routers.http-catchall.priority=1"
```
Details:
* Each internal HTTP service should be exposed on a dedicated HTTPS port through Traefik, by configuring separate entrypoints for each service.
* The port numbers used here are only examples; any available ports may be used.
* For each of these ports, a corresponding Traefik entrypoint must be configured so that Traefik listens on that port and routes traffic directly and securely to the appropriate internal service.
* Traefik is configured with a secured dashboard, accessible only on the configured domain or host, and protected with basic authentication.
* A global HTTP-to-HTTPS redirect middleware is configured to ensure all incoming HTTP requests are automatically redirected to HTTPS, improving security.
* Replace `<domain>` with the domain or host where the service will be accessible.
* Replace `<email>` with the email address that must be used for Let's Encrypt certificate registration.
* Before starting the Traefik container, ensure that a `/traefik` directory exists with a writable `acme.json` file, required for Traefik to store Let's Encrypt certificates.

###### _1.2. Exposing Services via Traefik_
To make your services accessible through Traefik using dedicated HTTPS ports, add the following labels to each service you want to publish:
```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.<service-name>.entrypoints=<service-name>"
  - "traefik.http.routers.<service-name>.rule=Host(`<domain>`)"
  - "traefik.http.routers.<service-name>.tls=true"
  - "traefik.http.services.<service-name>.loadbalancer.server.port=<internal-service-port>"
```
Details:
* For a simple HTTPS configuration with Traefik, services should be reachable through a Docker network.
* Replace `<service-name>` with a unique identifier for your service (e.g., connector-a). This must match the Traefik entrypoint configured for the service.
* Replace `<internal-service-port>` with the internal port the service listens on inside the container.
* Replace `<domain>` with the domain or host where the service will be accessible.

Notes:

For *MinIO* service, the unwanted automatic redirect to the console can be avoided by setting the environment variable MINIO_BROWSER=off and by modifying the startup command so that *MinIO* runs exclusively in API mode, disabling the *MinIO Console* and exposing only the S3 endpoint for storage operations, as shown below:

```yaml
  minio:
    image: minio/minio
    container_name: minio
    networks:
      - network-a
    volumes:
      - minio_data:/data
    environment:
      - MINIO_ROOT_USER=minioadmin
      - MINIO_ROOT_PASSWORD=minioadmin
      - MINIO_BROWSER=off
    command: server /data
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.minio.entrypoints=minio"
      - "traefik.http.routers.minio.rule=Host(`twineu.duckdns.org`)"
      - "traefik.http.routers.minio.tls=true"
      - "traefik.http.routers.minio.tls.certresolver=resolver"
      - "traefik.http.services.minio.loadbalancer.server.port=9000"
```

##### _1.3. Run and access the services_
After running the application, if the example configuration has been applied, the services will be securely accessible via HTTPS at the following ports:

* `https://<domain>:8445` → TwinEU DSP API A
* `https://<domain>:8446` → Connector A
* `https://<domain>:8447` → DSP Connector UI
* `https://<domain>:8448` → Dozzle Dashboard
* `https://<domain>:9443` → MinIO
* `https://<domain>` → Traefik Dashboard (requires basic auth)

##### _1.4. Docker Compose Example_

A complete example for this configuration is available in the */docker* directory in the *docker-compose-single-traefik.yml* file.
The `<domain>` and `<email>` placeholders must be replaced with the appropriate values.

###### 2. Traefik Configuration Using URL Path Prefixes
This configuration exposes internal services externally via Traefik on the standard HTTPS port (443), routing requests based on distinct URL path prefixes.

###### _2.1. Traefik generic configuration_
Add the following Traefik service definition to *docker-compose-single.yml*, under the services section:

```yaml
  traefik:
    image: traefik:v3.4
    container_name: traefik
    restart: unless-stopped
    networks:
      - network-a
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - ./traefik/acme.json:/acme.json
    command:
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--entrypoints.websecure.http.tls=true"
      - "--certificatesresolvers.resolver.acme.httpchallenge=true"
      - "--certificatesresolvers.resolver.acme.httpchallenge.entrypoint=web"
      - "--certificatesresolvers.resolver.acme.email=<email>"
      - "--certificatesresolvers.resolver.acme.storage=/acme.json"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.http-catchall.rule=Host(`<domain>`)"
      - "traefik.http.routers.http-catchall.entrypoints=web"
      - "traefik.http.routers.http-catchall.middlewares=redirect-to-https"
      - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"
```
Details:
* A global HTTP-to-HTTPS redirect middleware is configured to ensure all incoming HTTP requests are automatically redirected to HTTPS, improving security.
* Replace `<domain>` with the domain or host where the service will be accessible.
* Replace `<email>` with the email address that must be used for Let's Encrypt certificate registration.
* Before starting the Traefik container, ensure that a `/traefik` directory exists with a writable `acme.json` file, required for Traefik to store Let's Encrypt certificates.
* The Traefik dashboard is not exposed in this configuration.

###### _2.2. Exposing Services via Traefik_
To expose your services through Traefik using a Path Prefix and HTTPS, add the following labels to each service you want to publish:
```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.<service-name>.rule=Host(`<domain>`) && PathPrefix(`/<path-prefix>`)"
  - "traefik.http.routers.<service-name>.entrypoints=websecure"
  - "traefik.http.routers.<service-name>.tls=true"
  - "traefik.http.services.<service-name>.loadbalancer.server.port=<internal-service-port>"
  - "traefik.http.routers.<service-name>.middlewares=strip-<service-name>"
  - "traefik.http.middlewares.strip-<service-name>.stripprefix.prefixes=/<path-prefix>"
```
Details:
* Replace `<service-name>` with a unique identifier for your service (e.g., connector-a).
* Replace `<path-prefix>` with the path prefix under which the service will be reachable.
* Replace `<domain>` with the domain where the service will be accessible.
* Replace `<internal-service-port>` with the internal port the service listens on inside the container.
* *websecure* is the Traefik entrypoint configured for HTTPS (commonly port 443).
* Services are labeled with routing rules matching the appropriate path prefixes and associated middlewares.
* Middlewares are used to strip the path prefixes before forwarding requests to internal services.

Notes:

Some services, like Minio, Dozzle or certain GUI frontends, do not natively support being served under a path prefix due to how they handle internal routing and resources, which may cause issues with resource loading and redirects.

For these services, additional configuration or alternative approaches are needed to ensure proper functionality; it is generally recommended to expose them using a dedicated domain, a dedicated HTTPS port, or restrict access to internal networks only.

For *Dozzle*, the issue can be resolved by setting the environment variable `DOZZLE_BASE` to match the path prefix, as shown below:
```yaml
dozzle:
  image: amir20/dozzle:latest
  container_name: dozzle
  environment:
    - DOZZLE_BASE=/dozzle
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
  networks:
    - network-a
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.dozzle.rule=Host(`<domain>`) && PathPrefix(`/dozzle`)"
    - "traefik.http.routers.dozzle.entrypoints=websecure"
    - "traefik.http.routers.dozzle.tls=true"
    - "traefik.http.services.dozzle.loadbalancer.server.port=8080"
```

For the *DSP Connector UI*, it is preferable to serve the service directly on the root path (/).
The example below shows the corresponding Traefik labels configuration:
```yaml
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.dsp-connector-ui.rule=Host(`<domain>`) && PathPrefix(`/`)"
    - "traefik.http.routers.dsp-connector-ui.entrypoints=websecure"
    - "traefik.http.routers.dsp-connector-ui.tls=true"
    - "traefik.http.services.dsp-connector-ui.loadbalancer.server.port=8080"
```
###### _2.3. Run and access the services_
After running the application, the services will be securely accessible via HTTPS under the configured path prefixes:

* `https://<domain>` --> DSP Connector UI
* `https://<domain>/<path-prefix>` --> Corresponding service

<br />

For more detailed information and advanced configurations, check out the official Traefik documentation:
* [Traefik Documentation Homepage](https://doc.traefik.io/traefik/)
* [Getting Started with Docker and Traefik](https://doc.traefik.io/traefik/getting-started/docker/)

### Login & Connector Settings
A Graphical User Interface is available together with TwinEU Connector. It can be accessed through the url:
```
http://localhost:8081/
```

1.	You should see the login interface, sign-in using the username & password that you received from the TwinEU Data Space Framework administrators.

2.	Navigate to the connector settings by the sidebar menu & define the urls of your TwinEU DSP Api Url and Connector Url. Those 2 connector applications are running on the containers that you installed, so the urls must be configured accordingly as shown below.

#### TwinEU DSP Api Url
The url must be http://your_ip_where_the_containers_are_installed:30001/api
In the default testing configuration two local-api are exposed to the URLs http://localhost:30001/api or  http://localhost:30002/api, one for connector a and one for connector b.


#### Connector Url
In the default testing configuration the connectors are exposed to the URLs http://connector-a:8080 or  http://connector-b:8080.

**Warning: In a real environment, the connector URL must be exposed with a public IP address or DNS on the internet to allow all consumers to access the data provider.**

### Using the connector
In the standard test environment with two connectors, you should have two users, each configured with a different connector. For example:

user1 (provider):
- Local API URL: http://&lt;*server-ip-or-dns*&gt;:30001/api
- Endpoint connector URL: http://connector-a:8080

user2 (consumer):
- Local API URL: http://&lt;*server-ip-or-dns*&gt;:30002/api
- Endpoint connector URL: http://connector-b:8080

You can use *user1* as the provider and *user2* as the consumer or viceversa.

### Environment Configuration

Inside the */docker* project folder, there is an *.env* environment configuration file. This file allows you to set all Back End configurations of the TwinEU DSP Connector. 

#### Connector ENDPOINT API URL
In a real (production) environment, each connector must be exposed with a public IP address or DNS name so that all consumers can access the data provider.

Update the following environment variables accordingly: 

```
CONNECTOR_A_ENDPOINT_API_URL=https://<public_ip_address:port or domain>
CONNECTOR_B_ENDPOINT_API_URL=https://<public_ip_address:port or domain>
```

**Important**
- Make sure the URL is reachable from outside your local network.
- If HTTPS is not configured, you may temporarily use HTTP for testing (e.g., http://<public_ip_address>:port), but HTTPS is strongly recommended in production.

#### S3 External Presigned Endpoint Configuration
In the test environment, when running two connectors on the same machine, the default configuration uses the internal MinIO address:
```
S3_EXTERNAL_PRESIGNED_ENDPOINT = **http://minio:9000**
```
In a production environment, you need to publicly expose MinIO (or use Amazon S3) and update this environment variable accordingly:

```
S3_EXTERNAL_PRESIGNED_ENDPOINT = **https://<public_ip_address or domain>:<public_port>**
```

**Important**
- If you use a proxy (e.g., Nginx), do not configure MinIO under a subpath (e.g., https://domain.com/minio/), because MinIO does not support subpaths.
- Use a dedicated domain or subdomain instead (e.g., https://minio.domain.com/).
- If you are using HTTP instead of HTTPS, you can also expose MinIO directly on the port (e.g., http://<public_ip_address>:9000).

#### Push Mechanism Flow
The Push mechanism flow is enabled by default. To disable the function use the env variable below.
```
PUSH_ENABLED = **true|false**
```
The Push URI can be configured in Push service creation interface.

The same interface can also be used to configure authentication for external service calls. The main supported methods are:
- No Authentication
- Basic Authentication (username and password)
- API Key (name and value), which can be sent either as a URL parameter or as an HTTP header

#### NATS And Kafka Plugins
In order to enable/disable plugin functionalities in the GUI you must set these variables:
- NATS_ENABLED --> For NATS 
- KAFKA_ENABLED --> For Kafka

## Source code
The source code can be found in the following repositories::
* [Graphical User Interface](https://github.com/TwinEU-Digital-Twin-for-Europe/onenet-dsp-ui)
* [DSP Connector API](https://github.com/TwinEU-Digital-Twin-for-Europe/onenet-dsp-api)
* [DSP True Connector](https://github.com/Engineering-Research-and-Development/dsp-true-connector)
