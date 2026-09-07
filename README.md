# Personal Insights & Journal Platform

A secure backend application built with Java and Spring Boot for managing personal journal entries through REST APIs. The application combines authentication, authorization, journal lifecycle operations, sentiment classification, weather integration, Redis caching, email support, scheduled processing, validation, centralized exception handling, API documentation, automated testing, and Docker-based local infrastructure.

The project was developed with a focus on progressively improving backend architecture, security, maintainability, persistence, performance, integration, testing, and deployment readiness.

---

## Overview

The platform provides authenticated users with a private space to manage journal entries while applying supporting backend capabilities around those entries.

Core capabilities include:

- User registration and secure login
- JWT-based authentication with Spring Security
- Role-based access control
- Password hashing before persistence
- Journal create, read, update, and delete operations
- User-specific journal access
- Rule-based sentiment classification (`POSITIVE`, `NEGATIVE`, `NEUTRAL`)
- Weather information integration through an external API
- Redis caching to reduce repeated data/API access
- Email service integration
- Scheduled background processing
- DTO-based API contracts
- Entity-to-DTO and DTO-to-Entity mapping
- Request validation
- Centralized exception handling
- Swagger / OpenAPI documentation
- Automated testing with JUnit and Mockito
- Docker and Docker Compose support

---

## Architecture

```text
                              CLIENT
                       Postman / API Consumer
                                |
                                | HTTP / REST
                                v
                    +--------------------------+
                    |     Spring Boot API      |
                    +------------+-------------+
                                 |
                   +-------------+-------------+
                   |             |               |
                   v             v               v
             AuthController  JournalController  Admin/User APIs
                   |             |               |
                   +-------------+---------------+
                                 |
                                 v
                    +--------------------------+
                    |       Service Layer      |
                    +--------------------------+
                    | AuthService              |
                    | JournalEntryService     |
                    | UserService             |
                    | SentimentService        |
                    | WeatherService          |
                    | RedisService            |
                    | EmailService            |
                    +-------------+------------+
                                  |
                  +---------------+----------------+
                  |               |                |
                  v               v                v
             Repository        Redis       External Weather API
                  |
                  v
               MongoDB
                  |
                  v
             Response Entity
                  |
                  v
                Mapper
                  |
                  v
             Response DTO
                  |
                  v
                CLIENT


Security Flow
-------------

HTTP Request
     |
     v
JWT Authentication Filter
     |
     v
JWT Validation
     |
     v
Spring Security Context
     |
     v
Protected Controller


Error Flow
----------

Controller / Service
        |
        v
     Exception
        |
        v
GlobalExceptionHandler
        |
        v
Standard API Error Response


Cache Flow
----------

Service
  |
  v
Redis Cache
  |
  +---- Cache Hit  ----> Return cached data
  |
  +---- Cache Miss ----> Repository / External API
                              |
                              v
                           Store in Redis
```

---

## Request Lifecycle

```text
Client Request
     |
     v
Validation
     |
     v
Spring Security / JWT
     |
     v
Controller
     |
     v
Service Layer
     |
     +-------------------+--------------------+
     |                   |                    |
     v                   v                    v
Repository          Redis Cache         External Service
     |                   |                    |
     v                   |                    v
 MongoDB <---------------+              Weather API
     |
     v
Entity
     |
     v
Mapper
     |
     v
Response DTO
     |
     v
HTTP Response
```

---

## Tech Stack

| Category | Technology |
|---|---|
| Language | Java |
| Framework | Spring Boot |
| API | Spring MVC / REST |
| Security | Spring Security |
| Authentication | JWT / JJWT 0.12.6 |
| Password Security | PasswordEncoder |
| Database | MongoDB |
| Data Access | Spring Data MongoDB |
| Cache | Redis |
| Validation | Spring Boot Validation |
| Email | Spring Boot Starter Mail |
| API Documentation | Springdoc OpenAPI / Swagger UI |
| Build Tool | Maven |
| Boilerplate Reduction | Lombok |
| Testing | JUnit / Mockito / Spring Boot Test |
| Containerization | Docker |
| Local Orchestration | Docker Compose |
| External Integration | Weather API |

---

## Core Modules

### 1. Identity & Authentication

The authentication module handles account creation and login while integrating with Spring Security.

```text
Signup
  |
  +--> Validate input
  |
  +--> Check username/email uniqueness
  |
  +--> Hash password
  |
  +--> Assign default role
  |
  +--> Persist user
  |
  +--> Return UserResponse
```

Login:

```text
Username + Password
        |
        v
AuthenticationManager
        |
        v
Credential Verification
        |
        v
JWT Generation
        |
        v
AuthResponse
```

### 2. Authorization

Protected APIs are accessed using a valid JWT. Role information is used to control administrative access.

### 3. Journal Lifecycle Management

Authenticated users can create, retrieve, update, and delete their journal entries.

```text
POST   /journal
GET    /journal
GET    /journal/{id}
PUT    /journal/{id}
DELETE /journal/{id}
```

Journal operations are processed through the service layer before persistence.

### 4. Sentiment Classification

`SentimentService` classifies journal text into application-defined categories:

```text
POSITIVE
NEGATIVE
NEUTRAL
```

This is a rule-based application service; the project does not use an ML/LLM model for sentiment analysis.

### 5. Weather Integration

`WeatherService` communicates with an external weather API and returns a dedicated response model. Redis is used to cache weather data to reduce repeated external calls.

### 6. Redis Cache Layer

Redis provides fast access to cached values and reduces unnecessary database or external API requests.

### 7. Notification / Email Service

The application integrates Spring Mail for backend email operations.

### 8. Scheduled Background Processing

Scheduled components execute periodic background operations without requiring an incoming API request.

### 9. API Data Contracts

Request and response DTOs keep API contracts separate from persistent entities.

```text
Client
  |
  v
Request DTO
  |
  v
Service
  |
  v
Entity
  |
  v
Repository
  |
  v
Entity
  |
  v
Mapper
  |
  v
Response DTO
```

### 10. Centralized Error Handling

`GlobalExceptionHandler` centralizes API error handling and provides consistent HTTP responses for validation failures, bad requests, and unexpected errors.

---

## API Reference

### Health Check

```http
GET /auth/health
```

### Signup

```http
POST /auth/signup
Content-Type: application/json
```

Request:

```json
{
  "username": "ankit",
  "email": "ankit@example.com",
  "password": "Password@123"
}
```

### Login

```http
POST /auth/login
Content-Type: application/json
```

Request:

```json
{
  "username": "ankit",
  "password": "Password@123"
}
```

Response:

```json
{
  "token": "<JWT_TOKEN>",
  "username": "ankit",
  "role": "USER"
}
```

### Authenticated Journal Request

```http
Authorization: Bearer <JWT_TOKEN>
```

Create:

```http
POST /journal
```

Request:

```json
{
  "title": "My Day",
  "content": "Today was a productive day."
}
```

Read all for authenticated user:

```http
GET /journal
```

Read by id:

```http
GET /journal/{id}
```

Update:

```http
PUT /journal/{id}
```

Delete:

```http
DELETE /journal/{id}
```

### Admin

```http
GET /admin/all-users
```

Administrative endpoints require the application's configured authorization rules.

> Endpoint names should always be kept in sync with the current controller mappings.

---

## Uniform API Response

The application uses a common response wrapper for structured REST responses.

Example:

```json
{
  "success": true,
  "message": "Operation completed successfully",
  "data": {}
}
```

This provides a consistent response contract across API modules.

---

## Security Model

```text
                       AUTHENTICATION
                              |
                 +------------+------------+
                 |                         |
               Signup                    Login
                 |                         |
                 v                         v
          PasswordEncoder          AuthenticationManager
                 |                         |
                 v                         v
            Store User              Verify Credentials
                                           |
                                           v
                                      Generate JWT
                                           |
                                           v
                                         Client
                                           |
                               Authorization: Bearer <JWT>
                                           |
                                           v
                                JWT Authentication Filter
                                           |
                                           v
                                  Spring Security Context
                                           |
                                           v
                                   Protected Controller
```

Passwords are hashed before persistence and are not returned through response DTOs.

---

## Project Structure

```text
src/
├── main/
│   ├── java/
│   │   ├── api/
│   │   │   └── response/
│   │   │       └── WeatherResponse.java
│   │   │
│   │   ├── cache/
│   │   │   └── AppCache.java
│   │   │
│   │   ├── config/
│   │   │   ├── RedisConfig.java
│   │   │   └── SwaggerConfig.java
│   │   │
│   │   ├── controller/
│   │   │   ├── AdminController.java
│   │   │   ├── AuthController.java
│   │   │   ├── JournalController.java
│   │   │   ├── PublicController.java
│   │   │   └── UserController.java
│   │   │
│   │   ├── dto_request/
│   │   │   ├── LoginRequest.java
│   │   │   ├── SignupRequest.java
│   │   │   ├── JournalEntryRequest.java
│   │   │   └── JournalUpdateRequest.java
│   │   │
│   │   ├── dto_response/
│   │   │   ├── ApiResponse.java
│   │   │   ├── AuthResponse.java
│   │   │   ├── JournalEntryResponse.java
│   │   │   └── UserResponse.java
│   │   │
│   │   ├── entity/
│   │   │   ├── ConfigJournalAppEntity.java
│   │   │   ├── JournalEntry.java
│   │   │   └── User.java
│   │   │
│   │   ├── enums/
│   │   │   └── Sentiment.java
│   │   │
│   │   ├── Exception/
│   │   │   └── GlobalExceptionHandler.java
│   │   │
│   │   ├── Filter/
│   │   │   └── JwtFilter.java
│   │   │
│   │   ├── Mapper/
│   │   │   ├── UserMapper.java
│   │   │   └── journalMapper.java
│   │   │
│   │   ├── repository/
│   │   │   ├── ConfigJournalAppRepository.java
│   │   │   ├── JournalEntryRepository.java
│   │   │   ├── UserRepository.java
│   │   │   └── UserRepositoryImpl.java
│   │   │
│   │   ├── Scheduler/
│   │   │   └── UserScheduler.java
│   │   │
│   │   └── services/
│   │       ├── AuthService.java
│   │       ├── EmailService.java
│   │       ├── JournalService.java
│   │       ├── RedisService.java
│   │       ├── SentimentService.java
│   │       ├── UserDetailsServiceImpl.java
│   │       ├── UserService.java
│   │       └── WeatherService.java
│   │
│   └── resources/
│       └── application configuration
│
├── test/
│   └── java/
│       └── application tests
│
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── .gitignore
├── pom.xml
└── README.md
```

---

## Data Model

### User

```text
User
├── id
├── username
├── email
├── password (hashed)
├── roles
├── journalEntries
└── sentimentAnalysis
```

### Journal Entry

```text
JournalEntry
├── id
├── title
├── content
├── sentiment
└── weather
```

MongoDB is the persistent data store for application entities.

---

## Redis Caching

```text
Service Request
      |
      v
   Redis
      |
   +--+----------------+
   |                   |
 Cache Hit          Cache Miss
   |                   |
   v                   v
Return Value      Repository / API
                       |
                       v
                  Store in Redis
                       |
                       v
                    Response
```

The weather service uses a cache key based on the requested city so repeated weather lookups can be served from Redis when available.

---

## Docker

The project includes:

- `Dockerfile` for the Spring Boot application
- `docker-compose.yml` for local application infrastructure
- `.dockerignore` to reduce unnecessary Docker build context

### Docker Compose Services

```text
Docker Compose
     |
     +-------------------+
     |                   |
     v                   v
  MongoDB              Redis
     |                   |
     +---------+---------+
               |
               v
        Spring Boot App
```

Start:

```bash
docker compose up --build
```

Stop:

```bash
docker compose down
```

The current Compose configuration uses MongoDB and Redis containers and exposes the application through its configured port mapping.

---

## Configuration

Environment-specific values should be supplied through environment variables or local configuration rather than committed secrets.

Typical values include:

```text
MONGODB_URI
REDIS_HOST
REDIS_PORT
WEATHER_API_KEY
PORT
```

Do not commit passwords, API keys, JWT secrets, or mail credentials to GitHub.

---

## Local Development

### Prerequisites

- Java
- Maven
- MongoDB
- Redis
- Docker Desktop (recommended)

### Build

```bash
./mvnw clean package
```

Windows:

```bat
mvnw.cmd clean package
```

### Run

```bash
./mvnw spring-boot:run
```

Windows:

```bat
mvnw.cmd spring-boot:run
```

---

## API Documentation

Swagger UI:

```text
http://localhost:8080/swagger-ui/index.html
```

The OpenAPI definition provides an interactive view of the available REST endpoints.

---

## Testing

The project uses JUnit, Mockito, Spring Boot Test, and Spring Security Test dependencies for automated testing.

Run:

```bash
./mvnw test
```

Windows:

```bat
mvnw.cmd test
```

The test suite covers application behaviour such as services, sentiment processing, journal functionality, email-related operations, and security support present in the repository.

---

## Engineering Practices

The project follows a layered architecture and keeps responsibilities separated:

```text
Controller
    |
    v
Service
    |
    +----> Repository ----> MongoDB
    |
    +----> Redis
    |
    +----> External API
```

Supporting concerns are handled separately:

- DTOs define API contracts
- Mappers isolate Entity ↔ DTO conversion
- Security filters process JWT authentication
- Global exception handling standardizes error responses
- Validation prevents malformed requests from reaching business logic
- Configuration classes centralize infrastructure setup
- Scheduled components handle background work

---

## Development Timeline

The project was developed over **60+ days**, progressing through multiple backend engineering stages:

```text
Foundation
    ↓
REST API Development
    ↓
MongoDB Persistence
    ↓
Authentication & Authorization
    ↓
JWT Security
    ↓
DTO & Mapper Architecture
    ↓
Validation & Exception Handling
    ↓
Journal Lifecycle
    ↓
Sentiment Classification
    ↓
Weather API Integration
    ↓
Redis Caching
    ↓
Email & Scheduled Processing
    ↓
Swagger Documentation
    ↓
JUnit / Mockito Testing
    ↓
Dockerization
    ↓
Refactoring & Integration
```

The development process focused on making the backend more structured, secure, maintainable, testable, and deployment-ready rather than implementing only basic CRUD functionality.

---

## Key Design Decisions

### DTOs Instead of Exposing Entities

API input and output models are separated from persistence entities to reduce coupling and avoid exposing internal fields.

### Stateless JWT Authentication

JWT allows protected API requests to be authenticated without maintaining traditional server-side sessions.

### Redis for Frequently Accessed Data

Caching reduces repeated access to the database and external weather service.

### Service-Layer Business Logic

Controllers remain focused on HTTP handling while application behaviour stays inside dedicated services.

### Centralized Exception Handling

A single global handler keeps API error responses consistent instead of duplicating exception logic in every controller.

---

## Current Scope

The project is focused on backend engineering and REST APIs. Its main objective is to demonstrate practical Java and Spring Boot development across authentication, authorization, persistence, caching, external API integration, validation, background processing, testing, documentation, and containerized local infrastructure.

---

## Roadmap

Future improvements can include:

- Refresh-token based authentication
- Pagination and advanced journal search
- API versioning
- Rate limiting
- More comprehensive integration testing
- CI/CD automation
- Production observability and metrics
- Cloud deployment

---

## Author

**Ankit Srivastava**

Java | Spring Boot | Backend Development
