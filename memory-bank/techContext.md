# Technical Context

## Technology Stack

### Java Application
- **Language**: Java (JDK 11+)
- **Framework**: Jakarta Servlet API 6.1.0
- **Container**: Apache Tomcat 11.0
- **Build Tool**: Maven
- **Dependencies**:
  - MySQL Connector/J 9.1.0
  - Apache HttpClient 4.5.13
  - JSON library 20240303
  - Jakarta JSTL 2.0.0
- **Packaging**: WAR file (`MyWebApp.war`)

### Python Service
- **Language**: Python 3
- **Framework**: Flask 3.1.0
- **HTTP Server**: Werkzeug 3.1.0
- **Port**: 5000
- **Endpoints**: `/compute_average_age` (POST)

### Database
- **Type**: MySQL (latest)
- **Port**: 3306
- **Database**: `mydatabase`
- **User**: `myuser` / `mypassword`
- **Root Password**: `rootpassword`
- **Schema**: `mytable` (id INT, name VARCHAR(255), age INT)

### Observability Stack
- **Collector**: OpenTelemetry Collector Contrib (latest)
- **Protocol**: OTLP (gRPC and HTTP)
- **Backend**: Jaeger All-in-One (latest)
- **Metrics**: Prometheus-compatible endpoints

## Development Setup

### Prerequisites
- Docker and Docker Compose
- Maven (for building Java WAR file)
- Java JDK 11+ (for local development)

### Build Process

#### Java Application
```bash
cd java-app
mvn clean package
```
- Creates `MyWebApp.war` in `tomcat/` directory
- WAR file is copied into Tomcat image during Docker build

#### Python Service
- No build step required
- Dependencies installed via `pip install -r requirements.txt` in Dockerfile

### Docker Images

#### Tomcat Image (`tomcat/Dockerfile`)
- Base: `tomcat:11.0`
- Copies WAR file to `/usr/local/tomcat/webapps/`
- Exposes ports: 8080 (HTTP), 9010 (JMX), 1099 (RMI)

#### Python Service Image (`python-service/Dockerfile`)
- Base: `python:3`
- Installs Flask and Werkzeug
- Runs `compute.py` on port 5000

## Port Configuration

| Service | Port | Purpose |
|---------|------|---------|
| Tomcat | 8085 | Web application (mapped from 8080) |
| Tomcat | 9010 | JMX monitoring |
| Tomcat | 1099 | RMI for JMX |
| Python | 5000 | Flask API |
| MySQL | 3306 | Database |
| OTEL Collector | 4317 | OTLP gRPC receiver |
| OTEL Collector | 4318 | OTLP HTTP receiver |
| OTEL Collector | 8888 | Prometheus metrics |
| OTEL Collector | 13133 | Health check |
| OTEL Collector | 1888 | pprof extension |
| OTEL Collector | 55679 | zpages extension |
| Jaeger | 16686 | UI |
| Jaeger | 6831/6832 | UDP (legacy) |
| Jaeger | 14250 | gRPC |
| Jaeger | 14268 | HTTP |

## Network Configuration

- **Network Name**: `ht_network`
- **Service Discovery**: Services reference each other by container name:
  - `ht-mysql` (database)
  - `ht-tomcat` (Java app)
  - `ht-python-service` (Python service)
  - `ht-otel-collector` (collector)
  - `ht-jaeger` (Jaeger)

## Configuration Files

### OpenTelemetry Collector (`otel-collector-config.yaml`)
- **Receivers**: OTLP (gRPC + HTTP)
- **Processors**: Batch
- **Exporters**: 
  - Debug (detailed verbosity)
  - OTLP to Jaeger (traces only, endpoint: `http://ht-jaeger:4317`)
- **Pipelines**: Traces, Metrics, Logs (all configured)

### Docker Compose (`docker-compose.yml`)
- Defines all services
- Sets up volumes for MySQL persistence
- Configures environment variables
- Manages service dependencies

### Database Initialization (`initdb/init.sql`)
- Creates `mytable` schema
- Inserts sample data (Alice, Bob, Charlie)

## Technical Constraints

1. **JMX Configuration**: JMX configured for localhost only (not production-ready)
2. **Jaeger Endpoint**: Collector sends to Jaeger on port 4317 (OTLP), but Jaeger all-in-one typically uses 14250
3. **No Authentication**: All services run without authentication (educational/demo only)
4. **Single Network**: All services on same Docker network (no network isolation)

## Dependencies

### Java Dependencies (from `pom.xml`)
- Jakarta Servlet API (provided by Tomcat)
- MySQL Connector/J
- Apache HttpClient
- JSON library
- Jakarta JSTL

### Python Dependencies (from `requirements.txt`)
- Flask 3.1.0
- Werkzeug 3.1.0

## Development Workflow

1. **Build Java WAR**: `cd java-app && mvn clean package`
2. **Start Stack**: `docker-compose up --build`
3. **Access Application**: `http://localhost:8085`
4. **View Traces**: `http://localhost:16686` (Jaeger UI)
5. **View Metrics**: `http://localhost:8888/metrics` (Prometheus format)

## Known Technical Notes

- WAR file must be built before Docker Compose (Tomcat image expects it)
- MySQL initialization scripts run automatically on first start
- Collector configuration is mounted as volume (changes require restart)
- All services depend on network being created first
