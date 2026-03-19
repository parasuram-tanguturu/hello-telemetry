# Project Brief

## Overview
**hello-telemetry** is an example application created for the Udemy course "OpenTelemetry Foundations: Your Guide to Observability". This project demonstrates observability concepts using OpenTelemetry in a microservices architecture.

## Core Requirements
1. **Multi-service Architecture**: Demonstrate distributed tracing across multiple services
2. **Observability Stack**: Integrate OpenTelemetry Collector, Jaeger, and Prometheus metrics
3. **Cross-language Support**: Show telemetry collection from both Java and Python services
4. **Database Integration**: Include database operations to demonstrate span creation
5. **Educational Focus**: Serve as a learning example for OpenTelemetry concepts

## Project Goals
- Demonstrate OpenTelemetry instrumentation in a Java servlet application
- Show telemetry collection from a Python Flask microservice
- Configure OpenTelemetry Collector to receive, process, and export telemetry data
- Visualize traces in Jaeger
- Expose metrics via Prometheus endpoints
- Provide a complete, runnable example using Docker Compose

## Scope
- **In Scope**: Basic telemetry collection, distributed tracing, metrics exposure, Docker-based deployment
- **Out of Scope**: Production-grade security, advanced sampling, custom instrumentation, performance optimization

## Success Criteria
- All services can be started with `docker-compose up`
- Traces flow from Java app → Python service → Jaeger
- Metrics are exposed on Prometheus endpoints
- Database queries create observable spans
- HTTP calls between services are traced
