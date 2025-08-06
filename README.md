# ZalandoLite+ v2: Full Backend Microservices Project 📦

## 🚀 Overview

ZalandoLite+ v2 is a full-fledged microservices backend system simulating a fashion e-commerce platform—no UI, only REST APIs. You will build **seven microservices**, each secured using Google OAuth2, each containerized with Docker, and all communicating over the `zalando-backend` network.

This is a **group assignment** (2–3 students per group). Each student is responsible for at least **two microservices**.

---

## 🧩 Microservices List & Responsibilities

### 1. Auth Service *(Container: 9080)*

* Handles Google login using `spring-boot-starter-oauth2-client` (Thymeleaf optional).
* Issues JWT tokens (ID Token).
* Provides internal endpoint `/internal/token?userId=...` to issue the token to other services programmatically.

### 2. Product Service *(Container: 8586)*

* Manages product catalog (CRUD), categories, pricing.
* Uses PostgreSQL or MongoDB (chosen by your team).
* After creating a product, it calls Inventory Service to initialize stock with forwarded token.

### 3. Inventory Service *(Container: 8587)*

* Tracks stock per product.
* Exposes `GET /api/inventory/{productId}` & `POST /api/inventory`.
* Uses MongoDB (recommended) for document store flexibility.

### 4. Customer Service *(Container: 8588)*

* Manages user registration, VIP status, and preferences.
* Uses DB of your choice—Postgres if relational structure; Mongo for flexibility.

### 5. Order Service *(Container: 8589)*

* Processes new orders, validates stock, applies discounts, deducts inventory.
* Calls Product, Inventory, and Discount services, forwarding JWT token each time.

### 6. Discount Service *(Container: 8590)*

* Computes final price by applying business rules:

  * 10% off VIP customers
  * 20% off products in “Shoes” category
* Stateless logic service (DB optional if storing rules).

### 7. Review Service *(Container: 8591)*

* Allows customers to leave product reviews.
* Permits: `POST /api/reviews` and `GET /api/reviews/product/{productId}`.
* Uses document store (MongoDB recommended).

---

## 🔐 Token Flow & Internal Propagation

* Auth Service handles Google OAuth login and stores the token.
* All other services fetch this token silently via:

  ```
  GET http://auth-service:9080/internal/token?userId={customerId}
  ```
* Services then forward the token in their outgoing HTTP calls:

  ```java
  headers.set("Authorization", "Bearer " + fetchedToken);
  ```
* No token appears in browser or UI; the entire token flow is **implicit and invisible to end user**.

---

## 🐋 Standard Dockerfile Template

```dockerfile
# Stage 1 – Build
FROM maven:3.9.4-eclipse-temurin-17 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2 – Run
FROM eclipse-temurin:17-jdk-slim
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
EXPOSE ${PORT}
ENTRYPOINT ["java","-jar","/app/app.jar"]
```

---

## 🐳 Standard docker-compose.yml (each service)

```yaml
version: "3.8"

services:
  <service-name>:
    build: .
    container_name: <service-name>
    ports:
      - "${PORT}:${PORT}"
    env_file:
      - .env
    networks:
      - zalando-backend

networks:
  zalando-backend:
    external: true
```

**Before launching**, execute:

```bash
docker network create zalando-backend
```

All services must use this same network.

---

## ⚙️ Dependencies via Spring Initializr

* **All microservices**:

  * `spring-boot-starter-web`
  * `spring-boot-starter-security`
  * `spring-boot-starter-oauth2-resource-server`

* **Auth-Service only**:

  * Add `spring-boot-starter-oauth2-client`
  * Thymeleaf optional if UI login page is needed

* **To choose your DB**:

  * `spring-boot-starter-data-jpa` for PostgreSQL
  * `spring-boot-starter-data-mongodb` for MongoDB

---

## 🧠 Database Recommendations (Optional Per Team)

* **Product Service**: PostgreSQL—structured data, categories, relations.
* **Inventory Service**: MongoDB—fast document updates, flexible schema.
* **Working principle**: Decide per team, justify clearly in README.

---

## 📝 Development Flow Summary

1. Build Auth Service → authenticate and issue tokens.
2. Each other service fetches token from Auth Service internally.
3. Microservices call each other using RestTemplate/WebClient with forwarded token.
4. Use `.env` for variables (PORT, DB URL, etc.), not hardcoded.
5. Dockerize each service and run with:

   ```bash
   docker-compose up --build
   ```
6. Test via Postman: Login → obtain token → call secure endpoints with Authorization header.

---

## 📚 Bow-Tying Voice for you ;) 

> “You’re building a real-world backend. Every service has its identity, communicates securely, and trusts the central Auth Service to verify user legitimacy. No tokens in URL or UI—just clean efficient backend devotion. Each microservice is autonomous yet cohesive, scalable, and container-ready.”


