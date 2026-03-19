# hello-telemetry

Example application for Udemy course - [OpenTelemetry Foundations: Your Guide to Observability](https://www.udemy.com/course/opentelemetry-foundations)

## Quick Start

### Build and Run

```bash
# Build Java WAR file
cd java-app && mvn clean package

# Start all services
cd .. && docker compose up --build --detach
```

### Access URLs

| Component | URL | Description |
|-----------|-----|-------------|
| **Web Application** | `http://localhost:8085/MyWebApp/` | Main web application home page |
| **Database Results** | `http://localhost:8085/MyWebApp/results` | Servlet showing database data and Python service integration |
| **Jaeger UI** | `http://localhost:16686` | Distributed tracing visualization |
| **Collector Metrics** | `http://localhost:8888/metrics` | OpenTelemetry Collector metrics |
| **Python Service** | `http://localhost:5001/compute_average_age` | Flask API endpoint (POST) |

**Note:** The application is deployed at context path `/MyWebApp` because the WAR file is named `MyWebApp.war`. See [Components and URLs](docs/components-and-urls.md) for details.

## Documentation

- **[Components and URLs](docs/components-and-urls.md)** - Complete reference for all components, URLs, and network communication
- **[Docker Compose Commands](docs/docker-compose-commands.md)** - Detailed explanation of Docker Compose commands and flags
- **[Troubleshooting](docs/troubleshooting.md)** - Common issues and solutions

## Architecture

```
┌──────────┐      ┌──────────┐      ┌──────────┐
│ Browser  │──────│ Tomcat   │──────│ MySQL    │
│          │      │(Java App)│      │(Database)│
└──────────┘      └────┬─────┘      └──────────┘
                       │
                       │
                  ┌────▼─────┐
                  │ Python   │
                  │ Service  │
                  └──────────┘

┌──────────┐      ┌──────────┐
│Services  │──────│Collector │──────│ Jaeger   │
│          │      │(OTLP)    │      │(Tracing) │
└──────────┘      └──────────┘      └──────────┘
```

## Services

- **ht-tomcat** - Java web application (Tomcat 11)
- **ht-python-service** - Python Flask microservice
- **ht-mysql** - MySQL database
- **ht-otel-collector** - OpenTelemetry Collector
- **ht-jaeger** - Jaeger tracing backend