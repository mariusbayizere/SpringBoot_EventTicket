# BookingEventTicket

Simple Spring Boot microservice for booking event tickets. It provides REST endpoints for Users, Events and Bookings and is designed to integrate with Kafka, Docker, Kubernetes, CI/CD, Swagger/OpenAPI and JWT/OAuth2 security.

## Application entry

Main class:
`/home/marius/SpringBoot_EventTicket/BookingEventTicket/src/main/java/com/example/BookingEventTicket/BookingEventTicketApplication.java`

## Overview

Core entities / tables:

- User — account info (username, email, hashed password, roles).
- Event — event metadata (name, description, location, start/end time, capacity, price).
- Booking — records of user bookings (eventId, userId, quantity, status, timestamps).

## Tech stack

- Java 21, Spring Boot
- Spring Web, Spring Data JPA, Spring Security
- PostgreSQL (recommended)
- Kafka (messaging)
- JWT / OAuth2
- Docker, Kubernetes
- Maven (mvnw)
- Swagger / OpenAPI (springdoc)
- Testcontainers (recommended for integration tests)

## API (base path: /api)

Authentication

- POST /api/auth/register
  - Body: { "username","email","password" }
  - 201 Created
- POST /api/auth/login
  - Body: { "username","password" }
  - Response: { "token": "<jwt>" }

Users

- GET /api/users — list users (admin)
- GET /api/users/{id}
- POST /api/users
- PUT /api/users/{id}
- DELETE /api/users/{id}

Events

- GET /api/events — supports pagination & filters: ?page=&size=&name=
- GET /api/events/{id}
- POST /api/events
  - Body: { "name","description","location","startTime(ISO8601)","endTime(ISO8601)","capacity","price" }
- PUT /api/events/{id}
- DELETE /api/events/{id}

Bookings

- GET /api/bookings — user sees own bookings; admin sees all
- GET /api/bookings/{id}
- POST /api/bookings
  - Body: { "eventId", "userId", "quantity" }
  - 201 Created, emits Kafka topic `booking.created` (optional)
- PUT /api/bookings/{id}
- DELETE /api/bookings/{id} — cancel booking, emits `booking.cancelled` (optional)

Health & docs

- GET /actuator/health
- Swagger UI: /swagger-ui/index.html
- OpenAPI JSON: /v3/api-docs

Kafka topics (suggested)

- booking.created
- booking.updated
- booking.cancelled
- event.updated

Security

- Use header `Authorization: Bearer <jwt>` for protected endpoints.
- Public: /api/auth/\*\* and (optionally) GET /api/events
- Admin permissions required for creating/updating/deleting events and listing all users/bookings.
- Store secrets as environment variables or Kubernetes Secrets (do not commit).

## Running locally

Prereqs: JDK 21, PostgreSQL (or Testcontainers), Kafka (or Testcontainers), Maven or use `./mvnw`.

Example env:

- SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/booking
- SPRING_DATASOURCE_USERNAME=postgres
- SPRING_DATASOURCE_PASSWORD=secret
- SPRING_KAFKA_BOOTSTRAP_SERVERS=localhost:9092
- JWT_SECRET=replace_with_secure_key

Build & run:

```bash
./mvnw clean package
./mvnw spring-boot:run
```

Run tests:

```bash
./mvnw test
```

Example curl

- Login

```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"user","password":"secret"}'
```

- Create booking (authenticated)

```bash
curl -X POST http://localhost:8080/api/bookings \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"eventId":1,"userId":2,"quantity":2}'
```

## Docker

Add a Dockerfile to build the fat JAR and run with environment variables for DB, Kafka and JWT.

Build & run:

```bash
docker build -t booking-event-ticket:latest .
docker run -e SPRING_DATASOURCE_URL=... -e SPRING_DATASOURCE_USERNAME=... \
  -e SPRING_DATASOURCE_PASSWORD=... -p 8080:8080 booking-event-ticket:latest
```

## Kubernetes

- Provide Deployment, Service, ConfigMap and Secrets.
- Use Secrets for DB creds and JWT signing keys.
- Use InitContainer or DB migration job for schema migrations.

## CI/CD suggestions

Pipeline steps:

1. Checkout
2. ./mvnw -B -DskipTests=false verify
3. Build and push Docker image
4. Deploy to staging (kubectl/Helm)
5. Run integration tests

## Testing

- Unit tests: JUnit + Mockito
- Integration tests: Testcontainers for Postgres & Kafka
- API tests: MockMvc or RestAssured

## Next steps

- Implement entities, DTOs, repositories, services and controllers.
- Add MapStruct for mapping DTOs.
- Implement JWT generation and role-based authorization.
- Add OpenAPI annotations and Postman collection for frontend.

## References

- Main application: src/main/java/com/example/BookingEventTicket/BookingEventTicketApplication.java
- Config: src/main/resources/application.properties (or application.yml)
- Build:
