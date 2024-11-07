## Dapr Setup using Docker

This guide explains how to set up Dapr with Docker and integrates it with your application. Dapr is a powerful distributed application runtime that simplifies microservice development by providing features like service invocation, state management, pub/sub, and secret management.

## Prerequisites

Before proceeding, ensure you have the following installed:

- **Docker**: [Install Docker](https://www.docker.com/get-started)

## Daprize Your Application

### Docker Compose Setup

The `docker-compose.yml` file sets up the necessary services to run Dapr and your application. The following services are defined:

### Services

1. **Placement Service**:
   - **Image**: `daprio/dapr`
   - **Ports**: 3500:3500
   - **Description**: Manages Dapr's state, helping to track which app instances are available for communication.

2. **Redis**:
   - **Image**: `redis:7.4-alpine`
   - **Ports**: 6379:6379
   - **Description**: Used for distributed state or caching within the application.

3. **Zipkin**:
   - **Image**: `openzipkin/zipkin`
   - **Ports**: 9411:9411
   - **Description**: Distributed tracing system that helps monitor and troubleshoot microservices.

4. **Postgres Database (db)**:
   - **Image**: `postgres:16.3-alpine3.20`
   - **Ports**: 5432:5432
   - **Environment**: 
     - `POSTGRES_USER`: postgres
     - `POSTGRES_PASSWORD`: postgres
     - `POSTGRES_DB`: starter_db_dev
   - **Description**: Stores persistent application data.

5. **Starter Service**:
   - **Build context**: `./starter`
   - **Ports**: 3501:3501
   - **Depends On**: Redis, Placement, Zipkin
   - **Command**: `yarn start:dev`
   - **Description**: The core application service, interacting with Redis, Placement, and Zipkin.

6. **Starter Service Dapr**:
   - **Image**: `daprio/daprd:edge`
   - **Ports**: 3501:3501, 3502:3502
   - **Command**: 
    ```bash
      ./daprd \
        --app-id starter-service \  # Specifies the unique identifier for the application. This ID is used for tracking and managing the service.
        --app-port 3000 \  # Defines the application’s HTTP port. The application listens on port 3000.
        --dapr-http-port 3501 \  # Sets the HTTP port (3501) for Dapr to communicate with other services over HTTP.
        --dapr-grpc-port 3502 \  # Configures the gRPC port (3502) for faster communication with lower latency.
        --placement-host-address placement:3500 \  # Points to the Dapr placement service, which manages service state and actor locations.
        --resources-path /dapr/components \  # Specifies the directory containing the Dapr component files (e.g., state stores, pub/sub).
        --config /dapr/config.yaml # Path to the Dapr configuration file, which includes component configurations and other settings.
      
    ```
   - **Environment**: `DAPR_LOG_LEVEL=debug`
   - **Description**: Dapr sidecar for the starter service that provides features like state, pub/sub, and secret management.


## .dapr Folder

The `.dapr` folder contains configuration files for Dapr components. These files are essential for the Dapr sidecar to function properly.

### Contents of `.dapr` Folder:

1. **components**:
   - This directory contains Dapr component files that define how Dapr interacts with services like state stores, pub/sub, and secret management. The following YAML files are included:
     - **configstore.yaml**: Configures the Dapr configuration store, which stores and manages application configuration values.
     - **local-secret-store.yaml**: Defines the local secret store to securely store and retrieve secrets within the application.
     - **pubsub.yaml**: Configures the Dapr Pub/Sub component for message publishing and subscription across services.
     - **statestore.yaml**: Configures the Dapr state store, which is used to persist application state.

2. **config.yaml**:
   - This file contains global Dapr configuration settings, such as app ports, secrets, pub/sub configurations, etc. It is referenced in the `starter-service-dapr` container.

3. **secrets.json**:
   - The `secrets.json` file typically stores sensitive information in key-value pairs, such as API keys, database credentials, and other secrets needed for your application. When Dapr is configured to use a local or cloud-based secret store (e.g., Azure Key Vault, AWS Secrets Manager), it can retrieve these secrets securely.

The `.dapr` folder helps configure and manage Dapr’s behavior for your application’s services. Each component's configuration is defined to facilitate easy integration and deployment in a Dockerized environment.

### Folder structure 

This section outlines the main directories and files in the project, with descriptions for each.

```plaintext
project-root
├── .dapr/
│   ├── components/
│   │   ├── configstore.yaml
│   │   ├── local-secret-store.yaml
│   │   ├── pubsub.yaml
│   │   └── statestore.yaml
│   ├── config.yaml
│   └── secrets.json
├── starter/
│   ├── src/
│   ├── nest-cli.json
│   └── tsconfig.json
├── docker-compose.yml
```

### Run the project 
After placing project into starter folder
- Run `docker compose up` for start project 
- Run `docker compose down` for stop project 
- Run `docker stop $(docker ps -q)` For stop all containers.


### How It Works

1. **Dapr Sidecar Model**: Each microservice in the application runs alongside a Dapr sidecar, which handles communication, state, pub/sub, and other features, allowing for a clean and loosely coupled architecture.
   
2. **Placement**: Dapr uses the Placement service to manage app instances and facilitate communication between them.
   
3. **Redis**: Redis is used for distributed state management, ensuring that different services can cache and share state easily.

4. **Zipkin**: The Zipkin service enables distributed tracing, which helps track and debug requests across services.

5. **Starter Service**: The main application logic runs here, connecting to Redis, Placement, and Zipkin services.

6. **Dapr Sidecar (starter-service-dapr)**: The Dapr sidecar handles the application’s interaction with Dapr’s distributed features like state management, pub/sub, and secret management.
