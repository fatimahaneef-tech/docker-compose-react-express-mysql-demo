# Compose sample application

React application with a NodeJS backend and a MySQL/MariaDB database

This project is a multi-container application built and deployed using Docker Compose. It contains separate frontend, backend, and database services.

The original application is based on the React + Express + MySQL/MariaDB example from Docker's awesome-compose repository.

## Project structure

.
├── backend/
│ ├── Dockerfile
│ └── ...
├── db/
│ └── password.txt
├── frontend/
│ ├── Dockerfile
│ └── ...
├── compose.yaml
├── .env.example
└── README.md


## Compose configuration

**compose.yaml**

```yaml
services:
  backend:
    build: backend
    ports:
      - 80:80
      - 9229:9229
      - 9230:9230
    ...

  db:
    # We use a mariadb image which supports both amd64 & arm64 architecture
    image: mariadb:10.6.4-focal

    # If you really want to use MySQL, uncomment the following line
    # image: mysql:8.0.27
    ...

  frontend:
    build: frontend
    ports:
      - 3000:3000
    ...
```

The Compose file defines an application with three services: frontend, backend, and db.

When deploying the application, Docker Compose maps port 3000 of the frontend service container to port 3000 of the host as specified in the file. Make sure port 3000 on the host is not already in use.

> **INFO**
>
> For compatibility between AMD64 and ARM64 architectures, MariaDB is used as the database instead of MySQL.
>
> You can still use the MySQL image by uncommenting the following line in the Compose file:
>
> ```yaml
> # image: mysql:8.0.27
> ```

## Configuration improvements

The original Compose configuration was extended with a MariaDB healthcheck and environment-variable documentation.

### Database healthcheck

A healthcheck was added to the `db` service:

```yaml
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
  interval: 10s
  timeout: 5s
  retries: 5
```

The healthcheck allows Docker to monitor whether the MariaDB service is responding instead of relying only on the container's running state.

### Environment variables

An `.env.example` file was added to document the environment variables used by the application without committing actual credentials or secrets.

Example:

DATABASE_DB=example
DATABASE_USER=root
DATABASE_PASSWORD=
DATABASE_HOST=db
NODE_ENV=development


Copy the example file and provide appropriate values for your local environment as needed. Do not commit real passwords or other secrets.

## Docker and Compose skills demonstrated

- Multi-container application deployment
- Docker Compose service orchestration
- Container networking
- Database containers
- Environment variables
- Docker volumes
- Healthchecks
- Service dependencies
- Container status inspection
- Application log inspection
- Compose configuration validation
- Container troubleshooting

## Validate the Compose configuration

Before deploying, validate the Compose configuration:

```bash
docker compose config
```

If the configuration is valid, Docker Compose prints the resolved configuration without starting the application.

## Deploy with Docker Compose

Start the application in detached mode:

```bash
docker compose up -d
```

Example output:

Creating network "react-express-mysql_default" with the default driver
Building backend
Step 1/16 : FROM node:10
---> aa6432763c11
...
Successfully tagged react-express-mysql_frontend:latest
WARNING: Image for service frontend was built because it did not already exist. To rebuild this image you must use docker compose build or docker compose up --build.
Creating react-express-mysql_db_1 ... done
Creating react-express-mysql_backend_1 ... done
Creating react-express-mysql_frontend_1 ... done


## Check container status

Use the following command to inspect the running services:

```bash
docker compose ps
```

You can also use:

```bash
docker ps
```

Expected output should show the frontend, backend, and database containers running. The exact container IDs, creation times, image versions, and generated container names may differ from the example below.

CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
f3e1183e709e react-express-mysql_frontend "docker-entrypoint.s…" 8 minutes ago Up 8 minutes 0.0.0.0:3000->3000/tcp react-express-mysql_frontend_1
9422da53da76 react-express-mysql_backend "docker-entrypoint.s…" 8 minutes ago Up 8 minutes (healthy) 0.0.0.0:80->80/tcp, 0.0.0.0:9229-9230->9229-9230/tcp react-express-mysql_backend_1
a434bce6d2be mariadb:10.6.4-focal "docker-entrypoint.s…" 8 minutes ago Up 8 minutes 3306/tcp react-express-mysql_db_1


The database healthcheck allows the database container's health status to be monitored independently of whether the container itself is running.

## View application logs

View logs from all services:

```bash
docker compose logs -f
```

View logs for an individual service:

```bash
docker compose logs -f frontend
docker compose logs -f backend
docker compose logs -f db
```

## Expected result

After the application starts, navigate to:

http://localhost:3000


in your web browser.

The backend service container has port 80 mapped to port 80 on the host.

Test the backend with:

```bash
curl localhost:80
```

Example response:

```json
{"message":"Hello from MySQL 8.0.19"}
```

The exact response may vary depending on the database image and application configuration.

## Stop and remove the containers

To stop and remove the application containers and the default Compose network:

```bash
docker compose down
```

Example output:

Stopping react-express-mysql_frontend_1 ... done
Stopping react-express-mysql_backend_1 ... done
Stopping react-express-mysql_db_1 ... done
Removing react-express-mysql_frontend_1 ... done
Removing react-express-mysql_backend_1 ... done
Removing react-express-mysql_db_1 ... done
Removing network react-express-mysql_default


## Volumes and persistent database data

The application uses Docker volumes where configured by `compose.yaml` to persist database data across container recreation.

The command:

```bash
docker compose down
```

does not normally remove named volumes.

To remove the containers, network, and associated named volumes, use:

```bash
docker compose down -v
```

⚠️ Use the `-v` option carefully because removing database volumes can delete persisted database data.

## Original project

This project is based on the React + Express + MySQL/MariaDB example from Docker's awesome-compose repository:

https://github.com/docker/awesome-compose/tree/main/react-express-mysql

The original application structure has been retained while the Compose configuration has been extended with:

- A MariaDB healthcheck
- Environment-variable documentation through `.env.example`
- Compose configuration validation
- Additional operational and troubleshooting documentation

## What this project demonstrates

This project demonstrates practical Docker Compose skills, including:

- Deploying a React frontend, Express backend, and MariaDB database as separate services
- Connecting services through Docker Compose networking
- Building application containers from Dockerfiles
- Running and inspecting multi-container applications
- Mapping container ports to the host
- Managing Docker volumes for persistent database data
- Monitoring database availability with a healthcheck
- Documenting environment variables without committing secrets
- Inspecting service logs
- Validating Compose configuration
- Managing service dependencies
- Stopping and removing application containers and networks
- Troubleshooting containerized applications

## Quick command reference

Start the application:

```bash
docker compose up -d
```

Validate the Compose configuration:

```bash
docker compose config
```

Check service status:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs -f
```

Rebuild and start services:

```bash
docker compose up -d --build
```

Stop and remove containers and the network:

```bash
docker compose down
```

Stop and remove containers, network, and named volumes:

```bash
docker compose down -v
```
