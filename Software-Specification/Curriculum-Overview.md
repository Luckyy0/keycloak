# Curriculum Overview
# Tổng quan Giáo trình

## 1. The Complete Enterprise Keycloak Curriculum / Toàn bộ Giáo trình Keycloak Doanh nghiệp
This curriculum covers everything from Beginner to Enterprise Architect.
Giáo trình này bao phủ mọi thứ từ Người mới bắt đầu đến Kiến trúc sư Doanh nghiệp.

### 00. Prerequisites
- HTTP
- HTTPS
- Cookies
- Sessions
- CORS
- CSRF
- TLS
- SSL
- PKI
- Certificates
- DNS
- Reverse Proxy
- Load Balancer
- Cryptography
- Hash
- HMAC
- Symmetric Encryption
- Asymmetric Encryption
- RSA
- ECC
- AES
- Digital Signature
- JWT
- JWS
- JWE
- OAuth Terminology
- REST API Basics
- JSON
- XML
- Web Security
- OWASP Top 10
- XSS
- CSRF
- Clickjacking
- Replay Attack
- Session Fixation
- Session Hijacking

### 01. Introduction
- What is IAM
- What is Keycloak
- History
- Why Keycloak
- Enterprise Use Cases
- Keycloak Editions
- Architecture Overview

### 02. IAM Fundamentals
- Identity
- Authentication
- Authorization
- Federation
- Identity Provider
- Service Provider
- SSO
- MFA
- Passwordless
- Zero Trust
- Least Privilege

### 03. Keycloak Architecture
- Internal Architecture
- Components
- Realms
- Clients
- Users
- Groups
- Roles
- Sessions
- Cache
- Database
- Storage
- Events
- SPI
- Quarkus Architecture

### 04. Installation
- Local Installation
- Docker
- Docker Compose
- Kubernetes
- Operator
- Helm
- Production Installation
- HA Installation

### 05. Realms
- Realm Configuration
- Realm Settings
- Localization
- Themes
- Realm Import
- Realm Export

### 06. Users
- User Lifecycle
- Registration
- Email Verification
- User Profile
- Attributes
- Credentials
- Password Policies

### 07. Groups
- Group Hierarchy
- Group Roles
- Group Attributes
- Default Groups

### 08. Roles
- Realm Roles
- Client Roles
- Composite Roles
- Effective Roles

### 09. Clients
- Public Client
- Confidential Client
- Bearer Only
- Service Account
- Client Authentication
- Redirect URI
- PKCE
- Consent

### 10. Client Scopes
- Default Scope
- Optional Scope
- Scope Mapping
- Client Scope Evaluation

### 11. Protocol Mappers
- Built-in Mapper
- User Attribute Mapper
- Group Mapper
- Role Mapper
- Script Mapper
- Custom Mapper

### 12. Authentication
- Browser Flow
- Direct Grant
- Client Authentication
- Service Authentication
- OTP
- WebAuthn
- Passkeys
- Conditional Authentication

### 13. Authentication Flows
- Browser Flow
- Registration Flow
- Reset Credentials
- First Broker Login
- Post Broker Login
- Conditional Flow
- Sub Flow
- Execution
- Requirements
- Custom Flow

### 14. Authorization
- RBAC
- ABAC
- Fine-Grained Authorization
- Policy Based Access Control
- Decision Strategies

### 15. OAuth2
- RFC Overview
- OAuth Actors
- Authorization Code
- Authorization Code + PKCE
- Client Credentials
- Device Flow
- Refresh Token
- Token Revocation
- Token Introspection
- Discovery
- Dynamic Client Registration
- Security Best Current Practice
- Threat Model

### 16. OpenID Connect
- OIDC Discovery
- ID Token
- UserInfo
- Nonce
- State
- Prompt
- max_age
- Hybrid Flow
- Front Channel Logout
- Back Channel Logout
- Session Management
- Claims

### 17. SAML
- SAML Basics
- Assertions
- Metadata
- SAML Login
- SAML Logout
- SAML vs OIDC

### 18. Tokens
- JWT
- JWS
- JWE
- Access Token
- Refresh Token
- Offline Token
- ID Token
- Action Token
- Exchange Token
- Opaque Token
- Claims
- Signature
- Verification
- Expiration
- Audience
- Issuer
- Clock Skew
- Key Rotation
- JWKS Rotation

### 19. Identity Providers
- Google
- GitHub
- Facebook
- Azure AD
- Okta
- Generic OIDC
- Generic SAML

### 20. User Federation
- LDAP
- Active Directory
- Kerberos
- Read Only
- Import Mode
- Synchronization
- Caching

### 21. LDAP
- LDAP Schema
- LDAP Mapper
- Sync
- Password Validation

### 22. Active Directory
- AD Integration
- Group Mapping
- User Sync
- Password Policy

### 23. Kerberos
- Kerberos Basics
- SPNEGO
- Windows Login

### 24. Authorization Services
- Resources
- Scopes
- Policies
- Permissions
- Decision Strategies
- Evaluation
- Java Adapter
- Policy Enforcer

### 25. UMA
- User Managed Access
- Permission Ticket
- Protection API

### 26. Token Exchange
- Internal Exchange
- External Exchange
- Impersonation
- Delegation

### 27. Themes
- Login Theme
- Email Theme
- Admin Theme
- Account Theme

### 28. Events
- User Events
- Admin Events
- Event Listener SPI
- Kafka Integration
- Audit Logs

### 29. Admin REST API
- Authentication
- Users API
- Clients API
- Roles API
- Groups API
- Automation

### 30. Service Accounts
- M2M Authentication
- API Authentication
- Automation

### 31. Client Policies
- Security Policies
- Dynamic Client Registration
- FAPI

### 32. SPI Development
- SPI Architecture
- Provider SPI
- Factory
- Deployment
- Debugging

### 33. Custom Providers
- Custom Protocol Mapper
- Custom REST Endpoint
- Custom Event Listener

### 34. Custom Authenticators
- Custom Login
- Custom OTP
- Conditional Logic

### 35. Custom User Storage
- External Database
- REST Storage
- Legacy System Integration

### 36. Database
- Schema
- Tables
- Liquibase
- Migration
- Backup
- Restore

### 37. Cache
- Infinispan
- Cache Modes
- Distributed Cache
- Replication
- Invalidation

### 38. Cluster
- HA
- Sticky Session
- JGroups
- Split Brain
- Failover

### 39. Performance
- JVM
- Database Pool
- Cache
- Benchmark
- Load Testing

### 40. Security Hardening
- HTTPS
- CSP
- Cookie Security
- Key Rotation
- Secrets
- Vault
- OWASP
- OAuth2 BCP

### 41. Monitoring
- Health Check
- Metrics
- Prometheus
- Grafana
- OpenTelemetry
- Jaeger
- Loki

### 42. Troubleshooting
- invalid_client
- invalid_scope
- invalid_grant
- redirect_uri
- 401
- 403
- CORS
- CSRF
- Clock Skew
- SSL
- Token Expired

### 43. Docker
- Image
- Compose
- Volumes
- Networks
- Production Docker

### 44. Kubernetes
- Operator
- Helm
- StatefulSet
- ConfigMap
- Secret
- Ingress
- Autoscaling

### 45. Spring Security Fundamentals
- Security Filter Chain
- Authentication
- Authorization
- AuthenticationManager
- AuthenticationProvider
- SecurityContext
- Method Security
- AuthorizationManager
- Custom Filter

### 46. Spring Boot Integration
- OAuth2 Login
- OAuth2 Client
- Resource Server
- JWT Validation
- Opaque Token
- RBAC
- ABAC
- Method Security
- BFF
- Session Authentication
- Logout
- Refresh Token
- Token Relay
- Feign
- WebClient
- JWKS Cache
- Testing

### 47. Gateway Integration
- Spring Cloud Gateway
- API Gateway
- Token Relay
- Route Security

### 48. BFF Architecture
- Backend For Frontend
- Cookie Session
- CSRF
- Secure Cookie
- Token Handler Pattern

### 49. Frontend Integration
- React
- Angular
- Vue
- keycloak-js
- Silent Login
- Silent Refresh

### 50. Mobile Integration
- Android
- iOS
- Flutter
- React Native
- AppAuth

### 51. Microservice Integration
- Service-to-Service Authentication
- Client Credentials
- Token Relay
- Distributed Authorization

### 52. Enterprise Architecture
- Multi-Tenant
- Identity Broker
- Cross-Realm
- Multi-Region
- Disaster Recovery
- Blue-Green Deployment
- Canary Deployment
- Zero Trust Architecture

### 53. Migration & Upgrade
- Version Upgrade
- Realm Migration
- Data Migration
- Backup Strategy
- Rollback

### 54. Testing
- Unit Test
- Integration Test
- MockMvc
- Testcontainers
- WireMock
- End-to-End Test

### 55. Best Practices
- Naming Convention
- Realm Design
- Role Design
- Client Design
- Security Checklist
- Production Checklist
- Performance Checklist

### 56. Enterprise Projects
- Project 01 – Basic Login
- Project 02 – RBAC
- Project 03 – OAuth2 Client
- Project 04 – Spring Boot Resource Server
- Project 05 – BFF Architecture
- Project 06 – API Gateway
- Project 07 – Microservices
- Project 08 – LDAP Integration
- Project 09 – HA Cluster
- Project 10 – Enterprise IAM Platform

### 57. Capstone Project
- Keycloak Cluster
- PostgreSQL HA
- Redis
- Spring Boot Microservices
- Spring Cloud Gateway
- React SPA
- BFF
- Docker Compose
- Kubernetes
- Prometheus
- Grafana
- OpenTelemetry
- Loki
- Jaeger
- CI/CD
- Backup & Restore
- Production Deployment
- Security Hardening
- Performance Benchmark
- Monitoring
- Logging
- Disaster Recovery
- High Availability
- Multi-Tenant
- Zero Trust
- Complete Documentation
- End-to-End Testing
- Production Checklist

