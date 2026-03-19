# Troubleshooting Guide

This document tracks common issues encountered during development and their solutions.

---

## Port 5000 Already in Use (macOS)

**Date:** 2024-12-19  
**Issue:** Docker Compose fails with error: `ports are not available: exposing port TCP 0.0.0.0:5000 -> 127.0.0.1:0: listen tcp 0.0.0.0:5000: bind: address already in use`

### Problem Description

When attempting to start the Docker Compose stack, the `ht-python-service` container fails to start because port 5000 is already bound on the host machine.

### Root Cause

On macOS, port 5000 is used by **ControlCenter** (AirPlay Receiver service) for receiving AirPlay content from other Apple devices. This is a system service that runs by default.

**Evidence:**
```bash
$ lsof -i :5000
COMMAND   PID               USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
ControlCe 720 parasuramtanguturu   11u  IPv4 ... TCP *:commplex-main (LISTEN)
ControlCe 720 parasuramtanguturu   12u  IPv6 ... TCP *:commplex-main (LISTEN)

$ ps -p 720 -o comm=,args=
/System/Library/CoreServices/ControlCenter.app/Contents/MacOS/ControlCenter
```

Port 5000 is registered in `/etc/services` as `commplex-main` (both TCP and UDP).

### Solution

Changed the Docker port mapping in `docker-compose.yml` from `5000:5000` to `5001:5000`.

**Before:**
```yaml
ht-python-service:
  ports:
    - "5000:5000"  # Host port 5000 → Container port 5000
```

**After:**
```yaml
ht-python-service:
  ports:
    - "5001:5000"  # Host port 5001 → Container port 5000
```

### Why This Works

1. **Container port unchanged:** The Python service still listens on port 5000 **inside** the container
2. **Inter-container communication unaffected:** The Java app uses `http://ht-python-service:5000`, which resolves to the container's internal port 5000 (via Docker network DNS)
3. **No host port conflict:** The host now uses port 5001, which doesn't conflict with macOS ControlCenter

### Access Points

- **Internal (container-to-container):** `http://ht-python-service:5000` ✅ (unchanged)
- **External (host machine):** `http://localhost:5001` ✅ (changed from 5000)

### Verification Commands

To verify the fix:
```bash
# Check if port 5001 is available
lsof -i :5001

# Start Docker Compose
docker-compose up --build

# Verify Python service is accessible
curl http://localhost:5001/compute_average_age
```

### Alternative Solutions (Not Implemented)

1. **Disable AirPlay Receiver:** System Settings → General → AirDrop & Handoff → AirPlay Receiver → Off
   - Not recommended: Requires disabling a macOS feature
   
2. **Use different container port:** Change Python service to listen on a different port inside container
   - Not recommended: Requires code changes and affects inter-container communication

### Related Files

- `docker-compose.yml` - Port mapping configuration
- `python-service/compute.py` - Python Flask app (listens on port 5000 inside container)
- `java-app/src/main/java/com/bbsod/demo/MyServlet.java` - Uses `http://ht-python-service:5000` for internal calls

---

## How to Diagnose Port Conflicts

If you encounter similar port conflicts in the future:

1. **Identify what's using the port:**
   ```bash
   lsof -i :<PORT_NUMBER>
   ```

2. **Get process details:**
   ```bash
   ps -p $(lsof -ti :<PORT_NUMBER>)
   ```

3. **Check service name:**
   ```bash
   grep <PORT_NUMBER> /etc/services
   ```

4. **Find available ports:**
   ```bash
   # Check if a port is free
   lsof -i :<PORT_NUMBER>
   # If no output, port is available
   ```

---
