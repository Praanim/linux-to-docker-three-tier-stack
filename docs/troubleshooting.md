# Container Troubleshooting

## Purpose

The project used a temporary shell container to inspect the Node.js image without changing the main application container.

## Debugging Command

```bash
docker run --rm -it --entrypoint /bin/sh my-node-app:1.0
```

## Command Breakdown

| Part | Meaning |
|---|---|
| `docker run` | Creates and starts a container from an image |
| `--rm` | Removes the temporary container after it exits |
| `-i` | Keeps standard input open |
| `-t` | Allocates a terminal |
| `-it` | Provides an interactive terminal session |
| `--entrypoint /bin/sh` | Replaces the normal entrypoint with a shell |
| `my-node-app:1.0` | Image name and tag |

## Why This Helped

The temporary container allowed inspection of files, environment values, installed dependencies, and the image directory structure. It was disposable and did not alter the normal running container.

## Workflow

```text
Application problem
       |
       v
Temporary debugging container
       |
       v
Inspect files/environment/dependencies
       |
       v
Identify problem
       |
       v
Modify Dockerfile/configuration
       |
       v
Rebuild image
       |
       v
Recreate container
       |
       v
Test application
```

The original application problem was not provided, so this document does not invent one.

## Rebuild and Recreate

The exact commands were not included in the project brief:

```bash
<INSERT ACTUAL IMAGE REBUILD COMMAND USED>
<INSERT ACTUAL CONTAINER RECREATION COMMAND USED>
```

Use the commands from shell history or project notes. After recreation, apply the actual verification procedure used in the project:

```bash
<INSERT ACTUAL VERIFICATION COMMAND OR TEST USED>
```

This workflow demonstrates image inspection and container recreation. It does not provide automated health checks, rollback, CI/CD, monitoring, or centralized logging.
