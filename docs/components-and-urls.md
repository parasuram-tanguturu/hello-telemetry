# Components and URLs Reference

This document provides a comprehensive reference for all components in the hello-telemetry stack and their access URLs.

---

## Components Overview

The hello-telemetry stack consists of the following components:

```
┌─────────────────────────────────────────────────────────────┐
│                    hello-telemetry Stack                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐      ┌──────────────┐      ┌──────────┐  │
│  │   Browser    │──────│  ht-tomcat   │──────│ ht-mysql │  │
│  │  (User)      │      │ (Java App)   │      │(Database)│  │
│  └──────────────┘      └──────┬───────┘      └──────────┘  │
│                                │                            │
│                                │                            │
│                         ┌──────▼───────┐                    │
│                         │ht-python-   │                    │
│                         │service      │                    │
│                         │(Flask API)  │                    │
│                         └─────────────┘                    │
│                                                              │
│  ┌──────────────┐      ┌──────────────┐                    │
│  │ht-otel-      │──────│  ht-jaeger   │                    │
│  │collector     │      │ (Tracing UI) │                    │
│  │(Telemetry)   │      └──────────────┘                    │
│  └──────────────┘                                           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Components and URLs Table

| Component | Container Name | Service Type | Internal Port | External Port | Access URL | Purpose |
|-----------|---------------|--------------|---------------|---------------|------------|---------|
| **Java Web Application** | `ht-tomcat` | Tomcat Server | 8080 | 8085 | `http://localhost:8085/MyWebApp/` | Main web application home page |
| **Java Servlet** | `ht-tomcat` | Servlet | 8080 | 8085 | `http://localhost:8085/MyWebApp/results` | Database results and Python service integration |
| **Python Microservice** | `ht-python-service` | Flask API | 5000 | 5001 | `http://localhost:5001/compute_average_age` | Average age computation endpoint |
| **MySQL Database** | `ht-mysql` | MySQL Server | 3306 | 3306 | `jdbc:mysql://ht-mysql:3306/mydatabase` | Database for application data |
| **Jaeger UI** | `ht-jaeger` | Tracing UI | 16686 | 16686 | `http://localhost:16686` | Distributed tracing visualization |
| **OpenTelemetry Collector** | `ht-otel-collector` | Telemetry Gateway | Multiple | Multiple | See table below | Telemetry collection and export |
| **JMX Monitoring** | `ht-tomcat` | JMX | 9010, 1099 | 9010, 1099 | `service:jmx:rmi://localhost:1099/jndi/rmi://localhost:9010/jmxrmi` | Java application monitoring |

---

## Detailed URL Reference

### Java Web Application URLs

| URL | Description | Context Path | Notes |
|-----|-------------|--------------|-------|
| `http://localhost:8085/MyWebApp/` | Home page (index.jsp) | `/MyWebApp/` | Landing page with link to results |
| `http://localhost:8085/MyWebApp/results` | Database results servlet | `/MyWebApp/results` | Displays database data and calls Python service |

**Important:** The application is deployed at context path `/MyWebApp` because the WAR file is named `MyWebApp.war`. Tomcat uses the WAR filename as the context path.

**Why not root?** Accessing `http://localhost:8085/` returns 404 because there's no ROOT application deployed. To access at root, rename the WAR to `ROOT.war`.

### Python Microservice URLs

| URL | Method | Description | Request Body | Response |
|-----|--------|-------------|--------------|----------|
| `http://localhost:5001/compute_average_age` | POST | Compute average age from data | `{"data": [{"id": 1, "name": "John", "age": 30}, ...]}` | `{"average_age": 35.5}` |

**Internal Access:** From Java app: `http://ht-python-service:5000/compute_average_age` (uses container name, port 5000)

**External Access:** From host machine: `http://localhost:5001/compute_average_age` (port 5001 to avoid macOS ControlCenter conflict)

### OpenTelemetry Collector URLs

| Endpoint | Port | Protocol | Purpose | Access URL |
|----------|------|----------|---------|------------|
| **OTLP gRPC Receiver** | 4317 | gRPC | Receive telemetry via gRPC | `http://localhost:4317` |
| **OTLP HTTP Receiver** | 4318 | HTTP | Receive telemetry via HTTP | `http://localhost:4318` |
| **Prometheus Metrics** | 8888 | HTTP | Expose Collector metrics | `http://localhost:8888/metrics` |
| **Prometheus Exporter Metrics** | 8889 | HTTP | Expose exporter metrics | `http://localhost:8889/metrics` |
| **Health Check** | 13133 | HTTP | Collector health endpoint | `http://localhost:13133` |
| **pprof Extension** | 1888 | HTTP | Performance profiling | `http://localhost:1888` |
| **zpages Extension** | 55679 | HTTP | Debugging and diagnostics | `http://localhost:55679` |

### Jaeger URLs

| Endpoint | Port | Purpose | Access URL |
|----------|------|---------|------------|
| **Jaeger UI** | 16686 | Web interface for traces | `http://localhost:16686` |
| **OTLP gRPC** | 14250 | Receive traces via OTLP | `http://localhost:14250` |
| **HTTP Thrift** | 14268 | Receive traces via HTTP | `http://localhost:14268` |
| **Zipkin** | 9411 | Zipkin-compatible endpoint | `http://localhost:9411` |
| **UDP Thrift** | 6831, 6832 | Legacy UDP endpoints | UDP ports |

### MySQL Database URLs

| Connection Type | URL | Username | Password | Database |
|-----------------|-----|----------|----------|----------|
| **JDBC (from Java)** | `jdbc:mysql://ht-mysql:3306/mydatabase` | `myuser` | `mypassword` | `mydatabase` |
| **External (from host)** | `mysql://localhost:3306/mydatabase` | `myuser` | `mypassword` | `mydatabase` |

---

## Network Communication Flow

### Request Flow: User → Java App → Python Service

This diagram shows the complete bidirectional request/response flow with explicit request/response arrows:

```
                    ┌─────────┐
                    │ Browser │
                    └────┬────┘
                         │
                    ┌────▼────────────────────────────────────┐
                    │ ① REQUEST: HTTP GET                     │
                    │ GET /MyWebApp/results                   │
                    │ Host: localhost:8085                    │
                    └────┬────────────────────────────────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │  ht-tomcat:8080     │
              │  (Java Servlet)     │
              └────┬────────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
        │                     │
   ┌────▼─────┐         ┌─────▼──────┐
   │ ② REQ    │         │  ③ REQ     │
   │          │         │            │
   │ JDBC     │         │ HTTP POST  │
   │ SELECT   │         │ POST /     │
   │ * FROM   │         │ compute_   │
   │ mytable  │         │ average_   │
   │          │         │ age        │
   │          │         │ Body: JSON │
   └────┬─────┘         └─────┬──────┘
        │                     │
        │                     │
   ┌────▼─────┐         ┌─────▼──────┐
   │  ht-mysql│         │ht-python-  │
   │  :3306   │         │service:5000│
   └────┬─────┘         └─────┬──────┘
        │                     │
        │                     │
   ┌────▼─────┐         ┌─────▼──────┐
   │ ② RESP   │         │  ③ RESP    │
   │          │         │            │
   │ ResultSet│         │ HTTP 200   │
   │ [{id:1,  │         │ {"average_ │
   │  name:   │         │  age":35.5}│
   │  "John", │         │            │
   │  age:30},│         │            │
   │  ...]    │         │            │
   └────┬─────┘         └─────┬──────┘
        │                     │
        └──────────┬──────────┘
                   │
              ┌────▼────────────────────────────────────┐
              │ ④ PROCESS: Generate HTML                │
              │ Combines DB results + Python response   │
              └────┬────────────────────────────────────┘
                   │
              ┌────▼────────────────────────────────────┐
              │ ① RESPONSE: HTTP 200 OK                │
              │ Content-Type: text/html                │
              │ <html><body>                            │
              │   <h1>Database Results</h1>            │
              │   <table>...</table>                   │
              │   <h2>Average Age: 35.5</h2>           │
              │ </body></html>                         │
              └────┬────────────────────────────────────┘
                   │
                   ▼
              ┌─────────┐
              │ Browser │
              └─────────┘
```

**Alternative View: Sequential Flow with Bidirectional Arrows**

```
┌─────────┐
│ Browser │
└────┬────┘
     │
     │ ────REQUEST───────────────────────────────────────────────┐
     │ GET /MyWebApp/results                                      │
     │                                                            │
     ▼                                                            │
┌─────────────────┐                                              │
│  ht-tomcat:8080 │                                              │
│  (Java Servlet) │                                              │
└────┬────────────┘                                              │
     │                                                            │
     │ ────REQUEST────────────────┐                               │
     │ JDBC: SELECT * FROM mytable│                               │
     ▼                            │                               │
┌─────────────┐                  │                               │
│  ht-mysql   │                  │                               │
│  :3306      │                  │                               │
└────┬────────┘                  │                               │
     │                            │                               │
     │ ────RESPONSE───────────────┘                               │
     │ ResultSet: [{id:1, name:"John", age:30}, ...]            │
     │                                                            │
     │ ────REQUEST───────────────────────────────────┐           │
     │ HTTP POST /compute_average_age                │           │
     │ Body: {"data": [...]}                        │           │
     ▼                                                │           │
┌──────────────────┐                                 │           │
│ht-python-service │                                 │           │
│:5000             │                                 │           │
└────┬─────────────┘                                 │           │
     │                                                │           │
     │ ────RESPONSE──────────────────────────────────┘           │
     │ JSON: {"average_age": 35.5}                              │
     │                                                            │
     │ ────RESPONSE─────────────────────────────────────────────┘
     │ HTTP 200 OK + HTML page
     │
     ▼
┌─────────┐
│ Browser │
└─────────┘
```

**Bidirectional Flow Breakdown:**

| Step | Direction | Component | Protocol | Details |
|------|-----------|-----------|----------|---------|
| **①** | **Request** | Browser → Java Servlet | HTTP GET | `GET http://localhost:8085/MyWebApp/results` |
| **②** | **Request** | Java Servlet → MySQL | JDBC | `SELECT * FROM mytable` via `jdbc:mysql://ht-mysql:3306/mydatabase` |
| **②** | **Response** | MySQL → Java Servlet | JDBC ResultSet | Returns rows: `[{id:1, name:"John", age:30}, ...]` |
| **③** | **Request** | Java Servlet → Python Service | HTTP POST | `POST http://ht-python-service:5000/compute_average_age`<br>Body: `{"data": [{"id":1, "name":"John", "age":30}, ...]}` |
| **③** | **Response** | Python Service → Java Servlet | HTTP JSON | Returns: `{"average_age": 35.5}` |
| **④** | **Processing** | Java Servlet | Internal | Generates HTML page combining database results and average age |
| **①** | **Response** | Java Servlet → Browser | HTTP | `200 OK` with HTML content |

**Key Points:**
- All HTTP and database calls are **bidirectional** (request + response)
- The Java Servlet acts as an **orchestrator**, making multiple calls and combining results
- Database query (②) and Python service call (③) happen **sequentially** but could be parallelized
- Final HTML response includes data from both MySQL and Python service

### Telemetry Flow: Services → Collector → Jaeger

```
┌─────────────┐
│  Services   │
│ (Java/Python)│
└──────┬──────┘
       │ OTLP (gRPC/HTTP)
       │ Port 4317/4318
       ▼
┌──────────────────┐
│ht-otel-collector │
│  (Port 4317/4318)│
└──────┬───────────┘
       │ Export traces
       │ Port 14250
       ▼
┌─────────────┐
│ ht-jaeger   │
│ (Port 14250)│
└─────────────┘
```

---

## Quick Access Reference

### Most Common URLs

| Purpose | URL |
|---------|-----|
| **Web Application Home** | `http://localhost:8085/MyWebApp/` |
| **Database Results** | `http://localhost:8085/MyWebApp/results` |
| **Jaeger Tracing UI** | `http://localhost:16686` |
| **Collector Metrics** | `http://localhost:8888/metrics` |
| **Python Service** | `http://localhost:5001/compute_average_age` |

### Testing Commands

```bash
# Test Java web app home page
curl http://localhost:8085/MyWebApp/

# Test Java servlet
curl http://localhost:8085/MyWebApp/results

# Test Python service
curl -X POST http://localhost:5001/compute_average_age \
  -H "Content-Type: application/json" \
  -d '{"data": [{"id": 1, "name": "John", "age": 30}]}'

# Test Collector health
curl http://localhost:13133

# Test Collector metrics
curl http://localhost:8888/metrics

# Test Jaeger UI (should return HTML)
curl http://localhost:16686
```

---

## Port Mapping Reference

| Service | Container Port | Host Port | Reason |
|---------|---------------|-----------|--------|
| Tomcat | 8080 | 8085 | Avoid conflicts with other services |
| Python | 5000 | 5001 | Avoid macOS ControlCenter conflict (port 5000) |
| MySQL | 3306 | 3306 | Standard MySQL port |
| Jaeger UI | 16686 | 16686 | Standard Jaeger UI port |
| Collector OTLP gRPC | 4317 | 4317 | Standard OTLP gRPC port |
| Collector OTLP HTTP | 4318 | 4318 | Standard OTLP HTTP port |
| Collector Metrics | 8888 | 8888 | Standard metrics port |
| JMX | 9010, 1099 | 9010, 1099 | JMX monitoring ports |

---

## Context Path Explanation

### Why `/MyWebApp/`?

Tomcat uses the WAR filename to determine the context path:

```
WAR Filename          →  Context Path
─────────────────────────────────────
MyWebApp.war          →  /MyWebApp
ROOT.war              →  / (root)
myapp-v1.0.war        →  /myapp-v1.0
```

**Current Setup:**
- WAR file: `MyWebApp.war` (defined in `pom.xml`)
- Context path: `/MyWebApp`
- Full URLs: `http://localhost:8085/MyWebApp/` and `http://localhost:8085/MyWebApp/results`

**To Deploy at Root:**
1. Change `warName` in `pom.xml` from `MyWebApp` to `ROOT`
2. Update Dockerfile to copy `ROOT.war` instead of `MyWebApp.war`
3. Rebuild and redeploy

---

## Related Files

- `docker-compose.yml` - Port mappings and service configuration
- `java-app/pom.xml` - WAR file name configuration
- `tomcat/Dockerfile` - WAR file deployment
- `otel-collector-config.yaml` - Collector endpoint configuration

---

## Troubleshooting

### 404 Error on Root URL

**Problem:** `http://localhost:8085/` returns 404

**Solution:** Use `http://localhost:8085/MyWebApp/` instead, or rename WAR to `ROOT.war` to deploy at root.

### Port Already in Use

**Problem:** Port conflict when starting services

**Solution:** Check what's using the port:
```bash
lsof -i :<PORT_NUMBER>
```

See `docs/troubleshooting.md` for detailed port conflict resolution.

### Service Not Accessible

**Problem:** Service returns connection refused

**Solution:**
1. Check container status: `docker compose ps`
2. Check logs: `docker compose logs <service-name>`
3. Verify port mappings in `docker-compose.yml`
4. Ensure services are on the same network: `docker network inspect ht_network`
