# Org-Service - Detailed Project Explanation

## Executive Summary

**Org-Service (OMS - Organization Management Service)** is a multi-tenant, enterprise-grade organization and resource management platform built with Spring Boot 3.3.1 and Java 17. It provides comprehensive organization hierarchy management, fine-grained authorization (FGA) using SpiceDB/AuthZed, resource access control, user-organization mapping, and preference management for complex enterprise organizational structures.

---

## 1. Project Overview

### 1.1 Core Purpose
- **Multi-tenant organization management** for hierarchical organizational structures
- **Fine-grained authorization (FGA)** using SpiceDB/AuthZed for relationship-based permissions
- **Resource access control** with resource types and permission management
- **User-organization mapping** with role-based access within organizational context
- **Preference management** for organizational and user-level preferences
- **Bulk operations** for organization import/export via CSV
- **Temporal workflows** for asynchronous permission synchronization

### 1.2 Technical Specifications
- **Language**: Java 17
- **Framework**: Spring Boot 3.3.1
- **Build Tool**: Maven
- **Database**: PostgreSQL with JPA/Hibernate, Liquibase for migrations
- **Authorization**: SpiceDB/AuthZed (gRPC) for fine-grained authorization
- **Caching**: Redis for performance optimization
- **Workflow Engine**: Temporal SDK 1.30.0 for async operations
- **API Documentation**: OpenAPI 3.0 (SpringDoc)
- **Port**: 8080

---

## 2. Architecture & Design

### 2.1 Application Architecture

```
OmsApplication (Spring Boot Entry Point)
├── Controllers (REST API Layer)
│   ├── OrgManagementController (V1)
│   ├── OrgManagementV2Controller (V2)
│   ├── UserController / UserV2Controller
│   ├── ResourceV2Controller
│   ├── OrgTypeController / OrgTypeV2Controller
│   ├── RoleController
│   ├── PreferenceController
│   ├── SchemaController
│   └── FineGrainedAuthorizationController
├── Services (Business Logic Layer)
│   ├── OrgService / OrgServiceImpl / OrgServiceV2Impl
│   ├── UserService / UserServiceImpl / UserServiceV2Impl
│   ├── ResourceService / ResourceServiceImpl
│   ├── TenantService / TenantServiceImpl / TenantServiceV2Impl
│   ├── AuthZedService / AuthZedServiceImpl
│   ├── FGAService / FGAServiceImpl
│   ├── RoleService / RoleServiceImpl
│   ├── PreferenceService / PreferenceServiceImpl
│   └── BulkProcessorService
├── Repositories (Data Access Layer)
│   ├── OrgRepository
│   ├── UserRepository
│   ├── ResourceRepository
│   ├── TenantRepository
│   ├── RoleRepository
│   └── ...
├── Models (JPA Entities)
│   ├── OrgNode (hierarchical organization structure)
│   ├── User (organization users)
│   ├── Resource (resource access control)
│   ├── Tenant (multi-tenant isolation)
│   ├── Role (organization roles)
│   ├── OrgType (organization types)
│   ├── ResourceType (resource type definitions)
│   └── Preference / PreferenceGroup
└── Workflows (Temporal Integration)
    ├── SyncResourcePermissionsWorkflow
    ├── SyncUserPermissionsWorkflow
    ├── SyncUserPermissionCacheWorkflow
    └── ProvidePermissionWorkflow
```

### 2.2 Key Components

#### **Controllers (15+ REST Controllers)**

**V1 Controllers:**
- **OrgManagementController**: Organization CRUD, root org management, CSV import/export
- **UserController**: User management within organizations
- **OrgTypeController**: Organization type management
- **RoleController**: Role management
- **PreferenceController**: Preference group and preference management
- **ResourceTypeController**: Resource type definitions
- **SchemaController**: SpiceDB schema management

**V2 Controllers (Enhanced):**
- **OrgManagementV2Controller**: Enhanced org management with search, filtering, composite operations
- **UserV2Controller**: Enhanced user management
- **ResourceV2Controller**: Resource access management, bulk operations
- **OrgTypeV2Controller**: Enhanced org type management with UI hierarchy
- **FineGrainedAuthorizationController**: FGA refresh and permission management
- **BulkResourceAccessImportController**: Bulk resource access import

#### **Services (14+ Service Implementations)**

- **OrgService**: Core organization lifecycle management, hierarchy operations
- **UserService**: User management, user-organization mapping
- **ResourceService**: Resource access control, permission provisioning
- **TenantService**: Multi-tenant management, root organization setup
- **AuthZedService**: SpiceDB/AuthZed integration for FGA
- **FGAService**: Fine-grained authorization operations
- **RoleService**: Role management within organizations
- **PreferenceService**: Preference and preference group management
- **OrgTypeService**: Organization type definitions
- **ResourceTypeService**: Resource type management
- **BulkProcessorService**: CSV bulk import/export processing

#### **Models (9 JPA Entities)**

Core entities:
- **OrgNode**: Hierarchical organization structure with parent-child relationships
- **User**: Organization users with IAM user mapping
- **Resource**: Resources with resource types for access control
- **Tenant**: Multi-tenant isolation, root organization reference
- **Role**: Organization-specific roles
- **OrgType**: Organization type definitions (e.g., department, division, team)
- **ResourceType**: Resource type definitions (e.g., account, document, report)
- **Preference**: User/organization preferences
- **PreferenceGroup**: Grouped preferences

---

## 3. Core Features & Capabilities

### 3.1 Organization Management

#### **Hierarchical Organization Structure**
- **Tree Structure**: Parent-child relationships for organizational hierarchy
- **Root Organization**: Each tenant has a root organization
- **Multi-level Hierarchy**: Support for unlimited depth organizational trees
- **Organization Types**: Typed organizations (department, division, team, etc.)
- **External IDs**: Support for external system integration via external IDs
- **Extra Fields**: JSON-based extensible fields for custom attributes

#### **Organization Operations**
- **Create Organization**: Create new organizations with parent-child relationships
- **Update Organization**: Update organization name, external ID, type
- **Delete Organization**: Delete organizations (cascade to children)
- **Search Organizations**: Advanced search with filtering, pagination
- **Get Organization Tree**: Retrieve full organizational hierarchy
- **Bulk Import/Export**: CSV-based bulk organization import/export

#### **Organization-User Mapping**
- **Many-to-Many**: Users can belong to multiple organizations
- **User Assignment**: Assign users to organizations
- **User Removal**: Remove users from organizations
- **Organization Users**: Get all users in an organization

### 3.2 Fine-Grained Authorization (FGA)

#### **SpiceDB/AuthZed Integration**
- **Relationship-Based Permissions**: Define permissions based on relationships
- **Schema Management**: Upload and manage SpiceDB authorization schemas
- **Permission Checks**: Check user permissions on resources
- **Relationship Management**: Create, update, delete relationships
- **Lookup Operations**: Lookup resources and subjects based on permissions

#### **Authorization Features**
- **Resource Access Control**: Control access to resources based on organizational context
- **Hierarchical Permissions**: Permissions inherited through organizational hierarchy
- **Permission Refresh**: Refresh permissions for users/resources
- **Permission Synchronization**: Async permission sync via Temporal workflows
- **UI Access Control**: UI-level access control based on permissions

### 3.3 Resource Management

#### **Resource Access Control**
- **Resource Types**: Define resource types (account, document, report, etc.)
- **Resource Creation**: Create resources with types
- **User-Resource Mapping**: Map users to resources with permissions
- **Bulk Resource Access**: Bulk assign/remove resource access
- **Resource Search**: Search resources by type, user, organization
- **Access Queries**: Query user access to resources

#### **Resource Operations**
- **Add Resource**: Create new resources
- **Grant Access**: Grant user access to resources
- **Remove Access**: Remove user access from resources
- **Check Access**: Check if user has access to resource
- **List Resources**: List resources with filtering
- **Bulk Import**: CSV-based bulk resource access import

### 3.4 User Management

#### **User Operations**
- **Create User**: Create users within tenant/organization context
- **Update User**: Update user details, organization assignments
- **Delete User**: Delete users from organizations
- **Get User**: Get user details with organization associations
- **List Users**: List users with filtering, pagination
- **User-Organization Mapping**: Assign users to multiple organizations

#### **User Features**
- **IAM Integration**: Integration with Apex IAM for user identity
- **Role Assignment**: Assign roles to users within organizations
- **Organization Context**: Users belong to organizations with roles
- **Multi-Organization**: Users can belong to multiple organizations

### 3.5 Preference Management

#### **Preference Operations**
- **Preference Groups**: Group related preferences
- **Create Preferences**: Create user/organization preferences
- **Update Preferences**: Update preference values
- **Delete Preferences**: Remove preferences
- **Get Preferences**: Retrieve preferences with merging
- **Typed Preferences**: Type-safe preference management

#### **Preference Features**
- **User Preferences**: User-level preferences
- **Organization Preferences**: Organization-level preferences
- **Preference Merging**: Merge preferences from multiple sources
- **Preference Groups**: Organize preferences into groups

### 3.6 Bulk Operations

#### **CSV Import/Export**
- **Organization Import**: Import organizational structure from CSV
- **Organization Export**: Export organizational structure to CSV
- **Composite Import**: Import organizations with users, resources
- **Resource Access Import**: Bulk import resource access mappings
- **Validation**: CSV validation with error reporting
- **Jiffy Drive Integration**: Import/export from Jiffy Drive

### 3.7 Temporal Workflows

#### **Async Permission Synchronization**
- **SyncResourcePermissionsWorkflow**: Sync resource permissions to SpiceDB
- **SyncUserPermissionsWorkflow**: Sync user permissions
- **SyncUserPermissionCacheWorkflow**: Sync permission cache
- **ProvidePermissionWorkflow**: Provision permissions to resources
- **Workflow Semaphore**: Control concurrent workflow execution

#### **Workflow Activities**
- **ProvisionPermissionActivity**: Provision permissions to SpiceDB
- **RemovePermissionActivity**: Remove permissions from SpiceDB
- **RecalculateAuthzPermissionsActivity**: Recalculate authorization permissions
- **RecalculateUserPermissionsActivity**: Recalculate user permissions
- **SyncUserPermissionCacheActivity**: Sync permission cache

---

## 4. Database Schema

### 4.1 Core Tables

**org_node**
- Hierarchical organization structure
- Fields: id, name, display_name, external_id, parent_id, tenant_id, org_type_id, extra_fields (JSON)
- Relationships: parent (self-reference), children (one-to-many), tenant (many-to-one), org_type (many-to-one)

**org_user**
- Organization users
- Fields: id, iam_user_id, name, display_name, email_id, tenant_id, role_id
- Relationships: tenant (many-to-one), org_nodes (many-to-many), role (many-to-one)

**resource**
- Resources for access control
- Fields: id, resource_id, resource_name, tenant_id, resource_type_id
- Unique constraint: (tenant_id, resource_type_id, resource_id)

**tenant**
- Multi-tenant isolation
- Fields: id, tenant_name, root_org_id
- Relationships: root_org (one-to-one)

**org_type**
- Organization type definitions
- Fields: id, name, tenant_id

**resource_type**
- Resource type definitions
- Fields: id, resource_type, tenant_id

**role**
- Organization roles
- Fields: id, role_name, tenant_id

**preference_group**
- Preference groups
- Fields: id, group_name, tenant_id, app_id

**preference**
- Preferences
- Fields: id, preference_key, preference_value, preference_group_id

**org_user_mapping**
- Many-to-many mapping between organizations and users
- Fields: org_node_id, user_id

---

## 5. API Endpoints

### 5.1 Organization Management APIs (V2)

**Organization CRUD:**
- `GET /oms/api/v2/org` - Get organizations with filtering, pagination
- `POST /oms/api/v2/org` - Create organization
- `PUT /oms/api/v2/org/{orgId}` - Update organization
- `DELETE /oms/api/v2/org/{orgId}` - Delete organization
- `GET /oms/api/v2/org/{orgId}` - Get organization details

**Organization Search:**
- `GET /oms/api/v2/org/search` - Advanced organization search
- `GET /oms/api/v2/org/internal/search` - Internal organization search

**Bulk Operations:**
- `POST /oms/api/v2/org/import` - Import organizations from CSV
- `POST /oms/api/v2/org/import/composite` - Composite import (orgs + users + resources)
- `POST /oms/api/v2/org/import/composite/jiffydrive` - Import from Jiffy Drive
- `POST /oms/api/v2/org/export` - Export organizations to CSV

### 5.2 User Management APIs (V2)

- `GET /oms/api/v2/user` - Get users with filtering
- `GET /oms/api/v2/user/{userId}` - Get user details
- `POST /oms/api/v2/user` - Create user
- `PUT /oms/api/v2/user/{userId}` - Update user
- `DELETE /oms/api/v2/user/{userId}` - Delete user

### 5.3 Resource Management APIs (V2)

**Resource Access:**
- `POST /oms/api/v2/resource/access` - Grant resource access to user
- `POST /oms/api/v2/resource/remove-access` - Remove resource access
- `GET /oms/api/v2/resource/{resourceType}/{resourceId}/access/` - Get resource access list
- `GET /oms/api/v2/resource/access/{userId}` - Get user resource access
- `GET /oms/api/v2/resource/access` - Get all resource access with filtering
- `GET /oms/api/v2/resource/{resourceType}/{resourceId}/access/{userId}` - Check specific access
- `DELETE /oms/api/v2/resource/access/{userId}` - Remove all user resource access
- `DELETE /oms/api/v2/resource/{resourceType}/{resourceId}/access/{userId}` - Remove specific access
- `GET /oms/api/v2/resource/ui-access` - Get UI access permissions

**Bulk Operations:**
- `POST /oms/api/v2/resource/import` - Bulk import resource access

### 5.4 Fine-Grained Authorization APIs

- `POST /oms/api/v2/finegrainedauthorization/{resourceId}/refresh` - Refresh FGA permissions

### 5.5 Schema Management APIs

- `GET /oms/api/v1/internal/schema` - Read SpiceDB schema
- `POST /oms/api/v1/internal/schema` - Write/update SpiceDB schema

### 5.6 Preference Management APIs

- `POST /oms/api/v1/preference` - Create preference group
- `GET /oms/api/v1/preference/{groupId}` - Get preference group
- `PUT /oms/api/v1/preference/{groupId}` - Update preference group
- `GET /oms/api/v1/preference/by-name/{groupName}` - Get preference group by name
- `PUT /oms/api/v1/preference/by-name/{groupName}` - Update preference group by name
- `DELETE /oms/api/v1/preference/{groupId}` - Delete preference group
- `GET /oms/api/v1/preference` - List preference groups
- `GET /oms/api/v1/preference/merged` - Get merged preferences
- `GET /oms/api/v1/preference/typed` - Get typed preferences
- `POST /oms/api/v1/preference/typed` - Create typed preference group

---

## 6. Technology Stack & Dependencies

### 6.1 Core Technologies
- **Java 17**: Programming language
- **Spring Boot 3.3.1**: Application framework
- **Spring Data JPA**: Data access layer
- **PostgreSQL**: Primary database
- **Liquibase**: Database migration tool
- **Redis**: Caching and event logging
- **Temporal SDK 1.30.0**: Workflow orchestration
- **AuthZed SDK 1.3.0**: SpiceDB/AuthZed gRPC client
- **SpringDoc OpenAPI**: API documentation

### 6.2 Key Dependencies
- **IAM Utils**: Integration with Apex IAM for authentication
- **Commons CSV**: CSV parsing for bulk operations
- **Jackson**: JSON/YAML processing
- **Lombok**: Code generation
- **Logstash Logback Encoder**: Structured logging
- **Micrometer Prometheus**: Metrics collection

---

## 7. Key Design Patterns & Features

### 7.1 Multi-Tenant Architecture
- **Tenant Isolation**: All data scoped by tenant_id
- **Root Organization**: Each tenant has a root organization
- **Tenant-Scoped Queries**: All queries filtered by tenant
- **Cross-Tenant Prevention**: Data isolation enforced at service layer

### 7.2 Hierarchical Organization Model
- **Tree Structure**: Parent-child relationships using self-referencing foreign keys
- **Cascade Operations**: Delete cascades to children
- **Path Queries**: Efficient hierarchical queries
- **Organization Types**: Typed organizations for different business contexts

### 7.3 Fine-Grained Authorization (FGA)
- **SpiceDB Integration**: Relationship-based authorization using SpiceDB
- **Schema-Driven**: Authorization rules defined in SpiceDB schema (Zed language)
- **Permission Checks**: Real-time permission checking via gRPC
- **Relationship Management**: Create/update/delete authorization relationships
- **Permission Refresh**: Async permission refresh via workflows

### 7.4 Resource Access Control
- **Resource Types**: Typed resources for different access patterns
- **User-Resource Mapping**: Many-to-many mapping with permissions
- **Bulk Operations**: Efficient bulk resource access management
- **Access Queries**: Query resources accessible to users

### 7.5 Temporal Workflows
- **Async Processing**: Long-running permission sync operations
- **Fault Tolerance**: Automatic retry and error handling
- **Workflow Semaphore**: Control concurrent workflow execution
- **Activity Pattern**: Separate activities for different operations

### 7.6 Bulk Operations
- **CSV Processing**: CSV-based bulk import/export
- **Validation**: Comprehensive CSV validation
- **Error Reporting**: Detailed error reporting for failed imports
- **Composite Operations**: Import organizations with users and resources

---

## 8. Integration Points

### 8.1 External Services
- **Apex IAM**: User identity and authentication
- **SpiceDB/AuthZed**: Fine-grained authorization service
- **Jiffy Drive**: File storage for CSV import/export
- **Temporal Server**: Workflow orchestration
- **Redis**: Caching and event logging

### 8.2 Client Integrations
- **IAMClient**: IAM service integration for user management
- **JiffyDriveClient**: File storage integration
- **AppDataManagerClient**: Application data management

---

## 9. Data Flow Examples

### 9.1 Organization Creation Flow
```
1. Client Request → OrgManagementV2Controller.create()
2. → OrgServiceV2Impl.createOrg()
3. → Validate parent organization exists
4. → Create OrgNode entity
5. → Save to PostgreSQL
6. → Provision permissions to SpiceDB (via AuthZedService)
7. → Trigger SyncResourcePermissionsWorkflow (async)
8. → Return OrgDTO
```

### 9.2 Resource Access Grant Flow
```
1. Client Request → ResourceV2Controller.grantAccess()
2. → ResourceServiceImpl.addResourcesToUser()
3. → Create/Get Resource entity
4. → Add relationship to SpiceDB (via AuthZedService)
5. → Trigger SyncResourcePermissionsWorkflow (async)
6. → Return ResourceResponseDTO
```

### 9.3 Permission Check Flow
```
1. Client Request → ResourceV2Controller.checkAccess()
2. → AuthZedServiceImpl.checkPermission()
3. → Build CheckPermissionRequest
4. → gRPC call to SpiceDB
5. → Return permission result
```

---

## 10. Security & Authorization

### 10.1 Authentication
- **IAM Integration**: Authentication via Apex IAM
- **Token Validation**: JWT token validation
- **Tenant Context**: Tenant ID from request headers
- **User Context**: User ID from request headers

### 10.2 Authorization
- **SpiceDB FGA**: Fine-grained authorization via SpiceDB
- **Relationship-Based**: Permissions based on organizational relationships
- **Hierarchical Permissions**: Permissions inherited through org hierarchy
- **Resource-Level**: Resource-specific access control

### 10.3 Data Isolation
- **Tenant Isolation**: All queries scoped by tenant
- **Organization Context**: Operations within organizational context
- **Cross-Tenant Prevention**: Enforced at service layer

---

## 11. Performance & Scalability

### 11.1 Caching
- **Redis Caching**: Cache frequently accessed data
- **Permission Cache**: Cache permission check results
- **Organization Cache**: Cache organization hierarchy

### 11.2 Database Optimization
- **Indexes**: Indexes on tenant_id, parent_id, resource_id
- **Lazy Loading**: Lazy loading for relationships
- **Batch Operations**: Batch inserts/updates for bulk operations

### 11.3 Async Processing
- **Temporal Workflows**: Async permission synchronization
- **Workflow Semaphore**: Control concurrent workflows
- **Retry Logic**: Automatic retry for failed operations

---

## 12. Deployment & Infrastructure

### 12.1 Containerization
- **Docker**: Containerized application
- **Multi-stage Build**: Optimized Docker images
- **Health Checks**: Application health monitoring

### 12.2 Kubernetes
- **Helm Charts**: Kubernetes deployment via Helm
- **HPA**: Horizontal Pod Autoscaler for scaling
- **Service Mesh**: Istio integration for service mesh
- **ConfigMaps**: Configuration management
- **Secrets**: Secret management via CSI

### 12.3 Monitoring
- **Prometheus**: Metrics collection
- **Actuator**: Health and metrics endpoints
- **Logging**: Structured logging with Logstash encoder

---

## 13. Use Cases & Scenarios

### 13.1 Enterprise Organization Management
- **Hierarchical Structures**: Manage complex organizational hierarchies
- **Multi-level Organizations**: Support for departments, divisions, teams
- **User Assignment**: Assign users to organizations with roles
- **Bulk Operations**: Import/export organizational structures

### 13.2 Resource Access Control
- **Resource Types**: Define different resource types (accounts, documents, reports)
- **User-Resource Mapping**: Grant users access to specific resources
- **Permission Management**: Manage fine-grained permissions
- **Access Queries**: Query what resources users can access

### 13.3 Fine-Grained Authorization
- **Relationship-Based**: Permissions based on organizational relationships
- **Hierarchical Permissions**: Permissions inherited through hierarchy
- **Real-Time Checks**: Real-time permission checking
- **Permission Refresh**: Refresh permissions as organizational structure changes

### 13.4 Preference Management
- **User Preferences**: User-level preferences
- **Organization Preferences**: Organization-level preferences
- **Preference Groups**: Organize preferences into groups
- **Preference Merging**: Merge preferences from multiple sources

---

## 14. Key Differentiators

### 14.1 Fine-Grained Authorization
- **SpiceDB Integration**: Advanced relationship-based authorization
- **Schema-Driven**: Flexible authorization rules via SpiceDB schema
- **Real-Time Checks**: Sub-millisecond permission checks
- **Hierarchical Permissions**: Permissions inherited through org hierarchy

### 14.2 Hierarchical Organization Model
- **Tree Structure**: Efficient parent-child relationships
- **Unlimited Depth**: Support for any depth of organizational hierarchy
- **Type System**: Typed organizations for different contexts
- **Bulk Operations**: Efficient bulk import/export

### 14.3 Temporal Workflows
- **Async Processing**: Long-running permission sync operations
- **Fault Tolerance**: Automatic retry and error handling
- **Workflow Semaphore**: Control concurrent execution
- **Scalable**: Handle thousands of concurrent workflows

---

## 15. Summary

**Org-Service** is a comprehensive organization and resource management platform that provides:

1. **Hierarchical Organization Management**: Tree-based organizational structures with unlimited depth
2. **Fine-Grained Authorization**: Relationship-based permissions via SpiceDB/AuthZed
3. **Resource Access Control**: Typed resources with user-resource mapping
4. **Multi-Tenant Architecture**: Complete tenant isolation and data segregation
5. **Bulk Operations**: CSV-based import/export for organizations, users, resources
6. **Temporal Workflows**: Async permission synchronization with fault tolerance
7. **Preference Management**: User and organization-level preferences
8. **RESTful APIs**: Comprehensive REST APIs (V1 and V2) with OpenAPI documentation

The system is designed for enterprise-scale organizational management with advanced authorization capabilities, making it suitable for complex multi-tenant SaaS applications requiring fine-grained access control based on organizational relationships.

