# Aegis Cybernetic Security Systems

A robust, secure, and scalable authentication and authorization framework designed to protect your applications with enterprise-grade security features.

## 📋 Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Usage](#usage)
- [API Endpoints](#api-endpoints)
- [Security Notes](#security-notes)
- [Configuration](#configuration)
- [Contributing Guidelines](#contributing-guidelines)
- [License](#license)
- [Support](#support)

## ✨ Features

- **Multi-Factor Authentication (MFA)**: Support for TOTP, SMS, and email-based authentication
- **OAuth 2.0 & OpenID Connect**: Industry-standard authentication protocols
- **Role-Based Access Control (RBAC)**: Fine-grained permission management
- **JWT Token Management**: Secure token generation and validation
- **Session Management**: Robust session handling with automatic expiration
- **Audit Logging**: Comprehensive audit trails for security compliance
- **Rate Limiting**: Built-in protection against brute force attacks
- **CORS Support**: Cross-origin resource sharing with configurable policies
- **Database Agnostic**: Works with multiple database backends
- **Microservices Ready**: Designed for scalable, distributed architectures
- **RESTful API**: Clean and intuitive REST endpoints
- **Comprehensive Error Handling**: Detailed error responses for debugging

## 🚀 Installation

### Prerequisites

- Node.js >= 14.0.0
- npm >= 6.0.0 or yarn >= 1.22.0
- Database (PostgreSQL, MySQL, MongoDB, or SQLite)

### Via npm

```bash
npm install aegis
```

### Via yarn

```bash
yarn add aegis
```

### From Source

```bash
git clone https://github.com/naqqibb/Aegis.git
cd Aegis
npm install
npm run build
```

## 🏃 Quick Start

### Basic Setup

```javascript
const Aegis = require('aegis');

const aegis = new Aegis({
  secret: process.env.JWT_SECRET,
  database: {
    type: 'postgres',
    host: 'localhost',
    port: 5432,
    username: 'user',
    password: 'password',
    database: 'aegis_db'
  }
});

// Initialize the service
await aegis.initialize();

// Start the server
aegis.listen(3000, () => {
  console.log('Aegis is running on port 3000');
});
```

### Express.js Integration

```javascript
const express = require('express');
const Aegis = require('aegis');

const app = express();
const aegis = new Aegis(config);

// Use Aegis middleware
app.use(aegis.middleware());

// Protected route example
app.get('/protected', aegis.authenticate, (req, res) => {
  res.json({ message: 'Access granted', user: req.user });
});

app.listen(3000);
```

## 📖 Usage

### User Registration

```javascript
const user = await aegis.register({
  email: 'user@example.com',
  password: 'securePassword123',
  firstName: 'John',
  lastName: 'Doe'
});
```

### User Login

```javascript
const result = await aegis.login({
  email: 'user@example.com',
  password: 'securePassword123'
});

console.log(result.token); // JWT token
console.log(result.refreshToken); // Refresh token
```

### Token Refresh

```javascript
const newToken = await aegis.refreshToken(refreshToken);
```

### Role Assignment

```javascript
await aegis.assignRole(userId, 'admin');
```

### Permission Check

```javascript
const hasPermission = await aegis.hasPermission(userId, 'delete:user');
```

### Enable MFA

```javascript
const mfaSetup = await aegis.setupMFA(userId, 'totp');
console.log(mfaSetup.qrCode); // QR code for authenticator app
```

### Verify MFA Token

```javascript
const verified = await aegis.verifyMFA(userId, mfaToken);
```

## 🔌 API Endpoints

### Authentication

#### Register User
```
POST /api/auth/register
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securePassword123",
  "firstName": "John",
  "lastName": "Doe"
}

Response: 201 Created
{
  "id": "uuid",
  "email": "user@example.com",
  "token": "eyJhbGc...",
  "refreshToken": "eyJhbGc..."
}
```

#### Login
```
POST /api/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securePassword123"
}

Response: 200 OK
{
  "token": "eyJhbGc...",
  "refreshToken": "eyJhbGc...",
  "expiresIn": 3600
}
```

#### Refresh Token
```
POST /api/auth/refresh
Content-Type: application/json

{
  "refreshToken": "eyJhbGc..."
}

Response: 200 OK
{
  "token": "eyJhbGc...",
  "expiresIn": 3600
}
```

#### Logout
```
POST /api/auth/logout
Authorization: Bearer <token>

Response: 200 OK
{
  "message": "Successfully logged out"
}
```

### User Management

#### Get Current User
```
GET /api/users/me
Authorization: Bearer <token>

Response: 200 OK
{
  "id": "uuid",
  "email": "user@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "roles": ["user"]
}
```

#### Update Profile
```
PUT /api/users/me
Authorization: Bearer <token>
Content-Type: application/json

{
  "firstName": "Jane",
  "lastName": "Smith"
}

Response: 200 OK
```

#### Change Password
```
PUT /api/users/me/password
Authorization: Bearer <token>
Content-Type: application/json

{
  "currentPassword": "oldPassword123",
  "newPassword": "newPassword456"
}

Response: 200 OK
```

### MFA Endpoints

#### Setup MFA
```
POST /api/mfa/setup
Authorization: Bearer <token>
Content-Type: application/json

{
  "method": "totp"
}

Response: 200 OK
{
  "secret": "JBSWY3DPEBLW64TMMQ......",
  "qrCode": "data:image/png;base64,..."
}
```

#### Verify MFA Setup
```
POST /api/mfa/verify
Authorization: Bearer <token>
Content-Type: application/json

{
  "token": "123456"
}

Response: 200 OK
{
  "backupCodes": ["code1", "code2", "..."]
}
```

#### Disable MFA
```
DELETE /api/mfa
Authorization: Bearer <token>

Response: 200 OK
{
  "message": "MFA disabled"
}
```

### Role & Permission Management

#### List Roles
```
GET /api/roles
Authorization: Bearer <token>

Response: 200 OK
[
  {
    "id": "uuid",
    "name": "admin",
    "permissions": ["create:user", "delete:user", "edit:user"]
  }
]
```

#### Assign Role to User
```
POST /api/users/:userId/roles
Authorization: Bearer <token>
Content-Type: application/json

{
  "roleId": "uuid"
}

Response: 200 OK
```

## 🔒 Security Notes

### Password Security
- Passwords are hashed using bcrypt with a minimum of 10 salt rounds
- Minimum password length: 12 characters (configurable)
- Password complexity requirements enforced (uppercase, lowercase, numbers, special characters)

### Token Security
- JWT tokens use HS256 or RS256 algorithms
- Tokens have configurable expiration times (default: 1 hour)
- Refresh tokens are rotated automatically
- Tokens are validated against a blacklist on logout

### HTTPS Only
- All API endpoints should be served over HTTPS in production
- HTTP Strict-Transport-Security (HSTS) headers are recommended

### Rate Limiting
- Default: 100 requests per 15 minutes per IP
- Login endpoint: 5 attempts per 15 minutes
- MFA verification: 10 attempts per 15 minutes

### SQL Injection Prevention
- All database queries use parameterized statements
- Input validation and sanitization on all endpoints

### CORS Configuration
- Configure allowed origins explicitly
- Avoid using wildcard (*) in production

### Secrets Management
- Store JWT secret in environment variables
- Use secure key management systems in production (AWS KMS, HashiCorp Vault)
- Rotate secrets periodically

### Audit Logging
- All authentication events are logged
- Track failed login attempts
- Monitor role and permission changes

## ⚙️ Configuration

### Environment Variables

```bash
# Server Configuration
PORT=3000
NODE_ENV=production

# Database Configuration
DB_TYPE=postgres
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=user
DB_PASSWORD=password
DB_NAME=aegis_db

# JWT Configuration
JWT_SECRET=your-super-secret-key-change-this
JWT_EXPIRATION=3600
REFRESH_TOKEN_EXPIRATION=604800

# MFA Configuration
MFA_ENABLED=true
TOTP_WINDOW=1

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100

# CORS Configuration
CORS_ORIGIN=http://localhost:3000
CORS_CREDENTIALS=true

# Email Configuration (for email-based MFA)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=your-email@gmail.com
SMTP_PASSWORD=app-password

# Security
LOG_LEVEL=info
LOG_FILE=logs/aegis.log
```

### Configuration File

Create an `aegis.config.js` file:

```javascript
module.exports = {
  server: {
    port: process.env.PORT || 3000,
    environment: process.env.NODE_ENV || 'development'
  },
  database: {
    type: process.env.DB_TYPE || 'postgres',
    host: process.env.DB_HOST || 'localhost',
    port: process.env.DB_PORT || 5432,
    username: process.env.DB_USERNAME,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    ssl: process.env.NODE_ENV === 'production'
  },
  jwt: {
    secret: process.env.JWT_SECRET,
    expiration: parseInt(process.env.JWT_EXPIRATION) || 3600,
    algorithm: 'HS256'
  },
  mfa: {
    enabled: process.env.MFA_ENABLED === 'true',
    methods: ['totp', 'email', 'sms']
  },
  cors: {
    origin: process.env.CORS_ORIGIN,
    credentials: process.env.CORS_CREDENTIALS === 'true'
  }
};
```

## 🤝 Contributing Guidelines

We welcome contributions! Please follow these guidelines:

### Getting Started

1. Fork the repository
2. Clone your fork: `git clone https://github.com/your-username/Aegis.git`
3. Create a new branch: `git checkout -b feature/your-feature-name`
4. Install dependencies: `npm install`

### Development Setup

```bash
# Install dependencies
npm install

# Run tests
npm test

# Run linting
npm run lint

# Build project
npm run build

# Start development server
npm run dev
```

### Code Standards

- Follow ESLint configuration
- Use prettier for code formatting: `npm run format`
- Write tests for all new features
- Maintain test coverage above 80%
- Follow conventional commits: `feat:`, `fix:`, `docs:`, etc.

### Testing

```bash
# Run all tests
npm test

# Run tests with coverage
npm run test:coverage

# Run tests in watch mode
npm run test:watch
```

### Commit Guidelines

- Use descriptive commit messages
- Format: `type(scope): message`
- Examples:
  - `feat(auth): add TOTP support`
  - `fix(jwt): handle token expiration correctly`
  - `docs: update installation instructions`

### Pull Request Process

1. Update documentation as needed
2. Add tests for new functionality
3. Ensure all tests pass: `npm test`
4. Run linter: `npm run lint`
5. Submit PR with clear description
6. Link related issues
7. Respond to review comments promptly

### Reporting Issues

- Use the GitHub issue tracker
- Provide clear, descriptive titles
- Include steps to reproduce
- Add relevant environment information
- Attach error logs/screenshots when applicable

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

You are free to:
- Use commercially
- Modify the code
- Distribute
- Use privately

With the conditions that:
- License and copyright notice must be included
- Changes must be documented

## 🆘 Support

### Documentation

- **Official Docs**: [https://aegis-docs.example.com](https://aegis-docs.example.com)
- **API Documentation**: [https://api-docs.example.com](https://api-docs.example.com)
- **Changelog**: [CHANGELOG.md](CHANGELOG.md)

### Getting Help

#### GitHub Issues
- Report bugs: [Issues](https://github.com/naqqibb/Aegis/issues)
- Feature requests: [Discussions](https://github.com/naqqibb/Aegis/discussions)

#### Community
- **Discord**: [Join our Discord server](https://discord.gg/example)
- **Slack**: [Join our Slack workspace](https://slack.example.com)
- **Stack Overflow**: Tag questions with `aegis-auth`

#### Email Support
- **General Inquiries**: support@aegis.example.com
- **Security Issues**: security@aegis.example.com

### Security Vulnerabilities

**Please do not open public GitHub issues for security vulnerabilities.**

Send vulnerability reports to: **security@aegis.example.com**

Include:
- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if available)

We will acknowledge receipt within 48 hours and provide updates every week.

### Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history and updates.

### Roadmap

Planned features and improvements are tracked in our [GitHub Projects](https://github.com/naqqibb/Aegis/projects).

---

**Made with ❤️ by [naqqibb](https://github.com/naqqibb) and contributors**

Last Updated: December 19, 2025
