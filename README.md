# BussinesCardGenerator

A professional REST API backend for generating customizable business card PDFs for academic staff. Built with Spring Boot 3.4.4, this application provides secure user authentication with JWT, email verification, and automated PDF generation for business cards tiled on A4 pages.

## Features

- **User Registration & Authentication**
  - Email-based registration with confirmation via verification token
  - JWT (JSON Web Token) authentication for secure API access
  - Email validation and confirmation workflow
  - Role-based access control (USER, ADMIN roles)

- **Business Card PDF Generation**
  - Generate professional business cards with academic information
  - Supports scientific titles, academic positions, contact details
  - Automatic tiling of multiple cards on A4 pages for efficient printing
  - Front and back card templates with institutional branding
  - PDF download directly from API

- **PDF History & Management**
  - Track all generated PDFs in MongoDB
  - Retrieve previously generated business cards
  - Historical data persistence and retrieval

- **Email Integration**
  - Automatic email confirmation on registration
  - Support for local SMTP servers (MailHog) for development
  - Email validation with regex patterns

- **Security**
  - JWT token-based authentication (24-hour expiration)
  - Password encoding with Spring Security
  - Protected endpoints requiring valid Bearer tokens
  - MongoDB-based user and session management

## Tech Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| **Framework** | Spring Boot | 3.4.4 |
| **Language** | Java | 17 |
| **Database** | MongoDB | Latest |
| **Authentication** | Spring Security + JWT (JJWT) | 0.11.5 |
| **PDF Generation** | iText PDF | 7.2.5 |
| **PDF Processing** | Apache PDFBox | 2.0.27 |
| **Document Processing** | Docx4J + Apache FOP | 11.5.3 / 2.11 |
| **Office Conversion** | JODConverter | 4.4.8 |
| **Email** | Spring Mail | Built-in |
| **Utilities** | Lombok | 1.18.36 |
| **Build Tool** | Maven | 3.x |

## Prerequisites

Before you begin, ensure you have the following installed:

- **Java Development Kit (JDK)** 17 or higher
- **Maven** 3.6.0 or higher (or use the included Maven wrapper)
- **MongoDB** (running locally on default port 27017)
- **MailHog** (optional, for email testing during development)
  - Download: [MailHog GitHub](https://github.com/mailhog/MailHog)
  - Default SMTP: localhost:1025

### Verify Installation

```bash
java -version
mvn -version
```

## Getting Started

### 1. Clone the Repository

```bash
git clone <repository-url>
cd BussinesCardGenerator
```

### 2. Configure Environment

The application comes with a default `application.properties` file. Review and adjust as needed:

```properties
# Server configuration
server.port=8080
server.address=0.0.0.0

# MongoDB configuration
spring.data.mongodb.uri=mongodb://localhost:27017
spring.data.mongodb.database=BussinesCardAppDB

# JWT configuration
jwt.secret=9d30680b0d404f0ea691e9a63c386030d4deb84771f6498cb4b1f5e7d8f0d4b1
jwt.expiration=PT24H

# Email configuration (MailHog)
spring.mail.host=localhost
spring.mail.port=1025

# PDF storage location (adjust to your system)
pdf.storage.location=/path/to/pdf/storage
```

### 3. Start MongoDB

```bash
# On Windows
mongod

# On macOS
brew services start mongodb-community

# On Linux
sudo systemctl start mongod
```

### 4. Start MailHog (Optional - for Email Testing)

```bash
# Download and run MailHog
./mailhog

# Web UI will be available at: http://localhost:1025
# SMTP server listens on: localhost:1025
```

### 5. Build the Project

```bash
# Using Maven
mvn clean install

# Or using the Maven wrapper
./mvnw clean install
```

### 6. Run the Application

```bash
# Using Maven
mvn spring-boot:run

# Or run the built JAR
java -jar target/demo-0.0.1-SNAPSHOT.jar
```

The application will start on `http://localhost:8080` and you should see output confirming successful startup.

## API Endpoints

### Authentication Endpoints

#### Register a New User

```http
POST /api/v1/registration
Content-Type: application/json

{
  "firstName": "John",
  "lastName": "Doe",
  "email": "john.doe@university.edu",
  "password": "SecurePassword123!"
}
```

**Response (201 Created):**
```json
{
  "message": "User registered successfully. Please check your email to confirm your account."
}
```

#### Confirm Email Registration

```http
GET /api/v1/registration/confirm?token=<confirmation-token>
```

The confirmation token is sent via email after registration. Clicking the link in the email or using this endpoint confirms the email address.

**Response (200 OK):**
```json
{
  "message": "Email confirmed successfully. You can now login."
}
```

#### Login and Get JWT Token

```http
POST /login
Content-Type: application/json

{
  "email": "john.doe@university.edu",
  "password": "SecurePassword123!"
}
```

**Response (200 OK):**
```json
{
  "token": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJqb2huLmRvZUB1bml2ZXJzaXR5LmVkdSIsImlhdCI6MTcwMDAwMDAwMCwiZXhwIjoxNzAwMDg2NDAwfQ.SignatureHere",
  "expiresIn": 86400,
  "userEmail": "john.doe@university.edu"
}
```

### Business Card PDF Endpoints

#### Generate Business Card PDF

```http
POST /pdf/generate
Authorization: Bearer <jwt-token>
Content-Type: application/json

{
  "scientificTitle": "Ph.D.",
  "firstName": "John",
  "lastName": "Doe",
  "academicPosition": "Assistant Professor",
  "officePhone": "+40 256 123 456",
  "mobilePhone": "+40 789 123 456",
  "email": "john.doe@university.edu",
  "website": "https://university.edu/~johndoe"
}
```

**Response (200 OK):**
```json
{
  "id": "507f1f77bcf86cd799439011",
  "fileName": "businesscard_2024_01_15_143022.pdf",
  "filePath": "/path/to/pdf/storage/businesscard_2024_01_15_143022.pdf",
  "userEmail": "john.doe@university.edu",
  "generatedAt": "2024-01-15T14:30:22.000Z",
  "status": "success"
}
```

#### Retrieve PDF History

```http
GET /pdf/history
Authorization: Bearer <jwt-token>
```

**Response (200 OK):**
```json
{
  "totalCount": 3,
  "pdfs": [
    {
      "id": "507f1f77bcf86cd799439011",
      "fileName": "businesscard_2024_01_15_143022.pdf",
      "generatedAt": "2024-01-15T14:30:22.000Z",
      "userEmail": "john.doe@university.edu"
    },
    {
      "id": "507f1f77bcf86cd799439012",
      "fileName": "businesscard_2024_01_14_095510.pdf",
      "generatedAt": "2024-01-14T09:55:10.000Z",
      "userEmail": "john.doe@university.edu"
    }
  ]
}
```

#### Download Business Card PDF

```http
GET /pdf/download/{pdfId}
Authorization: Bearer <jwt-token>
```

**Response (200 OK):** Binary PDF file download

### Example Usage with cURL

**1. Register a new user:**
```bash
curl -X POST http://localhost:8080/api/v1/registration \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "John",
    "lastName": "Doe",
    "email": "john.doe@university.edu",
    "password": "SecurePassword123!"
  }'
```

**2. Confirm email (use the token from the email):**
```bash
curl "http://localhost:8080/api/v1/registration/confirm?token=abc123def456..."
```

**3. Login to get JWT token:**
```bash
curl -X POST http://localhost:8080/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@university.edu",
    "password": "SecurePassword123!"
  }'
```

**4. Generate a business card PDF:**
```bash
curl -X POST http://localhost:8080/pdf/generate \
  -H "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "scientificTitle": "Ph.D.",
    "firstName": "John",
    "lastName": "Doe",
    "academicPosition": "Assistant Professor",
    "officePhone": "+40 256 123 456",
    "mobilePhone": "+40 789 123 456",
    "email": "john.doe@university.edu",
    "website": "https://university.edu/~johndoe"
  }'
```

**5. Retrieve PDF generation history:**
```bash
curl -X GET http://localhost:8080/pdf/history \
  -H "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9..."
```

## Configuration

### Key Configuration Properties

Edit `src/main/resources/application.properties` to customize:

| Property | Default | Description |
|----------|---------|-------------|
| `server.port` | 8080 | HTTP server port |
| `server.address` | 0.0.0.0 | Server bind address (accessible externally) |
| `spring.data.mongodb.uri` | mongodb://localhost:27017 | MongoDB connection string |
| `spring.data.mongodb.database` | BussinesCardAppDB | MongoDB database name |
| `jwt.secret` | [hardcoded] | Secret key for JWT signing (should be externalized in production) |
| `jwt.expiration` | PT24H | JWT token expiration duration (ISO 8601 format) |
| `spring.mail.host` | localhost | SMTP server hostname |
| `spring.mail.port` | 1025 | SMTP server port |
| `pdf.storage.location` | /path/to/pdfs | Directory where generated PDFs are stored |

### Production Configuration

For production deployment:

1. **Externalize Secrets**: Store `jwt.secret` in environment variables or a secure vault
   ```properties
   jwt.secret=${JWT_SECRET_ENV}
   ```

2. **Database**: Update MongoDB connection string with proper authentication
   ```properties
   spring.data.mongodb.uri=mongodb://user:password@prod-mongo-host:27017
   ```

3. **Email**: Configure with your production SMTP server
   ```properties
   spring.mail.host=smtp.yourdomain.com
   spring.mail.port=587
   spring.mail.username=${MAIL_USERNAME}
   spring.mail.password=${MAIL_PASSWORD}
   spring.mail.properties.mail.smtp.auth=true
   spring.mail.properties.mail.smtp.starttls.enable=true
   ```

4. **PDF Storage**: Use a reliable file storage solution
   ```properties
   pdf.storage.location=/opt/app/pdfs
   ```

## Project Structure

```
src/main/java/com/BussinesCardApp/demo/
├── BussinesCardApplication.java           # Main Spring Boot entry point
├── TestController.java                    # Test/health check endpoints
│
├── PDF/                                   # Business card PDF generation
│   ├── Controller/
│   │   ├── PDFExportController.java       # POST /pdf/generate endpoint
│   │   └── PDFHistoryController.java      # PDF retrieval and history
│   └── Service/
│       ├── BusinessCardDTO.java           # Data transfer object for card details
│       ├── BusinessCardMaker.java         # Card image rendering logic
│       ├── PDFDownloadController.java     # PDF download handling
│       ├── PDFExport.java                 # MongoDB document model
│       ├── PDFExportDTO.java              # DTO for API responses
│       ├── PDFGeneratorService.java       # PDF generation orchestrator
│       └── PDFRepository.java             # MongoDB repository
│
└── user/                                  # User management and authentication
    ├── appuser/                           # User account management
    │   ├── AppUser.java                   # User entity
    │   ├── AppUserRepository.java         # MongoDB repository
    │   ├── AppUserRole.java               # Enum: USER, ADMIN roles
    │   ├── AppUserService.java            # Business logic
    │   ├── UserAccountDTO.java            # DTO for account operations
    │   └── Controller/AccountController.java
    │
    ├── authentication/                    # JWT authentication
    │   ├── AuthenticationController.java  # POST /login endpoint
    │   ├── AuthenticationDTO.java         # Login request/response DTOs
    │   ├── AuthenticationService.java     # Auth business logic
    │   └── jwt/
    │       ├── JwtAuthenticationFilter.java # Token validation filter
    │       └── JwtGenerator.java          # Token generation logic
    │
    ├── email/                             # Email services
    │   ├── EmailSender.java               # Email sending interface
    │   └── EmailService.java              # Implementation
    │
    ├── registration/                      # User registration workflow
    │   ├── EmailValidator.java            # Email format validation
    │   ├── RegistrationController.java    # POST /api/v1/registration
    │   ├── RegistrationRequest.java       # Registration DTO
    │   ├── RegistrationService.java       # Registration logic
    │   └── token/
    │       ├── ConfirmationToken.java     # Email confirmation token entity
    │       ├── ConfirmationTokenRepository.java
    │       └── ConfirmationTokenService.java
    │
    └── security/                          # Security configuration
        ├── config/WebSecurityConfig.java  # Spring Security setup
        └── PasswordEncoder.java           # Password hashing configuration

src/main/resources/
├── application.properties                 # Configuration properties
├── PDFs/                                  # Generated PDF storage directory
├── templates/                             # Email templates (if any)
└── images/
    ├── SiglaUVTFata.png                   # UVT logo (front)
    ├── SiglaUVTSpate.png                  # UVT logo (back)
    └── Fundal.jpg                         # Card background image

pom.xml                                    # Maven dependencies and build config
```

## Development Workflow

### Building the Project

```bash
# Clean build
mvn clean install

# Build without running tests
mvn clean install -DskipTests

# Build with specific profile
mvn clean install -P dev
```

### Running Tests

```bash
# Run all tests
mvn test

# Run specific test class
mvn test -Dtest=UserServiceTest
```

### Code Structure

- **Controllers**: Handle HTTP requests and responses
- **Services**: Contain business logic and orchestration
- **Repositories**: Data access layer (MongoDB)
- **DTOs**: Data transfer objects for API contracts
- **Entities**: Domain models mapped to MongoDB collections
