# PayLink

PayLink is a bill-payment and digital wallet backend project developed as part of the Software Design course.

## Phase 1 – DevOps Foundation

This phase establishes the basic development and deployment environment.

### Technologies
- Java 25
- Maven
- Docker
- Docker Compose
- PostgreSQL 17
- GitHub Actions
- Checkstyle
- JUnit 5

### Project Structure

```text
PayLink/
├── .github/
│   └── workflows/
│       └── ci.yml
├── docs/
├── src/
│   ├── main/
│   │   └── java/
│   │       └── com/paylink/
│   │           └── PayLinkApplication.java
│   └── test/
│       └── java/
│           └── com/paylink/
│               └── PayLinkApplicationTest.java
├── Dockerfile
├── docker-compose.yml
├── checkstyle.xml
├── pom.xml
└── README.md
```

### Requirements
- Docker Desktop
- Git

### Run the Application

Clone the repository:

```bash
git clone https://github.com/mustafahaiitham-alt/PayLink.git
cd PayLink
```

Start the application and database:

```bash
docker compose up --build
```

Open the application:

http://localhost:8080

Expected response:

```text
PayLink Application is running!
```

### Stop the Application

```bash
docker compose down
```

### Run Tests

```bash
mvn clean test
```

### Run Code Style Check

```bash
mvn checkstyle:check
```

### Continuous Integration

GitHub Actions automatically runs the following checks on every push and on pull requests targeting the main branch:

1. Build the project.
2. Check code style using Checkstyle.
3. Run automated tests using JUnit 5.

