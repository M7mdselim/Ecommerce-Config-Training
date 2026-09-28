# Ecommerce Config Repository (Spring Cloud Config Server)

> **Repository:** `M7mdselim/Ecommerce-Config-Training`  
> **Consumer:** Spring Cloud Config Server (`http://localhost:8888`)  
> **Format:** Standard YAML (.yaml)

---

## 1. Overview

This repository is the **single source of truth** for all externalized microservice configurations across development, containerized Docker, and production environments for the Ecommerce Platform.

Spring Cloud Config Server clones this repository at startup and serves configurations over HTTP/REST to all business microservices and API Gateway.

---

## 2. Configuration Mapping

| File | Port | Consuming Service | Key Configurations Managed |
|---|---|---|---|
| [`application.yaml`](./application.yaml) | — | **All Services** | Shared JWT HMAC secret, Eureka discovery defaults, Zipkin tracing endpoint, global logging levels, Actuator management endpoints |
| [`application-docker.yaml`](./application-docker.yaml) | — | **All Services** (Profile: `docker`) | Internal Docker DNS container names (`discovery-server:8761`, `kafka:9092`, `redis:6379`, `zipkin:9411`) |
| [`application-prod.yaml`](./application-prod.yaml) | — | **All Services** (Profile: `prod`) | Production PostgreSQL datasources, TLS endpoints, replica discovery |
| [`api-gateway.yaml`](./api-gateway.yaml) | 8080 | `api-gateway` | Route definitions, path rewrite filters, Redis token bucket rate limiting |
| [`product-service.yaml`](./product-service.yaml) | 8081 | `product-service` | Redis caching TTL (`300000ms`), PostgreSQL/H2 datasource, JPA dialect |
| [`order-service.yaml`](./order-service.yaml) | 8082 | `order-service` | Kafka topics & serializers, Resilience4j stack (CircuitBreaker, Retry, Bulkhead, TimeLimiter) |
| [`payment-service.yaml`](./payment-service.yaml) | 8083 | `payment-service` | H2/PostgreSQL datasource for `idempotency_keys` table, chaos testing failure rate & delay |
| [`inventory-service.yaml`](./inventory-service.yaml) | 8084 | `inventory-service` | Kafka consumer/producer, shared JWT secret validation |
| [`notification-service.yaml`](./notification-service.yaml) | 8085 | `notification-service` | Kafka consumer group (`notification-service`), retry topic, and Dead Letter Topic (DLT) |

---

## 3. Property Resolution & Inheritance Hierarchy

When a microservice requests its configuration, Spring Cloud Config Server resolves properties in the following order of precedence (later entries override earlier ones):

```
1. application.yaml                   (Shared global baseline across all services)
        ↓
2. application-{profile}.yaml         (Global profile overrides, e.g., 'docker', 'prod')
        ↓
3. {service-name}.yaml                (Service-specific default configuration)
        ↓
4. {service-name}-{profile}.yaml      (Service-specific profile overrides)
```

---

## 4. Runtime Configuration Reloading

Microservices can dynamically reload configuration changes without restarting:

1. **Push Changes:** Commit and push updated YAML files to this repository.
2. **Refresh Service:** Send a `POST` request to the target service's Actuator refresh endpoint:
   ```bash
   curl -X POST http://localhost:8082/actuator/refresh
   ```
3. Beans annotated with `@RefreshScope` will automatically rebind the updated configuration properties.
