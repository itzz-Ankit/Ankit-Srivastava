# Journal Management & Sentiment Analysis Platform

A backend application built with Java and Spring Boot for managing personal journal entries through a secure REST API. The application provides authentication, journal CRUD operations, sentiment classification, weather enrichment, caching, validation, scheduled processing, email support, and API documentation.

The project was developed as a **60+ day backend engineering project**, with a focus on clean service-layer design, authentication, persistence, caching, external API integration, testing, and Docker-based local infrastructure.

---

## Overview

The platform allows authenticated users to:

- Create, read, update, and delete journal entries
- Associate journal entries with sentiment analysis
- Retrieve weather information associated with journal activity
- Authenticate securely using JWT
- Store passwords using password hashing rather than plain text
- Cache frequently requested data with Redis
- Receive application emails through Spring Mail
- Validate incoming API requests
- Handle application errors through centralized exception handling
- Expose API documentation through OpenAPI / Swagger
- Run required infrastructure through Docker Compose

An administrator endpoint is also provided for retrieving registered users.

---

## Architecture

```text
                         CLIENT
                           |
                           | HTTP / REST
                           v
                +-----------------------+
                |   Spring Boot API     |
                +-----------+-----------+
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
        AuthController  JournalController  AdminController
              |             |             |
              v             v             v
        AuthService    JournalService    UserService
              |             |             |
              |       +-----+------+      |
              |       |            |      |
              v       v            v      v
          Spring   MongoDB     Redis   Repository Layer
          Security     |           |
              |        |           |
              v        |           |
             JWT       |           |
                       |           |
                       v           v
                Journal Data   Cached Data
                       |
             +---------+---------+
             |                   |
             v                   v
      SentimentService     WeatherService
             |                   |
             v                   v
       Sentiment Result    External Weather API
             |
             v
       Journal Response

             Supporting Components
             ---------------------
             Validation
             Global Exception Handling
             Scheduled Jobs
             Email Service
             OpenAPI / Swagger
             Docker Compose
```

### Request Flow

```text
HTTP Request
     |
     v
Controller
     |
     v
Validation / Security
     |
     v
Service Layer
     |
     +-------------------+
     |                   |
     v                   v
Repository          External Service
     |                   |
     v                   v
 MongoDB              Weather API
     |
     v
 Redis Cache (where applicable)
     |
     v
DTO Response
     |
     v
HTTP Response
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Java 8 |
| Framework | Spring Boot 2.7.18 |
| Web | Spring MVC / REST APIs |
| Database | MongoDB |
| Persistence | Spring Data MongoDB |
| Security | Spring Security |
| Authentication | JWT (JJWT 0.12.6) |
| Password Security | PasswordEncoder |
| Validation | Jakarta/Javax Bean Validation through Spring Boot Validation |
| Cache | Redis |
| Email | Spring Boot Starter Mail |
| API Documentation | Springdoc OpenAPI / Swagger UI |
| Build Tool | Maven |
| Boilerplate Reduction | Lombok |
| Containerization | Docker |
| Local Infrastructure | Docker Compose |
| Testing | Spring Boot Test + Spring Security Test + Mockito/JUnit ecosystem |

---

## Core Modules

### 1. Authentication

The authentication module provides:

- User signup
- User login
- JWT-based authentication
- Password hashing
- Authentication filters
- Protected API access
- Role information for users

Main endpoints:

```text
POST /auth/signup
POST /auth/login
GET  /auth/health
```

### 2. Journal Management

Authenticated users can manage their own journal entries.

```text
GET    /journal
POST   /journal
GET    /journal/{id}
PUT    /journal/{id}
DELETE /journal/{id}
```

Each journal entry can contain:

- Title
- Content
- Sentiment
- Weather information
- MongoDB ObjectId

### 3. Sentiment Analysis

Journal content can be classified into three application-level sentiment categories:

```text
POSITIVE
NEGATIVE
NEUTRAL
```

The sentiment logic is isolated inside `SentimentService`, keeping the controller and persistence layers independent of the classification logic.

### 4. Weather Integration

The application integrates weather information through `WeatherService`.

Frequently requested weather data is cached using Redis to reduce repeated external API calls.

```text
Request
  |
  v
Redis Cache
  |
  +---- cache hit ----> Return cached weather
  |
  +---- cache miss ---> External Weather API
                              |
                              v
                         Store in Redis
                              |
                              v
                         Return result
```

### 5. Caching

Redis is used for caching data that can be expensive or unnecessary to retrieve repeatedly, particularly weather-related data.

This reduces repeated external API requests and improves response efficiency.

### 6. Scheduled Processing

The project contains scheduled application logic for periodic processing of user/journal-related operations.

### 7. Email Service

Spring Mail is integrated for application-level email operations.

### 8. Centralized Exception Handling

A global exception handler provides a consistent API-level response for validation failures, bad requests, and unexpected server errors.

---

## API Reference

### Authentication

#### Signup

```http
POST /auth/signup
Content-Type: application/json
```

```json
{
  "username": "ankit",
  "email": "ankit@example.com",
  "password": "Password@123"
}
```

#### Login

```http
POST /auth/login
Content-Type: application/json
```

```json
{
  "username": "ankit",
  "password": "Password@123"
}
```

The login response contains the authentication token used for protected requests.

### Journal

#### Create Entry

```http
POST /journal
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

```json
{
  "title": "My Day",
  "content": "Today was a productive day."
}
```

#### Get My Entries

```http
GET /journal
Authorization: Bearer <JWT_TOKEN>
```

#### Get Entry By ID

```http
GET /journal/{id}
Authorization: Bearer <JWT_TOKEN>
```

#### Update Entry

```http
PUT /journal/{id}
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

#### Delete Entry

```http
DELETE /journal/{id}
Authorization: Bearer <JWT_TOKEN>
```

### Administration

```http
GET /admin/all-users
Authorization: Bearer <JWT_TOKEN>
```

---

## Response Format

The REST API uses a common response wrapper for API responses.

```json
{
  "success": true,
  "message": "Operation completed successfully",
  "data": {}
}
```

This keeps successful and error responses consistent across controllers.

---

## Security Model

```text
Client
  |
  | Username + Password
  v
Authentication Endpoint
  |
  v
AuthenticationManager
  |
  v
UserDetailsService
  |
  v
PasswordEncoder
  |
  v
JWT Generation
  |
  v
Client receives JWT
  |
  | Authorization: Bearer <token>
  v
JWT Authentication Filter
  |
  v
Spring Security Context
  |
  v
Protected Controller
```

Security responsibilities include:

- Authentication through Spring Security
- JWT token generation and validation
- Password hashing
- Protected endpoints
- Role-based user information
- Request validation
- Centralized exception handling

Passwords are not returned as part of user-facing response DTOs.

---

## Project Structure

```text
src/
├── main/
│   ├── java/
│   │   ├── controller/
│   │   │   ├── AuthController.java
│   │   │   ├── JournalController.java
│   │   │   ├── UserController.java
│   │   │   ├── AdminController.java
│   │   │   └── PublicController.java
│   │   │
│   │   ├── services/
│   │   │   ├── AuthService.java
│   │   │   ├── JournalService.java
│   │   │   ├── UserService.java
│   │   │   ├── SentimentService.java
│   │   │   ├── WeatherService.java
│   │   │   └── EmailService.java
│   │   │
│   │   ├── repository/
│   │   │   ├── UserRepository.java
│   │   │   ├── JournalEntryRepository.java
│   │   │   └── UserRepositoryImpl.java
│   │   │
│   │   ├── entity/
│   │   │   ├── User.java
│   │   │   └── JournalEntry.java
│   │   │
│   │   ├── dto_request/
│   │   ├── dto_response/
│   │   ├── Mapper/
│   │   ├── Filter/
│   │   ├── Exception/
│   │   ├── enums/
│   │   ├── config/
│   │   ├── cache/
│   │   ├── Scheduler/
│   │   └── api/
│   │
│   └── resources/
│       └── application.properties / application.yml
│
├── test/
│   └── java/
│       └── services/
│
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── pom.xml
└── README.md
```

---

## Data Model

### User

The user model stores account information, roles, journal relationships, and sentiment-analysis configuration.

Conceptually:

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

MongoDB is used as the primary persistence layer.

---

## Docker

The repository includes Docker configuration for the application and its local infrastructure.

### Services

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

### Start Infrastructure

```bash
docker compose up --build
```

The application is exposed through the configured Docker port mapping.

### Environment Variables

Configure environment-specific values rather than committing secrets to source control.

```text
MONGODB_URI
REDIS_HOST
REDIS_PORT
WEATHER_API_KEY
PORT
```

---

## Local Development

### Prerequisites

- Java 8+
- Maven
- MongoDB
- Redis
- Docker Desktop (recommended for local infrastructure)

### Build

```bash
./mvnw clean package
```

On Windows:

```bat
mvnw.cmd clean package
```

### Run

```bash
./mvnw spring-boot:run
```

On Windows:

```bat
mvnw.cmd spring-boot:run
```

---

## API Documentation

When the application is running, Swagger UI is available at:

```text
http://localhost:8080/swagger-ui/index.html
```

OpenAPI documentation provides an interactive way to inspect and test the REST endpoints.

---

## Testing

The project includes automated tests covering application services and security-related behaviour.

Run the test suite with:

```bash
./mvnw test
```

On Windows:

```bat
mvnw.cmd test
```

Testing areas include service-layer behaviour, sentiment processing, journal operations, and email-related functionality.

---

## Engineering Practices

The project follows a layered backend structure:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
MongoDB
```

Additional cross-cutting concerns are separated into dedicated components:

- DTOs for API boundaries
- Mappers for entity/DTO conversion
- Security filters for JWT processing
- Global exception handling
- Configuration classes for infrastructure concerns
- Services for external API integration
- Redis for caching
- Scheduled components for background processing

This separation keeps business logic out of controllers and makes individual components easier to test and maintain.

---

## Development Timeline

**60+ days of development** focused on progressively building and refining the backend:

```text
Foundation
   ↓
REST APIs
   ↓
MongoDB Persistence
   ↓
Authentication & JWT
   ↓
DTO / Mapper Layer
   ↓
Validation & Exception Handling
   ↓
Sentiment Analysis
   ↓
Weather API Integration
   ↓
Redis Caching
   ↓
Email & Scheduled Processing
   ↓
Testing & API Documentation
   ↓
Dockerized Local Environment
```

---

## Current Scope

The project is intentionally focused on backend engineering rather than a frontend UI. Its primary goal is to demonstrate the design and implementation of a secure, maintainable Spring Boot REST backend with persistence, authentication, caching, external-service integration, validation, testing, and containerized infrastructure.

---

## License

This project is currently maintained as a personal software engineering project.
