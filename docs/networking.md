# Docker Networking

## Overview

The application uses two user-defined Docker bridge networks:

```text
frontend_network
backend_network
```

The networks were deliberately separated to demonstrate Docker networking and avoid putting all containers on one shared network.

## Membership

| Container | frontend_network | backend_network |
|---|---:|---:|
| Nginx | Yes | No |
| Node.js | Yes | Yes |
| PostgreSQL | No | Yes |

## Bridge Networks

A Docker bridge network allows connected containers to communicate. A user-defined bridge network provides an explicit application network and Docker-managed name resolution.

## frontend_network

Nginx and Node.js are connected to `frontend_network`. Nginx forwards application traffic to Node.js through this network. PostgreSQL is not connected to it.

## backend_network

Node.js and PostgreSQL are connected to `backend_network`. Node.js uses this network for database communication. Nginx is not connected to it.

## Multi-Network Node.js Container

```text
Nginx --frontend_network--> Node.js --backend_network--> PostgreSQL
```

Node.js is the application boundary between the reverse proxy and database, so it requires access to both networks.

## Network Isolation

```text
Nginx -> Node.js       Allowed
Node.js -> PostgreSQL  Allowed
Nginx -> PostgreSQL    Not directly connected
```

This limits direct network membership. It does not make the full system automatically secure.

## Docker DNS

The project uses names such as:

```text
backend:8080
postgres:5432
```

`backend` may refer to a container name or network alias created by the manual Docker setup. It should not be described as a Compose service. Docker-resolvable names are preferred to fixed IP addresses because container IP addresses can change after recreation.

## Conceptual Request Flow

```text
Client
  |
  | Host port 80
  v
Nginx container
  |
  | frontend_network
  | backend:8080
  v
Node.js container
  |
  | backend_network
  | postgres:5432
  v
PostgreSQL container
```

Only Nginx is published to the host. Node.js and PostgreSQL communicate internally through their respective Docker networks.
