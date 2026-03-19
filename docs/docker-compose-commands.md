# Docker Compose Commands Reference

This document explains common Docker Compose commands and flags used in the hello-telemetry project.

---

## Complete Rebuild Command

**Command:**
```bash
docker compose up --build --force-recreate --remove-orphans --detach
```

### Overview

This command performs a complete, clean rebuild of your Docker Compose stack. It's useful when you want to:
- Rebuild all images from scratch
- Recreate containers even if configuration hasn't changed
- Remove containers from old compose configurations
- Run everything in the background

### Command Flow

```
┌─────────────────────────────────────────────────────────┐
│  docker compose up --build --force-recreate             │
│              --remove-orphans --detach                   │
└─────────────────────────────────────────────────────────┘
                        │
                        ▼
        ┌───────────────────────────────┐
        │  Step 1: Parse docker-compose.yml │
        │  Identify all services         │
        └───────────────────────────────┘
                        │
                        ▼
        ┌───────────────────────────────┐
        │  Step 2: --remove-orphans      │
        │  Find containers from old      │
        │  compose files and remove them │
        └───────────────────────────────┘
                        │
                        ▼
        ┌───────────────────────────────┐
        │  Step 3: --build               │
        │  Rebuild ALL images from       │
        │  Dockerfiles (no cache)        │
        └───────────────────────────────┘
                        │
                        ▼
        ┌───────────────────────────────┐
        │  Step 4: --force-recreate      │
        │  Stop & remove existing        │
        │  containers, create new ones  │
        └───────────────────────────────┘
                        │
                        ▼
        ┌───────────────────────────────┐
        │  Step 5: --detach              │
        │  Start containers in           │
        │  background (no logs)         │
        └───────────────────────────────┘
```

---

## Flag-by-Flag Breakdown

### `--build`

**What it does:** Rebuilds images before starting containers.

**Behavior:**
- Forces Docker to rebuild images from Dockerfiles
- Ignores existing images even if they're up-to-date
- Useful when you've made code or Dockerfile changes

**When to use:**
- After modifying source code
- After changing Dockerfiles
- When you want to ensure fresh builds

**Example:**
```bash
# Without --build: Uses existing images if available
docker compose up

# With --build: Always rebuilds images
docker compose up --build
```

---

### `--force-recreate`

**What it does:** Recreates containers even if their configuration appears unchanged.

**Behavior:**
- Stops and removes existing containers
- Creates new containers from the same images
- Ensures containers start with fresh state

**When to use:**
- When you want to reset container state
- After environment variable changes
- To ensure containers are completely fresh

**Note:** Without this flag, Docker Compose may reuse existing containers if the configuration hasn't changed.

---

### `--remove-orphans`

**What it does:** Removes containers for services that are no longer defined in your current `docker-compose.yml`.

**Behavior:**
- Identifies containers from previous compose runs
- Removes containers for services not in current compose file
- Cleans up "orphaned" containers

**When to use:**
- After removing services from `docker-compose.yml`
- When switching between different compose files
- To clean up containers from old configurations

**Example Scenario:**
```
Previous compose file had: service-a, service-b
Current compose file has:  service-a, service-c

--remove-orphans will remove: service-b containers
```

---

### `--detach` (or `-d`)

**What it does:** Runs containers in the background (detached mode).

**Behavior:**
- Starts all containers and returns control to terminal
- Containers continue running in background
- No log output to terminal

**When to use:**
- When you don't need to see logs immediately
- For production-like deployments
- When you want terminal control back

**Alternative:**
- Without `--detach`, logs stream to your terminal
- Press `Ctrl+C` to stop all containers when running in foreground

**View logs later:**
```bash
# View logs for all services
docker compose logs

# View logs for specific service
docker compose logs ht-tomcat

# Follow logs (like tail -f)
docker compose logs -f
```

---

## Command Breakdown Diagram

```
docker compose up --build --force-recreate --remove-orphans --detach
│           │   │         │                │                │
│           │   │         │                │                └─ Run in background
│           │   │         │                └─ Remove old containers
│           │   │         └─ Always recreate containers
│           │   └─ Rebuild images
│           └─ Start services
└─ Docker Compose command
```

---

## Common Use Cases

| Scenario | Recommended Command | Explanation |
|----------|-------------------|-------------|
| **First time setup** | `docker compose up --build --detach` | Build images and start in background |
| **Code changes** | `docker compose up --build --force-recreate` | Rebuild and recreate containers |
| **Clean slate** | `docker compose up --build --force-recreate --remove-orphans --detach` | Complete reset |
| **Quick restart** | `docker compose up --detach` | Just restart, no rebuild |
| **See logs** | `docker compose up` | No detach flag, see all logs |
| **After removing service** | `docker compose up --remove-orphans` | Clean up old containers |

---

## For hello-telemetry Project

When you run the complete rebuild command on this project, it will:

1. **Remove orphaned containers** - Clean up any containers from old compose configurations
2. **Rebuild images for:**
   - `ht-tomcat` (Java application)
   - `ht-python-service` (Python Flask service)
   - Any other services with build contexts
3. **Force recreate all containers** - Fresh container state for all services
4. **Start in background** - All services run detached

**Services affected:**
- `ht-mysql` - Database
- `ht-tomcat` - Java web application
- `ht-python-service` - Python microservice
- `ht-otel-collector` - OpenTelemetry collector
- `ht-jaeger` - Tracing backend

---

## After Running: Checking Logs and Verifying Services

Since the command runs with `--detach`, containers start in the background without showing logs. Here's how to verify everything started correctly:

### Step 1: Check Container Status

First, verify all containers are running:

```bash
docker compose ps
```

**Expected output:** All services should show `Up` status:
```
NAME                  IMAGE                    STATUS
ht-jaeger            jaegertracing/all-in-one  Up
ht-mysql             mysql:8.0                 Up
ht-otel-collector    otel/opentelemetry-collector Up
ht-python-service    ht-python-service         Up
ht-tomcat            ht-tomcat                 Up
```

### Step 2: Check Logs for Errors

View logs to ensure services started without errors:

```bash
# View all logs (last 100 lines)
docker compose logs --tail=100

# View logs for specific service
docker compose logs ht-tomcat
docker compose logs ht-python-service
docker compose logs ht-otel-collector
```

**What to look for:**
- ✅ **Success indicators:**
  - Java app: "Server startup in [X] milliseconds"
  - Python: "Running on http://0.0.0.0:5000"
  - Collector: "Everything is ready. Begin running and processing data"
  - MySQL: "ready for connections"

- ❌ **Error indicators:**
  - Connection refused errors
  - Port binding failures
  - Database connection errors
  - Missing dependencies

### Step 3: Follow Logs in Real-Time

To monitor logs as they happen:

```bash
# Follow all service logs
docker compose logs -f

# Follow specific service logs
docker compose logs -f ht-tomcat

# Follow multiple services
docker compose logs -f ht-tomcat ht-python-service
```

**Tip:** Press `Ctrl+C` to stop following logs (containers keep running).

### Step 4: Verify Service Health

Test that services are responding:

```bash
# Check Java web app
curl http://localhost:8085

# Check Python service
curl http://localhost:5001/compute_average_age

# Check Jaeger UI (should return HTML)
curl http://localhost:16686

# Check Collector metrics endpoint
curl http://localhost:8888/metrics
```

### Step 5: Check for Common Issues

If services aren't working, check these:

```bash
# Check if containers are actually running
docker compose ps

# Check container resource usage
docker stats

# Check network connectivity between containers
docker network inspect ht_network

# View detailed logs with timestamps
docker compose logs --timestamps
```

### Quick Verification Workflow

```bash
# 1. Run the command
docker compose up --build --force-recreate --remove-orphans --detach

# 2. Wait a few seconds for services to start
sleep 10

# 3. Check status
docker compose ps

# 4. Check logs for any errors
docker compose logs --tail=50

# 5. Test endpoints
curl http://localhost:8085
curl http://localhost:5001/compute_average_age
```

### Log Filtering Tips

```bash
# Show only errors
docker compose logs | grep -i error

# Show logs from last 5 minutes
docker compose logs --since 5m

# Show logs with timestamps
docker compose logs --timestamps

# Show logs for specific time range
docker compose logs --since 2024-01-01T10:00:00 --until 2024-01-01T11:00:00
```

---

## Quick Reference

| Flag | Short Form | Purpose |
|------|-----------|---------|
| `--build` | - | Rebuild images from Dockerfiles |
| `--force-recreate` | - | Always recreate containers |
| `--remove-orphans` | - | Remove containers not in current compose file |
| `--detach` | `-d` | Run in background |

---

## Related Commands

### View running containers
```bash
docker compose ps
```

### Stop all services
```bash
docker compose down
```

### Stop and remove volumes
```bash
docker compose down -v
```

### View logs
```bash
# All services
docker compose logs

# Specific service
docker compose logs ht-tomcat

# Follow logs
docker compose logs -f
```

### Restart specific service
```bash
docker compose restart ht-python-service
```

---

## Notes

- **Typo alert:** The correct flag is `--remove-orphans` (not `--remove-orpahns`)
- **Build context:** The `--build` flag requires Dockerfiles in your project
- **Network:** All services communicate via the `ht_network` Docker network
- **Ports:** Services use container names for internal communication (e.g., `ht-mysql:3306`)

---

## Related Files

- `docker-compose.yml` - Main compose configuration
- `java-app/Dockerfile` - Java application image
- `python-service/Dockerfile` - Python service image
- `tomcat/Dockerfile` - Tomcat server image
