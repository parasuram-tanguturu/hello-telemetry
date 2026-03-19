# System Patterns

## Architecture Overview

```
┌─────────────┐
│   Browser   │
└──────┬──────┘
       │ HTTP
       ▼
┌─────────────────────────────────┐
│  Java Servlet (Tomcat)          │
│  Port: 8085                     │
│  - Queries MySQL                │
│  - Calls Python service         │
└──────┬──────────────────────────┘
       │
       ├─────────────────┐
       │                 │
       ▼                 ▼
┌─────────────┐   ┌──────────────┐
│   MySQL     │   │   Python     │
│   Port:3306 │   │   Flask      │
│             │   │   Port:5000  │
└─────────────┘   └──────────────┘
       │                 │
       └────────┬────────┘
                │
                ▼
       ┌─────────────────────┐
       │  OTLP Telemetry     │
       │  (gRPC/HTTP)        │
       └──────────┬──────────┘
                  │
                  ▼
       ┌─────────────────────┐
       │  OTEL Collector     │
       │  Ports: 4317/4318   │
       └──────────┬──────────┘
                  │
                  ├──────────────┐
                  │              │
                  ▼              ▼
         ┌─────────────┐  ┌──────────┐
         │   Jaeger    │  │ Prometheus│
         │  Port:16686 │  │  Port:8888│
         └─────────────┘  └──────────┘
```

## Component Relationships

### Java Application (MyServlet)
- **Type**: Jakarta Servlet (Java 11+)
- **Container**: Apache Tomcat 11.0
- **Responsibilities**:
  - Handle HTTP GET requests at `/results`
  - Query MySQL database (`mytable`)
  - Call Python microservice via HTTP POST
  - Render HTML response
- **Dependencies**: MySQL, Python service

### Python Microservice
- **Framework**: Flask 3.1.0
- **Endpoint**: `/compute_average_age` (POST)
- **Responsibilities**:
  - Receive JSON data with age information
  - Calculate average age
  - Return JSON response
- **Dependencies**: None (standalone service)

### MySQL Database
- **Version**: Latest MySQL
- **Schema**: Single table `mytable` (id, name, age)
- **Initialization**: Scripts in `initdb/init.sql`
- **Usage**: Read-only queries from Java application

### OpenTelemetry Collector
- **Image**: `otel/opentelemetry-collector-contrib:latest`
- **Receivers**: OTLP (gRPC:4317, HTTP:4318)
- **Processors**: Batch processor
- **Exporters**: 
  - Debug (detailed verbosity)
  - OTLP to Jaeger (traces only)
- **Pipelines**: Traces, Metrics, Logs (all configured)

### Jaeger
- **Image**: `jaegertracing/all-in-one:latest`
- **UI**: Port 16686
- **Purpose**: Trace visualization and analysis
- **Ingestion**: Receives traces from OTEL Collector

## Design Patterns

### Microservices Pattern
- Services are independently deployable
- Communication via HTTP/REST
- Each service can be instrumented separately

### Observability Pattern
- **Instrumentation**: Code generates telemetry
- **Collection**: Collector aggregates telemetry
- **Export**: Collector sends to backends
- **Visualization**: Backends provide UI

### Containerization Pattern
- All services run in Docker containers
- Docker Compose orchestrates the stack
- Network isolation via `ht_network`
- Volume persistence for MySQL data

## Data Flow Patterns

### Request Flow
1. HTTP GET → Tomcat → MyServlet.doGet()
2. MyServlet queries MySQL
3. MyServlet POSTs to Python service
4. Python service computes and returns
5. MyServlet renders HTML response

### Telemetry Flow
1. Services generate spans/metrics/logs
2. Telemetry sent to Collector via OTLP
3. Collector batches and processes
4. Collector exports to Jaeger (traces) and Prometheus endpoints (metrics)
5. Debug exporter logs all telemetry

## Key Technical Decisions

1. **OTLP Protocol**: Standard OpenTelemetry protocol for telemetry transport
2. **Batch Processing**: Reduces overhead by batching telemetry before export
3. **Debug Exporter**: Helps with learning/debugging by showing all telemetry
4. **Jaeger All-in-One**: Simplifies setup for learning (not production-ready)
5. **Docker Compose**: Single command to start entire stack
6. **Network Isolation**: All services on `ht_network` for service discovery
