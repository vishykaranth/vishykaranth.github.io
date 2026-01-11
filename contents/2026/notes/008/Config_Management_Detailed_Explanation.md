# Config Management Service - Detailed Explanation

## Executive Summary

**Config Management Service (CMS)** is an enterprise-grade configuration and secrets management platform built with Spring Boot 3.3.11 and Java 17. It provides centralized configuration management for multi-tenant applications, supporting hierarchical configuration storage, secrets management via HashiCorp Vault, configuration validation against schemas, and preference management for organizations and users.

---

## 1. Project Overview

### 1.1 Core Purpose
- **Centralized Configuration Management**: Store and retrieve application configurations for multi-tenant environments
- **Secrets Management**: Secure storage and retrieval of sensitive data (API keys, passwords, certificates) via HashiCorp Vault
- **Configuration Validation**: Schema-based validation of configurations using JSON schemas
- **Preference Management**: User and organization-level preference storage and retrieval
- **Multi-Environment Support**: Environment-specific and application-specific configurations
- **Configuration Hierarchy**: Support for tenant → environment → app → org level configurations

### 1.2 Technical Specifications
- **Language**: Java 17
- **Framework**: Spring Boot 3.3.11
- **Build Tool**: Maven
- **Storage Backend**: Consul (for configurations), HashiCorp Vault (for secrets)
- **API Documentation**: OpenAPI 3.0 (SpringDoc)
- **Ports**: 8000 (application), 8008 (management/actuator)
- **Context Path**: `/config-management`

---

## 2. Architecture & Design

### 2.1 Application Architecture

```
CmsApplication (Spring Boot Entry Point)
├── Controllers (REST API Layer)
│   ├── ConfigApiController          # Configuration CRUD operations
│   ├── ConfigSecretsController      # Secrets management
│   ├── ConnInterfaceApiController   # Connector interface operations
│   ├── MediatorConfigApiController   # Mediator configurations
│   ├── PreferenceController          # User/org preferences
│   ├── ServiceConfigApiController    # Service configurations
│   └── GlobalExceptionHandler        # Error handling
├── Services (Business Logic Layer)
│   ├── ConfigService                 # Core config operations
│   ├── SecretFetcherService          # Secret retrieval
│   ├── ComponentLibraryService       # Component metadata
│   ├── OrgManagementService          # Organization management
│   ├── ModelRepoService              # Schema retrieval
│   ├── PreferenceService             # Preference management
│   └── IamService                    # IAM integration
├── Repositories (Data Access Layer)
│   ├── ConfigRepository              # Consul-based config storage
│   └── SecretRepository              # Vault-based secret storage
├── DTOs (Data Transfer Objects)
│   ├── ConfigKey                     # Configuration key structure
│   ├── ConfigDto                     # Configuration data
│   ├── PreferenceDTO                 # Preference data
│   └── ...
└── Utilities
    ├── ConfigUtil                     # Configuration utilities
    ├── SecretParser                   # Secret extraction
    ├── ConfigMerger                   # Configuration merging
    └── ConfigPropertyFormatter        # Property format conversion
```

### 2.2 Key Components

#### **Controllers (7 REST Controllers)**
1. **ConfigApiController**: 
   - Configuration CRUD operations (GET, POST, PUT, PATCH, DELETE)
   - Configuration retrieval with validation
   - Configuration format conversion (JSON to properties)
   - Multi-format support (JSON, properties)

2. **ConfigSecretsController**:
   - Secret retrieval by path
   - Bulk secret fetching
   - Secret authorization checks

3. **ConnInterfaceApiController**:
   - Connector interface operations
   - Integration configurations

4. **MediatorConfigApiController**:
   - Mediator service configurations

5. **PreferenceController**:
   - Preference group CRUD
   - User/org preference management
   - Typed preferences

6. **ServiceConfigApiController**:
   - Service-specific configurations

#### **Services (8+ Service Implementations)**
1. **ConfigServiceImpl**: 
   - Core configuration operations
   - Schema validation
   - Secret parsing and separation
   - Configuration merging
   - Reference resolution

2. **SecretFetcherServiceImpl**:
   - Secret retrieval from Vault
   - Secret path resolution
   - Authorization validation

3. **ComponentLibraryService**:
   - Component metadata retrieval
   - Config ID resolution

4. **OrgManagementService**:
   - Organization hierarchy management
   - Org-based configuration scoping

5. **PreferenceServiceImpl**:
   - Preference storage and retrieval
   - Preference merging
   - Typed preference support

6. **ModelRepoService**:
   - Schema retrieval from model repository
   - Configuration schema validation

#### **Repositories (2 Repository Implementations)**
1. **ConsulConfigRepositoryImpl**:
   - Configuration storage in Consul KV store
   - Configuration retrieval
   - Circuit breaker pattern for resilience

2. **SecretRepositoryImpl**:
   - Secret storage in HashiCorp Vault
   - Versioned secret management
   - Secret file support

---

## 3. Core Features & Capabilities

### 3.1 Configuration Management

#### **Configuration Types**
- **DEPLOYMENT**: Deployment-time configurations
- **RUNTIME**: Runtime configurations

#### **Configuration Hierarchy**
```
tenant/{tenantId}/conf/{configId}/env/{env}/app/{appId}/org/{orgId}/type/{DEPLOYMENT|RUNTIME}
```

**Hierarchy Levels**:
1. **Tenant Level**: Base configuration for tenant
2. **Environment Level**: Environment-specific overrides (dev, staging, prod)
3. **Application Level**: Application-specific configurations
4. **Organization Level**: Organization-specific overrides
5. **App Stage Level**: Stage-specific configurations (alpha, beta, production)

#### **Configuration Operations**
- **GET**: Retrieve configuration with optional validation and format conversion
- **POST**: Create new configuration
- **PUT**: Update entire configuration
- **PATCH**: Partial configuration update (merge)
- **DELETE**: Remove configuration
- **MOVE**: Move configuration to different path

#### **Configuration Features**
- **Schema Validation**: Validate configurations against JSON schemas
- **Format Conversion**: Convert JSON to properties format
- **Config References**: Support for referencing other component configurations
- **Secret Separation**: Automatically separate secrets from regular config
- **Config Merging**: Merge configurations from multiple sources
- **Multi-Format Support**: JSON and properties format

### 3.2 Secrets Management

#### **Secret Storage**
- **Backend**: HashiCorp Vault with versioned KV engine
- **Path Structure**: `{vaultRootPath}/{configPath}/config` or `{vaultRootPath}/{configPath}/files`
- **Versioning**: Vault versioned KV for secret history

#### **Secret Operations**
- **GET**: Retrieve secrets by path (single or bulk)
- **POST**: Bulk secret retrieval
- **Authorization**: IAM-based authorization checks
- **Secret Files**: Support for file-based secrets

#### **Secret Features**
- **Automatic Extraction**: Secrets automatically extracted from configurations
- **Secure Storage**: Secrets stored separately in Vault
- **Path-based Access**: Hierarchical secret paths
- **Multi-key Support**: Support for multiple secrets per path

### 3.3 Preference Management

#### **Preference Types**
- **Organization Preferences**: Org-level preferences
- **User Preferences**: User-level preferences
- **Typed Preferences**: Type-safe preference management

#### **Preference Operations**
- **Create**: Create preference groups
- **Get**: Retrieve preferences by ID or name
- **Update**: Update preference groups
- **Delete**: Remove preference groups
- **List**: List all preferences
- **Merged**: Get merged preferences (org + user)

#### **Preference Features**
- **Hierarchical Merging**: Merge org and user preferences
- **Type Safety**: Typed preference support
- **Multi-tenant**: Tenant-scoped preferences

### 3.4 Configuration Validation

#### **Schema-Based Validation**
- **Schema Source**: Model Repository service
- **Schema Format**: JSON Schema (Protobuf Attributes)
- **Validation Rules**: Type checking, required fields, format validation

#### **Validation Features**
- **Optional Validation**: Configurable validation flag
- **Schema Retrieval**: Automatic schema retrieval from model repo
- **Error Reporting**: Detailed validation error messages

### 3.5 Configuration Formatting

#### **Format Support**
- **JSON**: Default JSON format
- **Properties**: Java properties format conversion

#### **Format Features**
- **Property Conversion**: Convert nested JSON to flat properties
- **Prefix Support**: Property prefix configuration
- **Reference Resolution**: Resolve config references in properties format

---

## 4. Storage Architecture

### 4.1 Consul (Configuration Storage)

**Purpose**: Store non-sensitive configuration data

**Key Features**:
- **KV Store**: Key-value storage for configurations
- **Path Structure**: Hierarchical path structure
- **Circuit Breaker**: Resilience4j circuit breaker for fault tolerance
- **Token-based Auth**: Consul token authentication

**Path Format**:
```
config/tenant/{tenantId}/conf/{configId}/env/{env}/app/{appId}/org/{orgId}/type/{type}
```

**Operations**:
- `saveConfig()`: Store configuration
- `getConfig()`: Retrieve configuration
- `isConfigExists()`: Check existence
- `deleteConfig()`: Remove configuration

### 4.2 HashiCorp Vault (Secrets Storage)

**Purpose**: Store sensitive data (secrets, credentials, certificates)

**Key Features**:
- **Versioned KV Engine**: Versioned key-value storage
- **Secret Engine**: Configurable secret engine name
- **Root Path**: Configurable root path for secrets
- **File Support**: Support for file-based secrets

**Path Format**:
```
{vaultRootPath}/tenant/{tenantId}/conf/{configId}/env/{env}/app/{appId}/org/{orgId}/type/{type}/config
{vaultRootPath}/tenant/{tenantId}/conf/{configId}/env/{env}/app/{appId}/org/{orgId}/type/{type}/files
```

**Operations**:
- `saveSecret()`: Store secret
- `saveSecretFile()`: Store secret file
- `getSecret()`: Retrieve secret
- `getSecretFile()`: Retrieve secret file
- `deleteSecret()`: Remove secret

---

## 5. API Endpoints

### 5.1 Configuration APIs

#### **Get Configuration**
```
GET /config/v1/tenant/{tenantId}/component/{componentId}/type/{type}
Query Parameters:
  - env: Environment (optional)
  - app: Application instance ID (optional)
  - app_stage: Application stage (optional)
  - validate: Validate against schema (default: false)
  - format: Output format (json, property) (optional)
```

#### **Create Configuration**
```
POST /config/v1/tenant/{tenantId}/component/{componentId}/type/{type}
Body: JSON configuration
Query Parameters:
  - env: Environment (optional)
  - app: Application instance ID (optional)
  - app_stage: Application stage (optional)
```

#### **Update Configuration**
```
PUT /config/v1/tenant/{tenantId}/component/{componentId}/type/{type}
Body: JSON configuration
Query Parameters: Same as create
```

#### **Patch Configuration**
```
PATCH /config/v1/tenant/{tenantId}/component/{componentId}/type/{type}
Body: JSON patch
Query Parameters: Same as create
```

#### **Delete Configuration**
```
DELETE /config/v1/tenant/{tenantId}/component/{componentId}/type/{type}
Query Parameters: Same as create
```

### 5.2 Secrets APIs

#### **Get Secrets (Bulk)**
```
POST /config/v1/secrets
Body: Array of secret paths
Response: Map of secret paths to values
```

#### **Get Secret by Path (Deprecated)**
```
GET /config/v1/secrets?path={secretPath}
Response: Secret value (octet-stream)
```

### 5.3 Preference APIs

#### **Create Preference Group**
```
POST /cms/api/v1/preferences
Headers:
  - X-Jiffy-Tenant-Id: Tenant ID
  - X-Jiffy-App-Id: Application ID
Query Parameters:
  - orgId: Organization ID (optional)
  - iamUserId: User ID (optional)
  - queryTenantId: Query tenant ID (optional)
Body: CreatePreferenceGroupDTO
```

#### **Get Preference Group**
```
GET /cms/api/v1/preferences/{groupId}
GET /cms/api/v1/preferences/by-name/{groupName}
```

#### **Update Preference Group**
```
PUT /cms/api/v1/preferences/{groupId}
PUT /cms/api/v1/preferences/by-name/{groupName}
```

#### **Delete Preference Group**
```
DELETE /cms/api/v1/preferences/{groupId}
DELETE /cms/api/v1/preferences/by-name/{groupName}
```

#### **List Preferences**
```
GET /cms/api/v1/preferences
GET /cms/api/v1/preferences/merged
GET /cms/api/v1/preferences/typed
```

### 5.4 Connector Interface APIs

#### **Get Interface Operations**
```
GET /config/v1/interface/operation/{operationName}
Query Parameters:
  - tenantId: Tenant ID (optional)
  - env: Environment (optional)
  - app: Application instance ID (optional)
  - app_stage: Application stage (optional)
  - validate: Validate (optional)
```

---

## 6. Configuration Processing Pipeline

### 6.1 Configuration Retrieval Flow

```
1. Request → ConfigApiController.getConfig()
2. Build ConfigKey from path parameters
3. ConfigService.getConfig()
   ├── ConfigRepository.getConfig() → Consul
   ├── If validate=true:
   │   ├── ModelRepoService.getSchema() → Get schema
   │   └── Validate config against schema
   ├── If format=property:
   │   ├── ConfigPropertyFormatter → Convert to properties
   │   └── ConfigReferenceExtractor → Extract references
   │   └── Recursively resolve references
   └── Return formatted config
```

### 6.2 Configuration Storage Flow

```
1. Request → ConfigApiController.addConfig()/updateConfig()
2. Build ConfigKey from path parameters
3. ConfigService.addConfig()/updateConfig()
   ├── ModelRepoService.getSchema() → Get schema (if validate)
   ├── SecretParser → Parse and separate secrets
   │   ├── Extract secrets → SecretRepository.saveSecret()
   │   ├── Extract secret files → SecretRepository.saveSecretFile()
   │   └── Extract config → ConfigRepository.saveConfig()
   └── Store in Consul (config) and Vault (secrets)
```

### 6.3 Secret Retrieval Flow

```
1. Request → ConfigSecretsController.getSecret()
2. Validate authorization → IamService
3. SecretFetcherService.fetch()
   ├── Parse secret path
   ├── SecretRepository.getSecret() → Vault
   └── Return secret value(s)
```

---

## 7. Key Technologies & Dependencies

### 7.1 Core Technologies
- **Java 17**: Programming language
- **Spring Boot 3.3.11**: Application framework
- **Spring Cloud 2023.0.4**: Cloud-native features
- **Consul API**: Configuration storage client
- **Spring Vault 2.3.3**: HashiCorp Vault integration
- **Jackson**: JSON processing
- **Protobuf 3.25.5**: Schema definitions

### 7.2 Key Dependencies
- **Spring Boot Starter Web**: REST API support
- **Spring Boot Starter Actuator**: Health checks, metrics
- **Micrometer Prometheus**: Metrics export
- **Resilience4j**: Circuit breaker pattern
- **SpringDoc OpenAPI**: API documentation
- **IAM Utils**: IAM integration utilities
- **JSON Patch**: JSON patch operations

### 7.3 Infrastructure
- **Consul**: Configuration storage backend
- **HashiCorp Vault**: Secrets storage backend
- **Docker**: Containerization
- **Kubernetes**: Orchestration
- **Helm**: Deployment charts

---

## 8. Security & Authorization

### 8.1 Authentication
- **IAM Integration**: IAM service for authentication
- **Token-based**: JWT token validation
- **Service Account**: Service account authentication

### 8.2 Authorization
- **Path-based**: Authorization based on config/secret paths
- **IAM Permissions**: IAM-based permission checks
- **Tenant Isolation**: Tenant-scoped access control

### 8.3 Secrets Security
- **Vault Encryption**: Secrets encrypted at rest in Vault
- **Access Control**: Path-based access control
- **Audit Trail**: Vault audit logging

---

## 9. Configuration Key Structure

### 9.1 ConfigKey Components
```java
ConfigKey {
    tenantId: String          // Tenant identifier
    componentId: String       // Component identifier
    configId: String          // Configuration identifier
    type: ConfigType          // DEPLOYMENT or RUNTIME
    env: String              // Environment (dev, staging, prod)
    appInstanceId: String    // Application instance ID
    appStage: String         // Application stage
    orgId: String            // Organization ID
}
```

### 9.2 Path Generation
```
tenant/{tenantId}/conf/{configId}/env/{env}/app/{appId}/org/{orgId}/type/{type}
```

**Example Paths**:
- Tenant-level: `tenant/tenant-123/conf/comp-456/type/DEPLOYMENT`
- Environment-level: `tenant/tenant-123/conf/comp-456/env/prod/type/DEPLOYMENT`
- App-level: `tenant/tenant-123/conf/comp-456/env/prod/app/app-789/type/DEPLOYMENT`
- Org-level: `tenant/tenant-123/conf/comp-456/env/prod/app/app-789/org/org-101/type/DEPLOYMENT`

---

## 10. Configuration Processing Utilities

### 10.1 SecretParser
- **Purpose**: Extract secrets from configuration JSON
- **Functionality**: 
  - Identifies secret fields from schema
  - Separates secrets from regular config
  - Handles secret files

### 10.2 ConfigMerger
- **Purpose**: Merge configurations from multiple sources
- **Functionality**:
  - Merge parent and child configurations
  - Handle config references
  - Property format merging

### 10.3 ConfigPropertyFormatter
- **Purpose**: Convert JSON configuration to properties format
- **Functionality**:
  - Flatten nested JSON structure
  - Apply property prefixes
  - Handle arrays and nested objects

### 10.4 ConfigReferenceExtractor
- **Purpose**: Extract and resolve configuration references
- **Functionality**:
  - Find config references in schema
  - Resolve referenced configurations
  - Merge referenced configs

---

## 11. Integration Points

### 11.1 External Services
- **IAM Service**: Authentication and authorization
- **Model Repository**: Schema retrieval
- **Component Library Service**: Component metadata
- **Organization Service**: Organization management
- **Consul**: Configuration storage
- **HashiCorp Vault**: Secrets storage

### 11.2 Integration Patterns
- **REST APIs**: HTTP-based integration
- **Circuit Breaker**: Resilience4j for fault tolerance
- **Retry Logic**: Spring Retry for transient failures
- **Caching**: Platform service principal caching

---

## 12. Deployment & Infrastructure

### 12.1 Containerization
- **Docker**: Multi-stage Docker builds
- **Base Image**: Standard base images
- **Health Checks**: Actuator health endpoints

### 12.2 Kubernetes Deployment
- **Helm Charts**: Helm-based deployment
- **ConfigMaps**: Configuration management
- **Secrets**: CSI secret provider for Vault
- **Service Account**: RBAC configuration
- **HPA**: Horizontal pod autoscaling
- **PDB**: Pod disruption budgets
- **Ingress**: External access configuration

### 12.3 Monitoring
- **Actuator**: Health and metrics endpoints
- **Prometheus**: Metrics export
- **Health Checks**: Liveness and readiness probes

---

## 13. Use Cases & Scenarios

### 13.1 Application Configuration
- **Scenario**: Store application-specific configurations
- **Example**: Database connection strings, API endpoints, feature flags
- **Path**: `tenant/{tenantId}/conf/{componentId}/env/{env}/type/DEPLOYMENT`

### 13.2 Secrets Management
- **Scenario**: Store sensitive credentials
- **Example**: API keys, passwords, certificates
- **Storage**: HashiCorp Vault
- **Access**: Path-based authorization

### 13.3 Multi-Environment Configuration
- **Scenario**: Different configs for dev, staging, prod
- **Example**: Environment-specific database URLs
- **Path**: `tenant/{tenantId}/conf/{componentId}/env/{env}/type/DEPLOYMENT`

### 13.4 Organization-Specific Overrides
- **Scenario**: Org-level configuration overrides
- **Example**: Custom branding, org-specific settings
- **Path**: `tenant/{tenantId}/conf/{componentId}/env/{env}/app/{appId}/org/{orgId}/type/DEPLOYMENT`

### 13.5 Preference Management
- **Scenario**: User and organization preferences
- **Example**: UI preferences, notification settings
- **API**: PreferenceController endpoints

---

## 14. Key Design Patterns

### 14.1 Repository Pattern
- **ConfigRepository**: Abstraction for config storage
- **SecretRepository**: Abstraction for secret storage
- **Implementation**: Consul and Vault implementations

### 14.2 Circuit Breaker Pattern
- **Resilience4j**: Circuit breaker for Consul operations
- **Fallback Methods**: Graceful degradation
- **Fault Tolerance**: Handle Consul failures

### 14.3 Strategy Pattern
- **Config Formatting**: Different formatters (JSON, Properties)
- **Config Merging**: Different merge strategies
- **Secret Parsing**: Different parsing strategies

### 14.4 Interceptor Pattern
- **AuthInterceptor**: Request authentication
- **CORS Configuration**: Cross-origin handling

---

## 15. Configuration Schema Validation

### 15.1 Schema Source
- **Model Repository**: Protobuf-based schemas
- **Schema Format**: JSON Schema (Attributes.Attribute)
- **Schema Retrieval**: On-demand schema fetching

### 15.2 Validation Process
1. Retrieve schema from Model Repository
2. Parse configuration JSON
3. Validate against schema
4. Report validation errors

### 15.3 Validation Features
- **Type Checking**: Data type validation
- **Required Fields**: Required field validation
- **Format Validation**: String format validation
- **Nested Validation**: Recursive schema validation

---

## 16. Performance & Scalability

### 16.1 Caching
- **Platform Service Principal Cache**: Cache IAM tokens
- **Schema Caching**: Cache configuration schemas
- **Component Metadata Cache**: Cache component information

### 16.2 Circuit Breaker
- **Consul Operations**: Circuit breaker for Consul calls
- **Fallback Handling**: Graceful degradation
- **Resilience**: Fault tolerance

### 16.3 Scalability
- **Horizontal Scaling**: Kubernetes HPA
- **Stateless Design**: Stateless service design
- **External Storage**: Consul and Vault for persistence

---

## 17. Error Handling

### 17.1 Exception Types
- **ConfigNotFoundException**: Configuration not found
- **ConfigAlreadyExistsException**: Configuration already exists
- **InvalidConfigJson**: Invalid JSON format
- **InvalidConfigPathException**: Invalid path format
- **SecretNotFoundException**: Secret not found
- **UnauthorizedException**: Authorization failure
- **DMLException**: Data manipulation errors

### 17.2 Error Response Format
```json
{
  "error": "Error message",
  "code": "ERROR_CODE",
  "details": {}
}
```

---

## 18. Testing

### 18.1 Test Structure
- **Unit Tests**: Service and repository tests
- **Integration Tests**: End-to-end API tests
- **Mock Services**: Mocked external dependencies

### 18.2 Test Coverage
- **JaCoCo**: Code coverage reporting
- **Test Files**: Comprehensive test suite
- **Mockito**: Mocking framework

---

## 19. API Documentation

### 19.1 OpenAPI Specification
- **Format**: OpenAPI 3.0
- **Tool**: SpringDoc OpenAPI
- **UI**: Swagger UI available
- **Location**: `/config-management/swagger-ui.html`

### 19.2 API Groups
- **Configuration Operations**: Config CRUD
- **Secret Operations**: Secret management
- **Preference Operations**: Preference management
- **Connector Interface**: Interface operations

---

## 20. Summary

### 20.1 Key Strengths
- **Centralized Management**: Single source of truth for configurations
- **Security**: Secure secrets management via Vault
- **Flexibility**: Hierarchical configuration support
- **Validation**: Schema-based validation
- **Multi-format**: JSON and properties format support
- **Multi-tenant**: Tenant isolation and scoping

### 20.2 Use Cases
- **Application Configuration**: Store app configs
- **Secrets Management**: Secure credential storage
- **Environment Management**: Environment-specific configs
- **Organization Overrides**: Org-level customization
- **Preference Management**: User and org preferences

### 20.3 Technology Highlights
- **Consul**: Distributed configuration storage
- **Vault**: Enterprise secrets management
- **Spring Boot**: Modern Java framework
- **Circuit Breaker**: Fault tolerance
- **Kubernetes**: Cloud-native deployment

---

This comprehensive explanation covers all aspects of the Config Management Service, from architecture and design to implementation details and deployment strategies.

