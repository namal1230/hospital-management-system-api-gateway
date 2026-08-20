# Hospital Management System - API Gateway

A Spring Cloud API Gateway service that acts as the central entry point for the Hospital Management System microservices architecture. This gateway handles routing, load balancing, and service discovery for all backend services.

## 🏥 Project Overview

The Hospital Management System API Gateway is built on **Spring Cloud Gateway** and serves as:

- **Central Router**: Routes requests to appropriate microservices
- **Service Discoverer**: Integrates with Eureka for dynamic service discovery
- **Configuration Manager**: Centralizes configuration through Spring Cloud Config Server
- **Monitoring Point**: Provides health and metrics through actuators

## 🛠️ Technology Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| Java | 25 | Programming Language |
| Spring Boot | 4.1.0 | Application Framework |
| Spring Cloud | 2025.1.2 | Microservices Framework |
| Spring Cloud Gateway | Latest | API Gateway |
| Netflix Eureka | Latest | Service Discovery |
| Spring Cloud Config | Latest | Configuration Management |
| Spring Boot Actuator | Latest | Monitoring & Health Checks |
| Maven | Latest | Build Tool |

## ⚙️ Configuration

### Application Properties

The gateway configuration is externalized and managed by Spring Cloud Config Server.

**Bootstrap Configuration** (`application.properties`):
```properties
spring.application.name=APIGateway
spring.config.import=configserver:http://localhost:9000
```

**Key Configuration Parameters:**
- `spring.application.name`: Application identifier for service registry
- `spring.config.import`: Points to the Config Server for dynamic configuration loading

### Required Configuration Server Settings

Your Config Server should provide the following configuration:

```yaml
spring:
  cloud:
    gateway:
      routes:
        # Example route configuration
        - id: patient-service
          uri: lb://patient-service
          predicates:
            - Path=/patients/**
          filters:
            - StripPrefix=1
            
        - id: appointment-service
          uri: lb://appointment-service
          predicates:
            - Path=/appointments/**
          filters:
            - StripPrefix=1

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka
    instance:
      hostname: localhost
```

## 🚀 Getting Started

### Prerequisites

- Java 25 or higher
- Maven 3.8+
- Spring Cloud Config Server running on `http://localhost:9000`
- Netflix Eureka Server running on `http://localhost:8761`

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/namal1230/hospital-management-system-api-gateway.git
   cd hospital-management-system-api-gateway
   ```

2. **Build the project**
   ```bash
   mvn clean install
   ```

3. **Run the application**
   ```bash
   mvn spring-boot:run
   ```

   Or package and run:
   ```bash
   mvn clean package
   java -jar target/APIGateway-0.0.1-SNAPSHOT.jar
   ```

### Expected Output
```
Started APIGateway in X.XXX seconds
APIGateway listening on port 8080
Registered with Eureka: http://localhost:8761
```

## 📋 API Routes

The gateway will dynamically register routes based on configuration. Common patterns include:

| Service | Route Pattern | Example |
|---------|--------------|---------|
| Patient Service | `/patients/**` | `GET /patients/{id}` |
| Appointment Service | `/appointments/**` | `POST /appointments` |
| Medical Records | `/records/**` | `GET /records/patient/{patientId}` |
| User Service | `/users/**` | `POST /users/login` |

## 🔍 Health & Monitoring

### Health Check Endpoint
```bash
curl http://localhost:8080/actuator/health
```

**Response:**
```json
{
  "status": "UP",
  "components": {
    "discoveryClient": {
      "status": "UP"
    },
    "gateway": {
      "status": "UP"
    }
  }
}
```

### Available Actuator Endpoints

- `/actuator/health` - Application health status
- `/actuator/metrics` - Application metrics
- `/actuator/env` - Environment properties
- `/actuator/beans` - Spring beans information
- `/actuator/gateway/routes` - Registered gateway routes
- `/actuator/gateway/routefilters` - Route filters information

### Enable All Actuator Endpoints (Config Server)

```yaml
management:
  endpoints:
    web:
      exposure:
        include: '*'
  endpoint:
    health:
      show-details: always
```

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Client Requests                    │
└────────────────────┬────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────┐
│       Spring Cloud API Gateway (Port 8080)          │
│  ┌─────────────────────────────────────────────────┐│
│  │         Request Routing & Filtering             ││
│  │  • Path-based routing                           ││
│  │  • Load balancing                               ││
│  │  • Request/Response filtering                   ││
│  └─────────────────────────────────────────────────┘│
└───────┬──────────────────────────────────────┬──────┘
        │                                      │
┌───────▼──────────────┐        ┌──────────────▼─────┐
│   Eureka Discovery   │        │ Config Server      │
│   (Port 8761)        │        │ (Port 9000)        │
└──────────────────────┘        └────────────────────┘
        │ (Service Registry)     │ (Configuration)
        │                        │
   ┌────┴────┬─────────┬───────┬┴──────┐
   │          │         │       │       │
┌──▼──┐  ┌───▼──┐  ┌───▼──┐  ┌▼───┐  ┌▼───┐
│Patient Service
│Service  │Appt  │  │Record │ │User │  │...│
│Service  │Service│  │Service│ │Srvc│  │   │
└────────┘  └──────┘  └──────┘  └────┘  └───┘
```

## 🔐 Security Considerations

Currently, the gateway has **no authentication/authorization** configured. For production, add:

### Recommended Security Setup

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://auth-server:8080
          
  cloud:
    gateway:
      filter:
        request-rate-limiter:
          enabled: true
          key-resolver: userKeyResolver
```

### Add Spring Security Dependency

Update `pom.xml`:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-resource-server</artifactId>
</dependency>
```

## 🧪 Testing

### Unit Tests
```bash
mvn test
```

### Integration Tests
```bash
mvn verify
```

### Testing with curl

```bash
# Test gateway health
curl http://localhost:8080/actuator/health

# Test route to patient service
curl http://localhost:8080/patients/1

# View all routes
curl http://localhost:8080/actuator/gateway/routes
```

## 📦 Project Structure

```
hospital-management-system-api-gateway/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/apigateway/
│   │   │       ├── Application.java
│   │   │       ├── config/
│   │   │       ├── filters/
│   │   │       └── security/
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│       └── java/
│           └── com/example/apigateway/
└── pom.xml
```

## 🔧 Common Tasks

### Add a New Route

1. Update your Config Server configuration:
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: new-service
          uri: lb://new-service
          predicates:
            - Path=/newservice/**
```

2. Restart the gateway or trigger configuration refresh:
```bash
curl -X POST http://localhost:8080/actuator/refresh
```

### Change Port

Add to Config Server:
```yaml
server:
  port: 8888
```

### Enable CORS

```yaml
spring:
  cloud:
    gateway:
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOrigins: "http://localhost:3000"
            allowedMethods:
              - GET
              - POST
              - PUT
              - DELETE
            allowedHeaders: "*"
```

## 📊 Metrics & Logging

### Enable Debug Logging

Config Server:
```yaml
logging:
  level:
    org.springframework.cloud.gateway: DEBUG
    org.springframework.web: DEBUG
```

### View Metrics

```bash
curl http://localhost:8080/actuator/metrics
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 Dependencies

### Core Dependencies
- **spring-boot-starter-actuator**: Health checks and metrics
- **spring-cloud-starter-gateway-server-webmvc**: API Gateway functionality
- **spring-cloud-starter-netflix-eureka-client**: Service discovery
- **spring-cloud-starter-config**: Externalized configuration
- **spring-cloud-commons**: Cloud commons utilities

### Development & Testing
- **spring-boot-starter-actuator-test**: Actuator testing

## 🚨 Troubleshooting

### Gateway fails to start
- **Issue**: Config Server unreachable
- **Solution**: Ensure Config Server is running on `http://localhost:9000`

### Services not discoverable
- **Issue**: Eureka Server down
- **Solution**: Start Eureka Server on `http://localhost:8761`

### Routes not responding
- **Issue**: Downstream services not registered
- **Solution**: Check service registration with `/actuator/gateway/routes`

### Port already in use
- **Issue**: Port 8080 occupied
- **Solution**: Configure different port via Config Server or use `-Dserver.port=XXXX`

## 📚 Additional Resources

- [Spring Cloud Gateway Documentation](https://spring.io/projects/spring-cloud-gateway)
- [Netflix Eureka Documentation](https://github.com/Netflix/eureka/wiki)
- [Spring Cloud Config Documentation](https://spring.io/projects/spring-cloud-config)
- [Spring Boot Actuator Guide](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 👨‍💼 Author

**Namal1230**
- GitHub: [@namal1230](https://github.com/namal1230)
- Repository: [hospital-management-system-api-gateway](https://github.com/namal1230/hospital-management-system-api-gateway)

## 🙏 Support

For questions or issues:
1. Check the [Issues](https://github.com/namal1230/hospital-management-system-api-gateway/issues) page
2. Review the Troubleshooting section
3. Consult Spring Cloud Gateway documentation

---

**Last Updated**: August 2026  
**Version**: 0.0.1-SNAPSHOT
