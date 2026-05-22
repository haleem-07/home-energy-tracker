# Home Energy Tracker

[![Java](https://img.shields.io/badge/Java-21-orange.svg)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.0-green.svg)](https://spring.io/projects/spring-boot)
[![Spring Cloud](https://img.shields.io/badge/Spring%20Cloud-2025.1.0-blue.svg)](https://spring.io/projects/spring-cloud)
[![Kafka](https://img.shields.io/badge/Apache%20Kafka-4.2-black.svg)](https://kafka.apache.org/)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED.svg?logo=docker&logoColor=white)](https://docs.docker.com/compose/)
[![React](https://img.shields.io/badge/React-18-61DAFB.svg?logo=react&logoColor=black)](https://react.dev/)
[![License](https://img.shields.io/badge/license-MIT-lightgrey.svg)]()

A **production-grade microservices system** for monitoring and managing household electricity usage in real time. The system collects energy readings from smart devices, processes them asynchronously through an event-driven pipeline, stores time-series metrics, triggers alerts when usage exceeds thresholds, and exposes a unified API through a secured **API Gateway** — with a **React dashboard frontend** for visualization.

---

## 👨‍💻 Author

**Mohammad Haleem**
- GitHub: [@haleem-07](https://github.com/haleem-07)

---

## 📌 Project Overview

**Home Energy Tracker** simulates how a real smart home product collects **power consumption data** from devices like plugs, lights, and thermostats — aggregates it for dashboards — and notifies residents when energy consumption crosses configured thresholds.

**Problem it solves:**
Raw device events are high-volume and need reliable ingestion, decoupled processing, and specialized storage. This project demonstrates that split: HTTP APIs for users and devices, Kafka for event streaming, InfluxDB for usage time-series, and MySQL for durable domain data.

**Key capabilities:**
- Track **per-device** energy usage over time
- **Alert** users when instantaneous or aggregated power exceeds a limit
- **Gate** all public HTTP traffic through one entry point (API Gateway) with JWT validation
- **Observe** latency, errors, and circuit-breaker state with Prometheus and Grafana
- **Visualize** all data through a dark-themed React dashboard

---

## 🏗️ Architecture Overview

The system follows a **microservices architecture** built with **Spring Boot 4** and **Java 21**. Services are independently deployable; integration uses **synchronous HTTP** (client → gateway → service) and **asynchronous messaging** (Kafka) for loose coupling and scale.

```
React Frontend (localhost:5173)
        ↓
API Gateway (9000) — JWT auth, circuit breaker, routing
        ↓
┌───────────────────────────────────────────┐
│  user-service  │  device-service  │  insight-service  │
└───────────────────────────────────────────┘
        ↓
Ingestion Service (8082) → Kafka → Usage Service (8083)
                                        ↓
                              ┌─────────────────┐
                              │   InfluxDB       │  time-series storage
                              │   Alert Service  │  email notifications
                              └─────────────────┘
```

| Area | Approach |
|------|----------|
| **API Gateway** | Spring Cloud Gateway (Server MVC); single public HTTP façade |
| **Service communication** | REST between gateway and backends; Kafka for async pipeline |
| **Resilience** | Circuit breakers (Resilience4j) on gateway routes with fallbacks |
| **Security** | OAuth2 Resource Server on the gateway; Keycloak for identity |
| **Observability** | Spring Boot Actuator, Micrometer, Prometheus, Grafana dashboards |

---

## 🛠️ Services Breakdown

| Service | Port | Responsibility | Key Technologies |
|---------|------|----------------|-----------------|
| **api-gateway** | `9000` | Public entry: routing, circuit breaking, JWT validation | Spring Cloud Gateway, Resilience4j, OAuth2 |
| **user-service** | `8080` | User accounts and persistence | Spring Boot 4, JPA, MySQL, Flyway |
| **device-service** | `8081` | Device registry and metadata | Spring Boot 4, JPA, MySQL |
| **ingestion-service** | `8082` | Accept energy readings, publish to Kafka | Spring Boot 4, Kafka Producer |
| **usage-service** | `8083` | Consume events, store in InfluxDB, check thresholds | Spring Boot 4, Kafka Consumer, InfluxDB |
| **alert-service** | `8084` | Consume alert events, send email notifications | Spring Boot 4, Kafka, Mail, MySQL |
| **insight-service** | `8085` | AI-powered usage insights via Ollama | Spring Boot 3.5, Spring AI |

---

## 💻 Tech Stack

| Category | Technologies |
|----------|-------------|
| **Language** | Java 21 |
| **Framework** | Spring Boot 4, Spring Cloud 2025.1.0 |
| **Messaging** | Apache Kafka (KRaft mode) |
| **Databases** | MySQL 8 (relational), InfluxDB 2 (time-series) |
| **Security** | Keycloak, OAuth2, JWT |
| **Resilience** | Resilience4j Circuit Breaker |
| **Observability** | Micrometer, Prometheus, Grafana |
| **Email** | Mailpit (local SMTP) |
| **Infrastructure** | Docker, Docker Compose |
| **Build** | Maven |

---

## 🚀 Getting Started

### Prerequisites

- **JDK 21** — [Download Temurin 21](https://adoptium.net/temurin/releases/?version=21)
- **Docker Desktop** — [Download here](https://www.docker.com/products/docker-desktop/)
- **Maven 3.9+** — [Download here](https://maven.apache.org/download.cgi)

### Clone the Repository

```bash
git clone https://github.com/haleem-07/home-energy-tracker.git
cd home-energy-tracker
```

### Start Infrastructure (Docker)

```bash
docker compose up -d
```

This starts: MySQL, Kafka, Kafka UI, InfluxDB, Mailpit, Keycloak, Prometheus, and Grafana.

### Build All Services

```bash
# Windows PowerShell
$services = @("user-service","device-service","ingestion-service","usage-service","alert-service","insight-service","api-gateway")
foreach ($svc in $services) {
    cd $svc; mvn -q package -DskipTests; cd ..
}
```

### Run All Services

Start each service in a separate terminal:

```bash
cd user-service       && mvn spring-boot:run   # :8080
cd device-service     && mvn spring-boot:run   # :8081
cd ingestion-service  && mvn spring-boot:run   # :8082
cd usage-service      && mvn spring-boot:run   # :8083
cd alert-service      && mvn spring-boot:run   # :8084
cd insight-service    && mvn spring-boot:run   # :8085
cd api-gateway        && mvn spring-boot:run   # :9000 (start last)
```

### Start Frontend

```bash
cd frontend
npm install
npm run dev
```

Open **http://localhost:5173** in your browser.

---

## 🔗 Access Points

| Service | URL | Credentials |
|---------|-----|-------------|
| **React Frontend** | http://localhost:5173 | Register with your details |
| **API Gateway** | http://localhost:9000 | — |
| **Swagger UI** | http://localhost:9000/swagger-ui.html | — |
| **Grafana** | http://localhost:3000 | admin / admin |
| **Prometheus** | http://localhost:9090 | — |
| **Kafka UI** | http://localhost:8070 | — |
| **Mailpit** | http://localhost:8025 | — |
| **Keycloak** | http://localhost:8091 | admin / admin |
| **InfluxDB UI** | http://localhost:8072 | admin / admin123 |

---

## 🧪 Quick Pipeline Test

**1. Create a user:**
```bash
curl -X POST http://localhost:8080/api/v1/user \
  -H 'Content-Type: application/json' \
  -d '{"name":"John","surname":"Doe","email":"john@example.com","address":"123 Main St","alerting":true,"energyAlertingThreshold":1000.0}'
```

**2. Create a device:**
```bash
curl -X POST http://localhost:8081/api/v1/device/create \
  -H 'Content-Type: application/json' \
  -d '{"name":"Living Room Light","type":"LIGHT","location":"Living Room","userId":1}'
```

**3. Send an energy reading:**
```bash
curl -X POST http://localhost:8082/api/v1/ingestion \
  -H 'Content-Type: application/json' \
  -d '{"deviceId":1,"energyConsumed":500.0,"timestamp":"2025-01-01T12:00:00Z"}'
```

**4. Check results:**
- Kafka UI → http://localhost:8070 — see messages in `energy-usage` topic
- Mailpit → http://localhost:8025 — see alert emails when threshold exceeded
- Grafana → http://localhost:3000 — see live metrics dashboards

---

## 📊 Observability

- Each Spring Boot service exposes **`/actuator/prometheus`** metrics
- **Prometheus** scrapes all services and stores time-series metrics
- **Grafana** provides pre-built dashboards showing:
  - HTTP request rate per service
  - JVM heap usage
  - Circuit breaker state
  - Scrape target health

---

## 🗂️ Project Structure

```
home-energy-tracker/
├── api-gateway/          # Spring Cloud Gateway
├── user-service/         # User management
├── device-service/       # Device registry
├── ingestion-service/    # Energy reading ingestion + simulator
├── usage-service/        # Kafka consumer + InfluxDB + threshold logic
├── alert-service/        # Alert consumer + email sender
├── insight-service/      # AI insights (Spring AI + Ollama)
├── frontend/             # React dashboard
├── docker/               # Docker configs (Kafka, MySQL, Prometheus, Grafana)
├── docker-compose.yml    # Full infrastructure setup
└── README.md
```

---

## 🔮 Future Improvements

- End-to-end integration tests across the full pipeline
- CI/CD pipeline with GitHub Actions
- Kubernetes deployment with Helm charts
- WebSocket support for real-time frontend updates
- Mobile app for alerts and monitoring
- Centralized config with Spring Cloud Config Server

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

*Home Energy Tracker — a production-grade Spring Boot microservices project demonstrating real-world patterns: event streaming, polyglot persistence, API gateway, circuit breakers, observability, and a React frontend.*
