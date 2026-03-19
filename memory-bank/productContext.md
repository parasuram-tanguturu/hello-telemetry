# Product Context

## Why This Project Exists
This project serves as a **hands-on learning tool** for understanding OpenTelemetry observability concepts. It provides a complete, working example that students can run, modify, and experiment with to learn:

- How to instrument applications with OpenTelemetry
- How telemetry data flows through a collector
- How distributed traces connect services
- How to visualize observability data

## Problems It Solves
1. **Learning Gap**: Provides a concrete example when learning OpenTelemetry concepts
2. **Integration Complexity**: Shows how to connect multiple services with observability
3. **Cross-language Examples**: Demonstrates instrumentation in both Java and Python
4. **Complete Stack**: Includes collector, backend (Jaeger), and visualization

## How It Should Work

### User Flow
1. User accesses the Java web application at `http://localhost:8085`
2. User clicks "View Database Results" link
3. Application:
   - Queries MySQL database for user data
   - Calls Python microservice to compute average age
   - Displays results in HTML table with average age

### Observability Flow
1. **Instrumentation**: Both Java and Python services generate telemetry (traces, metrics, logs)
2. **Collection**: OpenTelemetry Collector receives telemetry via OTLP (gRPC/HTTP)
3. **Processing**: Collector batches and processes telemetry data
4. **Export**: Collector exports traces to Jaeger, metrics to Prometheus endpoints
5. **Visualization**: Users view traces in Jaeger UI at `http://localhost:16686`

## User Experience Goals
- **Simplicity**: Easy to start with `docker-compose up`
- **Clarity**: Clear example of telemetry flow
- **Educational**: Code is readable and well-commented for learning
- **Completeness**: Full stack from instrumentation to visualization

## Key Use Cases
1. **Learning OpenTelemetry**: Students follow along with course material
2. **Experimentation**: Modify instrumentation to see effects
3. **Reference**: Use as a template for other projects
4. **Demonstration**: Show OpenTelemetry capabilities to others
