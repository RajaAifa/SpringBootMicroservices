# 🧩 SpringBootMicroservices

**A microservices-based backend system built with Spring Boot and Spring Cloud, featuring service discovery, centralized configuration, an API gateway, Keycloak-based authentication, RabbitMQ messaging, and MySQL persistence.**

![Java](https://img.shields.io/badge/Java-8+-orange?style=flat-square)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-microservices-6DB33F?style=flat-square)
![Spring Cloud](https://img.shields.io/badge/Spring%20Cloud-Eureka%20%7C%20Config%20%7C%20Gateway-6DB33F?style=flat-square)
![Keycloak](https://img.shields.io/badge/Keycloak-security-4D4D4D?style=flat-square)
![RabbitMQ](https://img.shields.io/badge/RabbitMQ-messaging-FF6600?style=flat-square)
![MySQL](https://img.shields.io/badge/MySQL-database-4479A1?style=flat-square)

---

## 📖 Overview

This project implements a microservices architecture composed of **seven independent Spring Boot applications**:

| Service | Role |
|---|---|
| **Discovery Server** | Eureka server for service registration and discovery |
| **Config Server** | Spring Cloud Config server serving centralized configuration to all services from a native (classpath) source |
| **API Gateway** | Spring Cloud Gateway routing incoming requests to the right downstream service based on path predicates |
| **User Service** | Manages user data, backed by MySQL and secured with Keycloak |
| **Advertisement Service** | Manages advertisements, backed by MySQL, secured with Keycloak, and publishes / consumes events via RabbitMQ |
| **Management Service** | Handles management operations, secured with Keycloak, with circuit breaker support via Resilience4j and health monitoring via Spring Boot Actuator |
| **Report Service** | Generates reports, backed by MySQL, secured with Keycloak, and integrated with RabbitMQ messaging |

All business services register with Eureka, pull their configuration from the Config Server, and are protected by Keycloak.

---

## 🏗️ Architecture

```text
                          ┌──────────────────────┐
        Client ─────────► │   API Gateway :8600  │
                          └──────────┬───────────┘
                                     │ routes by path
        ┌──────────────┬─────────────┼───────────────┬────────────────┐
        ▼              ▼             ▼               ▼                │
  User Service   Advertisement   Management      Report Service       │
     :9000        Service :9001  Service :9002       :9003            │
        │              │  ▲          │                │ ▲            │
        │              ▼  │          │                ▼ │            │
        │            RabbitMQ ◄──────┼────────────────┘              │
        ▼              ▼                                              │
      MySQL          MySQL                    Keycloak (auth) ◄───────┘

  Every service ──► registers with Discovery Server (Eureka) :8761
  Every service ──► loads its configuration from Config Server :9191
```

---

## 🛠️ Technologies Used

| Layer | Technology |
|---|---|
| Language | Java |
| Framework | Spring Boot, Spring Cloud (Config, Gateway, Netflix Eureka) |
| Security | Keycloak (`spring-boot-starter-security`, `keycloak-spring-boot-starter`, `keycloak-admin-client`) |
| Persistence | Spring Data JPA, MySQL (`mysql-connector-java`) |
| Messaging | RabbitMQ (`spring-boot-starter-amqp`) |
| Resilience and monitoring | Resilience4j (circuit breaker), Spring Boot Actuator |
| Build tool | Maven |
| Other | Lombok |

---

## 📋 Prerequisites

- Java 8+ (JDK)
- Maven 3.6+
- MySQL Server
- RabbitMQ Server
- Keycloak Server

---

## 🔌 Services and Ports

| Service | Port | Description |
|---|---|---|
| Discovery Server (Eureka) | 8761 | Service registry |
| Config Server | 9191 | Centralized configuration |
| API Gateway | 8600 | Entry point / routing |
| User Service | 9000 | User management |
| Advertisement Service | 9001 | Advertisement management |
| Management Service | 9002 | Management operations |
| Report Service | 9003 | Report generation |

---

## 🚀 Run Locally

### 1. Clone the project

```bash
git clone https://github.com/RajaAifa/SpringBootMicroservices.git
cd SpringBootMicroservices
```

### 2. Build all modules

```bash
mvn clean install
```

### 3. Start the infrastructure

Start **MySQL**, **RabbitMQ** and **Keycloak** before starting the services.

A `docker-compose.yml` is provided for **Keycloak** and **RabbitMQ**:

```bash
docker compose up -d
```

| Service | URL | Default credentials (local development only) |
|---|---|---|
| Keycloak | http://localhost:8181 | `admin` / `admin` |
| RabbitMQ management | http://localhost:15672 | `rabbitmq` / `123456` |

You can override them with the environment variables `KEYCLOAK_ADMIN`, `KEYCLOAK_ADMIN_PASSWORD`, `RABBITMQ_USER` and `RABBITMQ_PASS`. MySQL is not included in the compose file, so install and start it separately.

### 4. Start the services in this order

```bash
# 1. Discovery Server
cd discoveryserver && mvn spring-boot:run

# 2. Config Server
cd configserver && mvn spring-boot:run

# 3. API Gateway
cd api-gateway && mvn spring-boot:run

# 4. Business services (any order)
cd user-service && mvn spring-boot:run
cd advertisement-service && mvn spring-boot:run
cd management-service && mvn spring-boot:run
cd report-service && mvn spring-boot:run
```

Run each command in its own terminal, from the project root.

- Eureka dashboard: **http://localhost:8761**
- API Gateway: **http://localhost:8600**

---

## ⚙️ Configuration

Each service imports its configuration from the Config Server (`http://localhost:9191`) on startup. The centralized configuration files (datasource, Keycloak settings, RabbitMQ settings, service ports) live under:

```text
configserver/src/main/resources/configurations/
```

Before running the project, replace the placeholder values with your own environment values:

| Setting | What to provide |
|---|---|
| MySQL | connection URL, username and password |
| Keycloak | realm, auth server URL and client credentials |
| RabbitMQ | host, username and password |

> 🔒 The repository contains placeholder values only. Never commit real passwords, client secrets or tokens.

---

## 🔀 API Routing (API Gateway)

| Route | Path | Target service |
|---|---|---|
| Users | `/api/v1/users/*` | `user-service` |
| Reports | `/api/v1/reports/*` | `report-service` |
| Advertisement / Management | `/api/**` | `advertisement-service`, `management-service` |

---

## 📁 Project Structure

```text
SpringBootMicroservices/
├── discoveryserver/          # Eureka service discovery server
├── configserver/             # Spring Cloud Config server
├── api-gateway/              # Spring Cloud Gateway (routing)
├── user-service/             # User management microservice
├── advertisement-service/    # Advertisement management microservice
├── management-service/       # Management microservice
├── report-service/           # Report generation microservice
├── docker-compose.yml        # Keycloak and RabbitMQ for local development
└── pom.xml                   # Parent Maven module
```

---


| Eureka dashboard | API Gateway request |
|---|---|
| ![Eureka dashboard](docs/screenshots/eureka.png) | ![API Gateway request](docs/screenshots/gateway.png) |

-->



## 👩‍💻 Author

**Raja Aifa**

🎓 Professional Master's Degree in Web Services and Multimedia,Tunisia

---

## 📜 License

This project is intended for educational purposes.
