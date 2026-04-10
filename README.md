# Native Cloud vs Cloud.
## Native Cloud Approach by implementing all the best practices. 
## An implementation of a bank application
### used:
#### Java Springboot
#### DTO design pattern
#### Microservices
### Config server as an microservice for all configurations and different profiles (test, Production, staging)
#### API first focus
### Spring Cloud


#### Docker & kubernetes
#### Clear implementation with right sizing of pods and security thinking



# 1. Architectural Foundations
## API-First & DTO Design
Contract-First Development: Use OpenAPI (Swagger) to define your banking API contracts before writing code. This allows frontend and backend teams to work in parallel and ensures the Kubernetes ingress/mesh knows exactly what traffic to expect.

## Data Decoupling: 
Use DTOs (Data Transfer Objects) to ensure your internal database schema is never exposed directly. This is critical in banking for security and to allow microservices to evolve their internal logic without breaking the API contract.

## Microservices Granularity
Domain-Driven Design (DDD): Divide the bank into bounded contexts (e.g., account-service, payment-service, auth-service).

Statelessness: Ensure Spring Boot services are stateless. Any session data or state should be stored in external distributed caches (like Redis) or databases to allow Kubernetes to kill and restart pods without data loss.

# 2. Containerization & Orchestration
Docker Optimization
Multi-stage Builds: Use multi-stage Dockerfiles to keep images small. Build the JAR in the first stage and copy only the runtime to a "Distroless" or Alpine-based image.

## Non-Root Users:
Never run your Spring Boot application as root inside the container. Define a specific USER in the Dockerfile to mitigate container breakout risks.

## Kubernetes Deployment Strategy
Right-Sizing Pods: * Requests vs. Limits: Set requests based on the baseline Java heap requirements and limits to prevent a single service from consuming the entire node's resources (OOMKilled).

Vertical Pod Autoscaler (VPA): Use VPA in "Recommendation mode" during staging to find the perfect CPU/Memory balance for your Spring Boot apps.

Liveness and Readiness Probes: Use the spring-boot-starter-actuator health endpoints.

Readiness: Tells K8s when the bank app is ready to accept traffic (e.g., DB connection is live).

Liveness: Tells K8s if the app has deadlocked and needs a restart.

# 3. Cloud-Native Security Thinking
In a banking context, security is integrated into the infrastructure, not added as an afterthought.

Network Policies: By default, all pods in Kubernetes can talk to each other. Implement Zero Trust by using Network Policies to restrict traffic (e.g., the web-frontend pod should never be able to talk directly to the database pod; it must go through the account-service).

Secrets Management: Do not store DB passwords in Kubernetes Secrets (which are only Base64 encoded). Use a sidecar pattern or CSI driver to inject secrets from a hardened vault (like HashiCorp Vault).

mTLS (Service Mesh): Consider a service mesh like Istio or Linkerd to ensure all communication between your Java microservices is encrypted via mutual TLS and to provide fine-grained observability.
