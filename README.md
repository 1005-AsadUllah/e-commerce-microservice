# E-Commerce Microservice Architecture

A comprehensive microservices-based e-commerce platform built with Spring Boot, Spring Cloud, and Kubernetes. This project demonstrates modern microservices patterns including API Gateway, Service Discovery, Circuit Breaker, Distributed Tracing, and Rate Limiting.

## 🏗️ Architecture Overview

This project follows a microservices architecture pattern with the following components:

```
┌─────────────────┐
│   Client App    │
│  (Web/Mobile)   │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────┐
│              API Gateway (Port: 9090)                   │
│  • Routing • Circuit Breaker • Rate Limiting • Tracing  │
└────────┬────────────────────────────────────────────────┘
         │
         ├─────────────────┬─────────────────┬──────────────┐
         ▼                 ▼                 ▼              ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Product    │  │    Order     │  │   Payment    │  │    Config    │
│   Service    │  │   Service    │  │   Service    │  │    Server    │
│  (Port:8082) │  │  (Port:8083) │  │  (Port:8081) │  │  (Port:9296) │
└──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘
       │                 │                 │
       └─────────────────┴─────────────────┘
                        │
         ┌──────────────┴──────────────┐
         ▼                             ▼
┌──────────────┐              ┌──────────────┐
│    MySQL     │              │    Redis     │
│  (Port:3306) │              │  (Port:6379) │
└──────────────┘              └──────────────┘
```

## 🚀 Services

### 1. **API Gateway** (`CloudGateWay`)
- **Port**: 9090
- **Technology**: Spring Cloud Gateway (WebFlux)
- **Features**:
  - Request routing to microservices
  - Circuit Breaker with Resilience4j
  - Rate limiting using Redis
  - Distributed tracing with Zipkin
  - Fallback mechanisms for service failures

### 2. **Config Server** (`ConfigServer`)
- **Port**: 9296
- **Technology**: Spring Cloud Config Server
- **Features**:
  - Centralized configuration management
  - Git-based configuration repository
  - Dynamic configuration updates

### 3. **Product Service** (`ProductService`)
- **Port**: 8082
- **Database**: MySQL (`productdb`)
- **Features**:
  - Product CRUD operations
  - Product inventory management
  - Quantity reduction for orders
  - RESTful API endpoints

### 4. **Order Service** (`OrderService`)
- **Port**: 8083
- **Database**: MySQL (`orderdb`)
- **Features**:
  - Order placement and management
  - Integration with Product and Payment services via Feign Client
  - Circuit Breaker for external service calls
  - WebFlux for reactive programming

### 5. **Payment Service** (`PaymentService`)
- **Port**: 8081
- **Database**: MySQL (`paymentdb`)
- **Features**:
  - Payment processing
  - Payment history tracking
  - Payment retrieval by order ID

## 🛠️ Technology Stack

### Core Technologies
- **Java**: 21
- **Spring Boot**: 3.5.6
- **Spring Cloud**: 2025.0.0
- **Build Tool**: Maven

### Spring Cloud Components
- **Spring Cloud Gateway**: API Gateway with WebFlux
- **Spring Cloud Config**: Centralized configuration
- **Spring Cloud OpenFeign**: Declarative HTTP client
- **Spring Cloud LoadBalancer**: Client-side load balancing
- **Resilience4j**: Circuit Breaker implementation

### Infrastructure
- **MySQL**: Relational database (8.0)
- **Redis**: Rate limiting and caching
- **Zipkin**: Distributed tracing
- **Docker**: Containerization
- **Kubernetes**: Orchestration

### Additional Libraries
- **Lombok**: Boilerplate code reduction
- **Spring Data JPA**: Database access
- **Spring WebFlux**: Reactive programming
- **Micrometer**: Metrics and observability
- **OpenTelemetry**: Distributed tracing

## 📋 Prerequisites

- **Java**: JDK 21 or higher
- **Maven**: 3.6+ (or use Maven wrapper included in each service)
- **Docker**: For containerization
- **Kubernetes**: For orchestration (optional)
- **MySQL**: 8.0+ (or use Docker)
- **Redis**: 7.0+ (or use Docker)
- **Git**: For cloning the repository

## 🔧 Setup Instructions

### 1. Clone the Repository

⚠️ **Important**: This project uses Git submodules. Use the correct clone command:

```bash
# ✅ Correct way
git clone --recurse-submodules https://github.com/your-username/e-commerce-microservice.git

# OR if already cloned:
git submodule update --init --recursive
```

### 2. Database Setup

#### Using Docker Compose (Recommended)
```bash
# Start MySQL and Redis
docker-compose up -d
```

#### Manual Setup
1. Create MySQL databases:
   ```sql
   CREATE DATABASE productdb;
   CREATE DATABASE orderdb;
   CREATE DATABASE paymentdb;
   ```

2. Start Redis server:
   ```bash
   redis-server
   ```

### 3. Configuration Server Setup

The Config Server requires a Git repository with configuration files. Update the repository URL in `ConfigServer/src/main/resources/application.yml`:

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/your-username/spring-config.git
          clone-on-start: true
```

### 4. Build the Project

Build all services using Maven wrapper:

**On macOS/Linux:**
```bash
./mvnw clean package -DskipTests
```

**On Windows (PowerShell/CMD):**
```powershell
mvnw.cmd clean package -DskipTests
```

**Or build individual services:**

```bash
# Build Config Server
cd ConfigServer
./mvnw clean install  # or mvnw.cmd on Windows

# Build API Gateway
cd ../CloudGateWay
./mvnw clean install

# Build Product Service
cd ../ProductService
./mvnw clean install

# Build Order Service
cd ../OrderService
./mvnw clean install

# Build Payment Service
cd ../PaymentService
./mvnw clean install
```

### 5. Run Services

**Start services in the following order:**

1. **Config Server** (Port 9296)
   ```bash
   cd ConfigServer
   ./mvnw spring-boot:run  # or mvnw.cmd on Windows
   ```

2. **Product Service** (Port 8082)
   ```bash
   cd ProductService
   ./mvnw spring-boot:run
   ```

3. **Payment Service** (Port 8081)
   ```bash
   cd PaymentService
   ./mvnw spring-boot:run
   ```

4. **Order Service** (Port 8083)
   ```bash
   cd OrderService
   ./mvnw spring-boot:run
   ```

5. **API Gateway** (Port 9090)
   ```bash
   cd CloudGateWay
   ./mvnw spring-boot:run
   ```

### 6. Environment Variables

Set the following environment variables if needed:

```bash
# Database host (default: localhost)
export DB_HOST=localhost

# Config Server URL (default: http://config-server:9296)
export CONFIG_SERVER=http://localhost:9296
```

## 📡 API Endpoints

### API Gateway Base URL: `http://localhost:9090`

### Product Service Endpoints
- `GET /product` - Get all products
- `GET /product/{id}` - Get product by ID
- `POST /product` - Create a new product
- `PUT /product/{id}` - Update product
- `PUT /product/reduceQuantity/{id}?quantity={qty}` - Reduce product quantity
- `DELETE /product/{id}` - Delete product

### Order Service Endpoints
- `GET /order` - Get all orders
- `POST /order/placeOrder` - Place a new order
- `GET /order/check` - Health check

### Payment Service Endpoints
- `POST /payment/doPayment` - Process payment
- `GET /payment/{orderId}` - Get payment by order ID
- `GET /payment/by/{id}` - Get payment by payment ID

## 🐳 Docker Deployment

### Build Docker Images

Each service includes a Dockerfile. Build images using:

```bash
# Build Config Server
cd ConfigServer
docker build -t config-service:latest .

# Build API Gateway
cd CloudGateWay
docker build -t cloud-service:latest .

# Build Product Service
cd ProductService
docker build -t product-service:latest .

# Build Order Service
cd OrderService
docker build -t order-service:latest .

# Build Payment Service
cd PaymentService
docker build -t payment-service:latest .
```

### Using Jib Maven Plugin (Recommended)

The project uses Jib for building container images:

```bash
# Build and push to Docker Hub
mvn compile jib:build

# Build to local Docker daemon
mvn compile jib:dockerBuild
```

## ☸️ Kubernetes Deployment

Kubernetes deployment manifests are available in the `K8s/` directory.

### Deploy Infrastructure Components

```bash
# Deploy MySQL
kubectl apply -f K8s/mysql-deployment.yaml

# Deploy Redis
kubectl apply -f K8s/redis-deployment.yaml

# Deploy Zipkin
kubectl apply -f K8s/zipkin-deployment.yaml

# Deploy Config Maps
kubectl apply -f K8s/config-maps.yaml
```

### Deploy Microservices

```bash
# Deploy Config Server
kubectl apply -f K8s/config-server-deployment.yaml

# Deploy Product Service
kubectl apply -f K8s/product-service-deployment.yaml

# Deploy Payment Service
kubectl apply -f K8s/payment-service-deployment.yaml

# Deploy Order Service
kubectl apply -f K8s/order-service-deployment.yaml

# Deploy API Gateway
kubectl apply -f K8s/cloud-gateway-deployment.yaml
```

**Or deploy all at once:**
```bash
kubectl apply -f K8s/
```

**Note**: Check and edit `K8s/config-maps.yaml` and secrets for environment-specific configuration before applying.

## ✨ Key Features

### 1. **Circuit Breaker Pattern**
- Implemented using Resilience4j
- Prevents cascading failures
- Automatic fallback responses
- Configurable failure thresholds

### 2. **Rate Limiting**
- Redis-based rate limiting
- Configurable per-service limits
- Prevents API abuse

### 3. **Distributed Tracing**
- Zipkin integration
- OpenTelemetry for trace collection
- End-to-end request tracking

### 4. **Centralized Configuration**
- Spring Cloud Config Server
- Git-based configuration management
- Dynamic configuration updates

### 5. **Service Communication**
- OpenFeign for synchronous calls
- WebFlux for reactive programming
- Load balancing support

### 6. **Observability**
- Spring Boot Actuator endpoints
- Health checks
- Metrics collection

## 📁 Project Structure

```
e-commerce-microservice/
├── CloudGateWay/              # API Gateway Service
│   ├── src/
│   │   └── main/
│   │       ├── java/
│   │       └── resources/
│   │           └── application.properties
│   ├── Dockerfile
│   └── pom.xml
│
├── ConfigServer/              # Configuration Server
│   ├── src/
│   │   └── main/
│   │       ├── java/
│   │       └── resources/
│   │           └── application.yml
│   ├── Dockerfile
│   └── pom.xml
│
├── OrderService/              # Order Management Service
│   ├── src/
│   │   └── main/
│   │       ├── java/
│   │       │   └── com/Tulip_Tech/OrderService/
│   │       │       ├── client/          # Feign Clients
│   │       │       ├── controller/      # REST Controllers
│   │       │       ├── entity/          # JPA Entities
│   │       │       ├── model/           # Domain Models & DTOs
│   │       │       ├── repository/      # Data Repositories
│   │       │       └── service/         # Business Logic
│   │       └── resources/
│   │           └── application.yml
│   └── pom.xml
│
├── PaymentService/            # Payment Processing Service
│   ├── src/
│   │   └── main/
│   │       ├── java/
│   │       │   └── com/Tulip_Tech/PaymentService/
│   │       │       ├── controller/
│   │       │       ├── entity/
│   │       │       ├── model/
│   │       │       ├── repository/
│   │       │       └── service/
│   │       └── resources/
│   │           └── application.properties
│   └── pom.xml
│
├── ProductService/            # Product Management Service
│   ├── src/
│   │   └── main/
│   │       ├── java/
│   │       │   └── com/Tulip_Tech/ProductService/
│   │       │       ├── controller/
│   │       │       ├── entity/
│   │       │       ├── exception/
│   │       │       ├── model/
│   │       │       ├── repository/
│   │       │       └── service/
│   │       └── resources/
│   │           └── application.yml
│   ├── Dockerfile
│   └── pom.xml
│
└── K8s/                       # Kubernetes Deployment Manifests
    ├── cloud-gateway-deployment.yaml
    ├── config-server-deployment.yaml
    ├── config-maps.yaml
    ├── mysql-deployment.yaml
    ├── order-service-deployment.yaml
    ├── payment-service-deployment.yaml
    ├── product-service-deployment.yaml
    ├── redis-deployment.yaml
    └── zipkin-deployment.yaml
```

## 🔍 Monitoring & Observability

### Actuator Endpoints

Each service exposes Spring Boot Actuator endpoints:

- Health: `http://localhost:{port}/actuator/health`
- Metrics: `http://localhost:{port}/actuator/metrics`
- Info: `http://localhost:{port}/actuator/info`

### Distributed Tracing

Access Zipkin UI at: `http://localhost:9411`

View traces across all microservices and track request flows.

## 🔐 Configuration

### Database Configuration
- Default MySQL credentials: `USERNAME AND PASSWORD`
- Databases are auto-created by JPA (`ddl-auto: update`)

### Gateway Routes
Configured in `CloudGateWay/src/main/resources/application.properties`:
- Payment Service: `/payment/**`
- Order Service: `/order/**`
- Product Service: `/product/**`

### Circuit Breaker Configuration
- Failure rate threshold: 50%
- Minimum number of calls: 5
- Wait duration in open state: 5s
- Sliding window type: COUNT_BASED

## 🚦 Rate Limiting

Rate limiting is configured per route:
- **Payment Service**: 1 request/second, burst capacity: 2
- **Order Service**: 1 request/second, burst capacity: 1
- **Product Service**: 1 request/second, burst capacity: 1

## 📝 Notes

- All services use Java 21 and Spring Boot 3.5.6
- Services are configured to use Spring Cloud Config Server
- Docker images are configured to push to Docker Hub (update registry URLs in `pom.xml`)
- Kubernetes manifests use LoadBalancer service type for external access
- Use the included Maven wrappers (`mvnw`, `mvnw.cmd`) to avoid needing a globally installed Maven
- Ports, database connections, and external dependencies are configured per-service (see `src/main/resources` of each module)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

Feel free to open issues or pull requests. For quick communication, leave a comment in the repository.

## 📄 License

This project is for educational purposes.

## 👨‍💻 Author

- GitHub: [Asadullah](https://github.com/1005-AsadUllah)

## 🙏 Acknowledgments

- Spring Cloud documentation
- Microservices patterns and best practices
- Kubernetes documentation

---

**Happy Coding! 🚀**
