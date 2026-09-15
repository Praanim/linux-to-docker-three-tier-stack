# Linux to Docker Three-Tier Stack

A hands-on DevOps project that began as a three-tier application deployed directly on one Ubuntu Linux machine and was later containerized manually with Docker.

The application consists of:

- Nginx as the reverse proxy
- Node.js as the application server
- PostgreSQL as the database

The project focuses on Linux services, networking, reverse proxying, Docker containers, user-defined bridge networks, Docker DNS, persistent storage, environment variables, and container troubleshooting.

> This repository documents the completed project as it currently exists. TLS, automated backups, health checks, CI/CD, monitoring, centralized logging, cloud deployment, and Kubernetes are future improvements and are not completed features.

## Overview

The project was built in two phases. First, Nginx, Node.js, and PostgreSQL ran directly on one Ubuntu Linux machine. This made it possible to understand services, ports, reverse proxying, and application/database connectivity without adding container abstraction.

Second, the same architecture was containerized manually with Docker. Nginx, Node.js, and PostgreSQL run in separate containers. Two user-defined bridge networks separate frontend traffic from backend database traffic.

Node.js connects to both `frontend_network` and `backend_network`. Nginx connects only to `frontend_network`, while PostgreSQL connects only to `backend_network`.

## Project Evolution

### Phase 1: Bare-Metal Linux

```text
Client
   |
   v
Nginx :80
   |
   v
Node.js :8080
   |
   v
PostgreSQL :5432
```

This phase covered Ubuntu/Linux, Nginx, Node.js, PostgreSQL, reverse proxying, ports, listening services, networking, systemd, firewall concepts, connectivity, and troubleshooting.

### Phase 2: Dockerized Architecture

```text
+----------------------------------------------------------+
| Ubuntu Host                                              |
|                                                          |
|  Docker                                                  |
|   |                                                      |
|   +-- frontend_network                                   |
|   |      +---------+       +-----------+                 |
|   |      | Nginx   | ----> | Node.js   |                 |
|   |      | :80     |       | :8080     |                 |
|   |      +---------+       +-----+-----+                 |
|   |                               |                       |
|   +-- backend_network             |                       |
|          +------------------------+                       |
|          |                                                |
|          v                                                |
|   +-------------+       +----------------+                |
|   | PostgreSQL  | ----> | postgres_data  |                |
|   | :5432       |       | named volume  |                |
|   +-------------+       +----------------+                |
+----------------------------------------------------------+
```

The project intentionally did not use Docker Compose. Containers, networks, ports, and the volume were managed manually to understand Docker fundamentals before using higher-level configuration tools.

## Docker Network Architecture

| Container | `frontend_network` | `backend_network` |
|---|---:|---:|
| Nginx | Yes | No |
| Node.js | Yes | Yes |
| PostgreSQL | No | Yes |

Node.js is connected to both networks because it receives requests from Nginx and communicates with PostgreSQL. PostgreSQL is limited to `backend_network` because it does not need direct connectivity with Nginx. Nginx is limited to `frontend_network` because it forwards requests to Node.js but does not need direct database access.

## Request Flow

```text
Client
  -> Ubuntu host :80
  -> Nginx container
  -> Node.js container
  -> PostgreSQL container
```

The client sends HTTP traffic to host port `80`. The host forwards it to Nginx. Nginx forwards the application request through `frontend_network` to Node.js. Node.js uses `backend_network` to communicate with PostgreSQL when database access is required.

## Port Exposure

| Service | Container Port | Host Port | Purpose |
|---|---:|---:|---|
| Nginx | `80` | `80` | Receives client HTTP traffic |
| Node.js | `8080` | Not published | Internal application traffic |
| PostgreSQL | `5432` | Not published | Internal database traffic |

Only Nginx is exposed to the host. This reduces unnecessary host-level exposure and makes Nginx the intended entry point. Not publishing ports alone does not make the system fully secure.

## Docker DNS / Service Discovery

User-defined Docker networks provide Docker-managed name resolution. The project uses names such as:

```text
http://backend:8080
postgres:5432
```

`backend` may be a container name or a network alias, depending on the manual Docker command used. It should not be called a Docker Compose service because Docker Compose was not used. Names are preferable to hardcoded container IP addresses because IP addresses can change when containers are recreated.

## Persistent Storage

PostgreSQL uses the named Docker volume `postgres_data`.

```text
PostgreSQL container
        |
        v
+----------------------+
| postgres_data        |
| Docker named volume  |
+----------+-----------+
           |
           v
/var/lib/postgresql
```

Recorded volume information:

| Property | Value |
|---|---|
| Type | `volume` |
| Name | `postgres_data` |
| Source | `/var/lib/docker/volumes/postgres_data/_data` |
| Destination | `/var/lib/postgresql` |
| Driver | `local` |
| Mode | `z` |
| Read/write | `true` |

A named volume keeps database data separate from the PostgreSQL container filesystem. Recreating or removing the container does not inherently remove the named volume.

> Persistence is not the same as backup.

Automated backup and restore are not implemented.

## Environment Configuration

The project uses a local `.env` file for environment-specific values. The real file must not be committed because it may contain credentials. The repository includes `.env.example` with placeholders, and `.gitignore` excludes `.env`.

```dotenv
PORT=8080
DB_HOST=postgres
DB_PORT=5432
DB_NAME=<database_name>
DB_USER=<database_user>
DB_PASSWORD=<database_password>
```

## Nginx Reverse Proxy

Nginx is the externally exposed entry point. It receives HTTP traffic on port `80` and forwards requests to Node.js through Docker networking. The supplied project information identifies the upstream form as `http://backend:8080`.

The exact Nginx configuration used in the project was not supplied. The repository therefore includes a clearly marked placeholder rather than inventing directives.

## Container Troubleshooting

The project used this temporary debugging command:

```bash
docker run --rm -it --entrypoint /bin/sh my-node-app:1.0
```

- `docker run` creates and starts a container.
- `--rm` removes it after exit.
- `-i` keeps standard input open.
- `-t` allocates a terminal.
- `--entrypoint /bin/sh` starts a shell instead of the normal entrypoint.
- `my-node-app:1.0` is the image and tag.

The workflow was: inspect the image, identify a problem, modify the Dockerfile or configuration, rebuild the image, recreate the container, and test again. The original problem was not provided and is not invented here.

## Docker Concepts Demonstrated

| Concept | Demonstration |
|---|---|
| Dockerfile | Used to build the Node.js image |
| Images | Used to create application and debugging containers |
| Containers | Separate Nginx, Node.js, and PostgreSQL containers |
| Port publishing | Only Nginx published host port `80` |
| Bridge networks | `frontend_network` and `backend_network` |
| Docker DNS | Name-based internal communication |
| Multi-network containers | Node.js connected to both networks |
| Network isolation | Nginx and PostgreSQL not directly connected |
| Named volume | PostgreSQL uses `postgres_data` |
| Environment variables | Local `.env` configuration |
| Container debugging | Temporary interactive shell container |
| Image rebuilding | Rebuilt after Dockerfile/configuration changes |
| Reverse proxy | Nginx forwards traffic to Node.js |

## Security Considerations

Implemented practices include exposing only Nginx, keeping Node.js and PostgreSQL unpublished on the host, separating frontend and backend network membership, using environment variables, and excluding `.env` from Git.

This is not a complete production security implementation. TLS, certificate management, vulnerability scanning, automated patching, and broader hardening are not completed.

## What I Learned

The project connected Linux services, ports, networking, reverse proxying, containers, Docker networks, Docker DNS, persistence, and troubleshooting in one progression. Starting with bare-metal deployment made the containerized version easier to understand because the underlying request path was already clear.

## Limitations

- Single Ubuntu host
- No TLS/HTTPS
- No automated database backup or restore
- No health checks
- No centralized logging
- No vulnerability scanning
- No CI/CD or automated deployment
- No monitoring
- No cloud deployment
- No orchestration or Kubernetes

## Future Improvements

1. Add TLS/HTTPS termination.
2. Add health checks and structured logging.
3. Implement and test PostgreSQL backup and restore.
4. Add Docker security hardening and Trivy scanning.
5. Add GitHub Actions CI/CD.
6. Add monitoring with metrics and dashboards.
7. Evaluate cloud deployment and Infrastructure as Code.
8. Evaluate Kubernetes only after the simpler workflow is well understood.

## How to Run

The exact manual Docker commands were not supplied in the project brief. They must be copied from the project's shell history or notes rather than guessed.

The intended manual sequence is:

```text
1. Create frontend_network.2. Create backend_network.
3. Create postgres_data.
4. Build the Node.js image.
5. Create the PostgreSQL container.
6. Create the Node.js container.
7. Connect Node.js to both networks.
8. Create the Nginx container.
9. Verify containers, networks, ports, storage, and application behavior.
```


## Completed Features

- Three-tier application
- Bare-metal Linux deployment
- Nginx reverse proxy
- Node.js application server
- PostgreSQL database
- Separate Docker containers
- Two user-defined Docker networks
- Multi-network Node.js container
- Docker DNS/service discovery
- Network segmentation
- Only Nginx exposed to the host
- PostgreSQL named volume
- Environment variable configuration
- Interactive container troubleshooting
- Docker image rebuilding

## Future Architecture

```text
Internet
   |
   v
HTTPS
   |
   v
Nginx
   |
   v
Node.js
   |
   v
PostgreSQL
   |
   +--> Backup / Monitoring / Logging
```

This is a future design only. HTTPS, automated backups, monitoring, and centralized logging are not currently implemented.

## Lessons Learned

1. Linux deployment clarifies how services and ports relate.
2. Nginx provides one intended application entry point.
3. Separate containers make service boundaries visible.
4. User-defined networks make communication boundaries explicit.
5. Node.js can bridge two network segments when necessary.
6. Docker DNS is more stable than hardcoded container IP addresses.
7. A named volume separates data from the container lifecycle.
8. Persistence does not replace backups.
9. Temporary shell containers help inspect image contents.
10. Manual Docker management builds a foundation for later automation.

## License

No license has been selected because none was provided in the project brief. Add a license only after choosing the terms under which you want to publish the project.
