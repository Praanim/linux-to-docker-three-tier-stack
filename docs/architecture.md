# Application Architecture

## Purpose

This document describes the three-tier application before and after containerization. The project began with a bare-metal deployment on one Ubuntu Linux machine and then moved to a manually managed Docker deployment.

## Bare-Metal Architecture

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

All services ran directly on the same Linux host. This phase built understanding of Linux services, listening ports, reverse proxying, database connectivity, systemd, firewall concepts, and troubleshooting.

## Docker Architecture

```text
+----------------------------------------------------------+
| Ubuntu Host                                              |
|  +----------------------------------------------------+  |
|  | Docker                                             |  |
|  |                                                    |  |
|  | frontend_network                                  |  |
|  |   +-------+       +---------+                     |  |
|  |   | Nginx | ----> | Node.js |                     |  |
|  |   | :80   |       | :8080   |                     |  |
|  |   +-------+       +----+----+                     |  |
|  |                            |                       |  |
|  | backend_network            |                       |  |
|  |                    +-------v-------+               |  |
|  |                    | PostgreSQL    |               |  |
|  |                    | :5432         |               |  |
|  |                    +-------+-------+               |  |
|  |                            |                       |  |
|  |                    +-------v-------+               |  |
|  |                    | postgres_data |               |  |
|  |                    | named volume |               |  |
|  |                    +---------------+               |  |
|  +----------------------------------------------------+  |
+----------------------------------------------------------+
```

## Network Boundaries

| Container | frontend_network | backend_network |
|---|---:|---:|
| Nginx | Yes | No |
| Node.js | Yes | Yes |
| PostgreSQL | No | Yes |

Node.js is the middle tier and therefore connects to both networks. Nginx needs access to Node.js but not PostgreSQL. PostgreSQL needs access to Node.js but not Nginx.

## Port Boundaries

| Service | Container port | Host port |
|---|---:|---:|
| Nginx | `80` | `80` |
| Node.js | `8080` | Not published |
| PostgreSQL | `5432` | Not published |

Only Nginx is exposed to the host. This creates one intended entry point but does not by itself provide complete security.

## Request Flow

```text
Client -> Ubuntu host :80 -> Nginx -> Node.js -> PostgreSQL
```

Nginx forwards requests to Node.js through `frontend_network`. Node.js accesses PostgreSQL through `backend_network`.

## Docker DNS

The project uses Docker-resolvable names such as `backend:8080` and `postgres:5432` instead of fixed IP addresses. `backend` may be a container name or network alias, depending on the actual manual Docker command. It is not a Docker Compose service name because Docker Compose was not used.

Container IP addresses can change when containers are recreated, so names are more appropriate for this internal communication.

## Persistent Data

PostgreSQL uses the named volume `postgres_data`:

```text
PostgreSQL container
        |
        v
postgres_data
        |
        v
/var/lib/postgresql
```

The recorded volume source was `/var/lib/docker/volumes/postgres_data/_data`, with destination `/var/lib/postgresql`, local driver, mode `z`, and read/write access.

> Persistence is not the same as backup.

Automated backup and restore were not implemented.

## Why Only Nginx Is Exposed

Nginx is the intended external entry point. Node.js and PostgreSQL are internal services. Keeping their ports unpublished reduces unnecessary host exposure and makes the reverse proxy responsible for application traffic. This should not be presented as a complete production security design.

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

This is a future design. HTTPS, backups, monitoring, and centralized logging are not currently implemented.
