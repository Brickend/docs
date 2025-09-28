# Brickend Framework Principles

> **Version**: 1.0  
> **Status**: Draft  
> **Target**: All language implementations

## Table of Contents

- [Core Philosophy](#core-philosophy)
- [Universal Principles](#universal-principles)
- [Configuration Contract](#configuration-contract)
- [Type System](#type-system)
- [Architecture Contract](#architecture-contract)
- [API Contract](#api-contract)
- [Implementation Requirements](#implementation-requirements)
- [Compliance Validation](#compliance-validation)
- [Ecosystem Standards](#ecosystem-standards)

---

## Core Philosophy

### The Brickend Promise

**Configuration over Code**: Developers describe what they want, not how to build it.

**Language Agnostic**: The same configuration produces functionally identical backends across any supported language.

**Copy-Paste Ownership**: Generated code belongs to the developer. No runtime dependencies. No vendor lock-in.

**Progressive Complexity**: Start minimal, scale up. The framework grows with project needs.

### Anti-Patterns

Brickend explicitly rejects:
- Magic runtime behavior that cannot be inspected
- Proprietary configuration formats or vendor-specific extensions
- Language-specific features that break cross-language compatibility
- Complex abstractions that obscure the generated code

---

## Universal Principles

### Principle 1: Configuration Determinism

**Rule**: Identical configuration must produce functionally equivalent outputs across all language implementations.

```yaml
# This configuration MUST produce the same API behavior 
# whether generated for Python, Go, TypeScript, Rust, etc.
project:
  name: "user-api"

models:
  User:
    fields:
      id: "uuid, primary_key"
      email: "string, unique, required"

endpoints:
  - name: "users"
    model: "User"
    type: "crud"
```

**Validation**: All implementations must pass identical integration test suites.

### Principle 2: Minimal Viable Abstraction

**Rule**: Only abstract concepts that exist universally across target languages.

**Supported Universal Concepts**:
- HTTP methods: GET, POST, PUT, DELETE, PATCH
- Status codes: 200, 201, 400, 401, 403, 404, 409, 422, 500
- Data types: string, integer, float, boolean, uuid, timestamp, date, json
- Constraints: required, unique, indexed, primary_key, auto_increment
- Relations: one_to_one, one_to_many, many_to_one, many_to_many
- Authentication: jwt, api_key, bearer_token
- CRUD operations: create, read, update, delete, list

**Explicitly NOT Supported** (language-specific):
- Language-specific libraries or frameworks beyond the target
- Runtime-specific features (async/await patterns, goroutines, etc.)
- Platform-specific deployment configurations
- Language-specific data types without universal equivalents

### Principle 3: Error Surface Minimization

**Rule**: Fail fast and provide clear error messages for unsupported configurations.

```yaml
# Implementation MUST reject this with clear error message
database:
  type: "mongodb"  # If MongoDB not supported by target language

# Error format requirement:
# "MongoDB not supported in brickend-go v1.2. Supported databases: postgresql, mysql, sqlite"
```

### Principle 4: Idempotent Generation

**Rule**: Re-running generation with the same configuration produces identical outputs.

**Requirements**:
- No timestamps in generated files unless explicitly configured
- Deterministic file ordering and content structure
- Consistent formatting and naming conventions
- Reproducible builds across different environments

### Principle 5: Non-Destructive Updates

**Rule**: Framework updates preserve user modifications to business logic.

**Architecture Requirement**: Clear separation between:
- **Framework files**: Regenerated on every `brickend generate`
- **Business logic files**: Generated once, safe for user modification
- **Configuration files**: User-owned, never overwritten

---

## Configuration Contract

### Configuration Structure

All Brickend projects MUST use the `brickend/` folder for configuration with progressive complexity:

```
project-root/
├── brickend/
│   ├── brickend.yaml       # Main configuration (ALWAYS REQUIRED)
│   ├── security.yaml       # Security & auth (optional)
│   ├── api.yaml            # API standards (optional)
│   ├── environments/       # Environment configs (optional)
│   │   ├── development.yaml
│   │   ├── staging.yaml
│   │   └── production.yaml
│   └── services/           # Service modules (optional)
│       ├── users.yaml
│       ├── organizations.yaml
│       └── [service].yaml
└── [generated files...]
```

**Progressive Complexity Philosophy:**
- **Simple projects**: Only `brickend/brickend.yaml` (everything inline)
- **Medium projects**: `brickend.yaml` + optional `security.yaml` or `api.yaml`
- **Complex projects**: Full modular structure with services and environments

### Schema Structure Options

#### Option 1: Standalone Configuration (Simple Projects)

**Single File** (`brickend/brickend.yaml`):

```yaml
# Complete standalone configuration - everything in one file
project:
  name: "simple-api"              # REQUIRED: Project identifier
  version: "1.0.0"                # OPTIONAL: Semantic version
  description: "A simple API"     # OPTIONAL: Project description

# Database configuration - REQUIRED
database:
  type: postgresql                # REQUIRED: postgresql|mysql|sqlite|supabase
  host: localhost                 # OPTIONAL: Default localhost  
  port: 5432                      # OPTIONAL: Default per DB type
  name: simple_api_db             # REQUIRED: Database name

# Authentication - OPTIONAL (inline)
auth:
  enabled: true                   # OPTIONAL: Enable authentication
  type: jwt                       # OPTIONAL: jwt|api_key|bearer_token
  secret_key: "${JWT_SECRET}"     # REQUIRED if auth.enabled

# API configuration - OPTIONAL (inline)
api:
  host: "0.0.0.0"                # OPTIONAL: Default 127.0.0.1
  port: 8000                      # OPTIONAL: Default 8000
  cors:                           # OPTIONAL: CORS configuration
    enabled: true
    origins: ["*"]

# Models - REQUIRED (inline)
models:
  User:
    fields:
      id: "uuid, primary_key"
      email: "string, unique, required"
      name: "string, required"
      created_at: "timestamp, auto_add"

# Endpoints - REQUIRED (inline)
endpoints:
  - name: "users"                 # REQUIRED: Endpoint identifier  
    model: "User"                 # REQUIRED: Reference to model
    type: "crud"                  # REQUIRED: crud|custom
    auth_required: true           # OPTIONAL: Default false
```

#### Option 2: Modular Configuration (Complex Projects)

**Main File** (`brickend/brickend.yaml`):
```yaml
# Project definition - REQUIRED
project:
  name: "complex-api"
  version: "2.0.0"
  description: "Enterprise API with modular configuration"

# Database configuration - REQUIRED
database:
  type: postgresql
  name: complex_api_db

# Import configurations - OPTIONAL
imports:
  security: "security.yaml"      # Import auth & permissions
  api: "api.yaml"               # Import API standards
  
# Service modules - OPTIONAL
services:
  - name: "users"
    file: "services/users.yaml"
  - name: "organizations"  
    file: "services/organizations.yaml"
```

**Security Configuration** (`brickend/security.yaml`):
```yaml
# Authentication providers
auth:
  enabled: true
  type: jwt
  secret_key: "${JWT_SECRET}"
  
# Role-based permissions
roles:
  - name: "admin"
    permissions: ["users:*", "organizations:*"]
  - name: "user"
    permissions: ["users:read:own", "organizations:read"]
```

**API Standards** (`brickend/api.yaml`):
```yaml
# Global API configuration
global:
  version: "v1"
  base_path: "/api"
  
# Pagination defaults
pagination:
  default_page_size: 20
  max_page_size: 100
  
# Response standards
responses:
  include_metadata: true
  error_format: "detailed"
```

**Service Configuration** (`brickend/services/users.yaml`):
```yaml
# Service definition
service:
  name: "users"
  description: "User management service"

# Data models
models:
  User:
    fields:
      id: "uuid, primary_key"
      email: "string, unique, required"
      name: "string, required"
      role: "string, enum=admin|user"
      created_at: "timestamp, auto_add"

# API endpoints
endpoints:
  - name: "users"
    model: "User"
    type: "crud"
    auth_required: true
  - name: "profile"
    type: "custom"
    method: "GET"
    path: "/me"
    auth_required: true
```

### Field Definition Language

All implementations MUST parse field definitions using this syntax:

```
field_definition := type [, constraint]*
type := "string" | "text" | "integer" | "float" | "boolean" | "uuid" | "timestamp" | "date" | "json"
constraint := "required" | "unique" | "indexed" | "primary_key" | "auto_increment" | "auto_add" | "auto_update" | "max_length=" integer | "min_length=" integer | "default=" value
```

**Examples**:
```yaml
fields:
  id: "uuid, primary_key"
  email: "string, required, unique, max_length=255"  
  age: "integer, min_value=0, max_value=150"
  created_at: "timestamp, auto_add"
  is_active: "boolean, default=true"
```

### Validation Rules

**Configuration Discovery**:
- `brickend/brickend.yaml` MUST always exist (entry point)
- If `imports` section exists, referenced files MUST exist
- If `services` section exists, referenced service files MUST exist
- Environment files in `brickend/environments/` are auto-discovered if present

**Standalone Validation** (single file):
- All required sections MUST be present in `brickend.yaml`
- Field definitions MUST follow the universal syntax
- Model and endpoint references MUST be valid within the file

**Modular Validation** (multiple files):
- Base configuration in `brickend.yaml` MUST be valid
- Each imported file MUST be valid independently
- Cross-file references MUST resolve correctly
- Service names MUST be unique across all service files

**Universal Validation Rules**:
- Every model MUST have exactly one primary_key field
- Field names MUST be valid identifiers in all target languages
- Model and service names MUST be PascalCase and language-compatible
- Database provider MUST be supported by target language implementation

---

## Type System

### Universal Type Mapping

Each language implementation MUST provide exact mappings for these universal types:

| Universal Type | Description | Constraints |
|---------------|-------------|-------------|
| `string` | Variable-length text | max_length, min_length, pattern |
| `text` | Long-form text content | No length limits |
| `integer` | Signed 32-bit integer | min_value, max_value |
| `float` | IEEE 754 double precision | min_value, max_value |
| `boolean` | True/false value | None |
| `uuid` | RFC 4122 UUID | None |
| `timestamp` | ISO 8601 datetime with timezone | auto_add, auto_update |
| `date` | ISO 8601 date (YYYY-MM-DD) | None |
| `json` | Structured JSON object | schema validation (future) |

### Language-Specific Mapping Examples

**Python/FastAPI**:
```python
TYPE_MAPPING = {
    "string": "str",
    "integer": "int", 
    "float": "float",
    "boolean": "bool",
    "uuid": "UUID",
    "timestamp": "datetime",
    "date": "date",
    "json": "Dict[str, Any]"
}
```

**Go/Gin**:
```go
TYPE_MAPPING = map[string]string{
    "string":    "string",
    "integer":   "int32",
    "float":     "float64", 
    "boolean":   "bool",
    "uuid":      "uuid.UUID",
    "timestamp": "time.Time",
    "date":      "time.Time",
    "json":      "map[string]interface{}",
}
```

**TypeScript/Express**:
```typescript
TYPE_MAPPING = {
    "string": "string",
    "integer": "number",
    "float": "number", 
    "boolean": "boolean",
    "uuid": "string",
    "timestamp": "Date",
    "date": "Date", 
    "json": "Record<string, any>"
}
```

---

## Architecture Contract

### Required File Structure

Each language implementation MUST generate this logical structure:

```
generated-project/
├── brickend/               # Configuration directory (preserved)
│   ├── brickend.yaml       # Main configuration (always present)
│   ├── security.yaml       # Security config (optional)
│   ├── api.yaml            # API config (optional)
│   ├── environments/       # Environment configs (optional)
│   └── services/           # Service configs (optional)
├── [entry-point]           # Language-specific entry point
├── [dependency-file]       # Language-specific dependencies  
├── database/               # Database configuration
│   ├── connection.[ext]    # Database connection setup
│   └── migrations/         # Database migrations (if supported)
├── models/                 # Data models
│   ├── [model].[ext]       # One file per model
│   └── [relationships].[ext] # Model relationships
├── schemas/                # Validation schemas  
│   ├── [model].[ext]       # Model validation
│   └── [common].[ext]      # Shared validation logic
├── routes/                 # HTTP routes
│   ├── [endpoint].[ext]    # Route definitions
│   └── [auth].[ext]        # Authentication routes (if enabled)
├── middleware/             # HTTP middleware
│   ├── [auth].[ext]        # Authentication middleware (if enabled)
│   ├── [cors].[ext]        # CORS middleware (if enabled)
│   └── [validation].[ext]  # Request validation
└── utils/                  # Utility functions
    ├── [responses].[ext]   # Standard response helpers
    ├── [errors].[ext]      # Error handling
    └── [config].[ext]      # Configuration loaders
```

### Generation Contract

**Configuration Files** (user-owned, never overwritten):
- `brickend/brickend.yaml` - Main configuration (always required)
- `brickend/security.yaml` - Security and authentication settings (optional)
- `brickend/api.yaml` - API configuration and standards (optional)
- `brickend/services/[service].yaml` - Service definitions (optional)
- `brickend/environments/[env].yaml` - Environment configs (optional)

**Framework Files** (regenerated every time):
- Entry point and server configuration
- Route definitions and middleware setup  
- Base model and schema definitions from all configuration files
- Database connection and migration files
- Standard utility functions and configuration loaders
- Security middleware (if security.yaml exists)
- API middleware (if api.yaml exists)

**Business Logic Files** (generated once, user-modifiable):
- Route handler implementations
- Custom validation logic
- Business-specific utility functions
- Custom middleware implementations

### Naming Conventions

**Universal Naming Rules**:
- Model names: PascalCase (`User`, `BlogPost`)
- Field names: snake_case (`first_name`, `created_at`)  
- Endpoint names: kebab-case (`user-profiles`, `blog-posts`)
- File names: Follow target language conventions

**HTTP Route Patterns**:
```
CRUD endpoints MUST follow this pattern:
GET    /api/[endpoint]           # List resources
POST   /api/[endpoint]           # Create resource  
GET    /api/[endpoint]/{id}      # Get single resource
PUT    /api/[endpoint]/{id}      # Update resource
DELETE /api/[endpoint]/{id}      # Delete resource
```

---

## API Contract

### Response Format Standard

All implementations MUST return responses in this exact format:

**Success Response**:
```json
{
  "data": [actual_response_data],
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "request_id": "uuid-string"
  }
}
```

**Error Response**:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Human-readable error description",
    "details": {
      "field": "email",
      "constraint": "unique",
      "value": "duplicate@example.com"
    },
    "timestamp": "2024-01-15T10:30:00Z",
    "request_id": "uuid-string"
  }
}
```

### Standard Error Codes

All implementations MUST use these exact error codes and HTTP status mappings:

| Error Code | HTTP Status | Description |
|------------|-------------|-------------|
| `VALIDATION_ERROR` | 400 | Request validation failed |
| `UNAUTHORIZED` | 401 | Authentication required |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource does not exist |
| `CONFLICT` | 409 | Resource conflict (e.g., duplicate) |
| `UNPROCESSABLE_ENTITY` | 422 | Semantic validation failed |
| `INTERNAL_ERROR` | 500 | Unexpected server error |

### Authentication Contract

When `auth.enabled=true`, all implementations MUST:

1. **Protected endpoints** require valid authentication token
2. **Authentication header** format: `Authorization: Bearer [token]`
3. **JWT token structure** (if `auth.type=jwt`):
   ```json
   {
     "sub": "user-uuid",
     "exp": 1705320600,
     "iat": 1705234200,
     "user_id": "user-uuid"
   }
   ```
4. **Automatic auth endpoints**:
   - `POST /auth/login` - User authentication
   - `POST /auth/register` - User registration  
   - `GET /auth/me` - Current user profile

---

## Implementation Requirements

### Command Line Interface

Every language implementation MUST provide these exact commands:

```bash
# Project initialization
brickend init [project-name]    # Create new project with brickend/ folder and default brickend.yaml
brickend init --template [name] # Create from template (simple, modular, enterprise)

# Code generation  
brickend generate               # Generate code from brickend/brickend.yaml and any imported files
brickend generate --dry-run     # Show what would be generated
brickend generate --force       # Overwrite existing business logic files
brickend generate --service [name] # Generate specific service only (if modular structure)
brickend generate --env [name]  # Use environment-specific configuration

# Validation
brickend validate               # Validate brickend/brickend.yaml and all referenced files
brickend validate --service [name] # Validate specific service file (if exists)
brickend validate --verbose     # Show detailed validation results

# Modularization helpers (optional)
brickend split                  # Convert standalone config to modular structure
brickend merge                  # Convert modular config back to standalone

# Utilities
brickend version               # Show version information
brickend help                  # Show help information
```

### Environment Integration

**Required Environment Variables**:
```bash
BRICKEND_DATABASE_URL    # Database connection string
BRICKEND_JWT_SECRET      # JWT signing secret (if auth enabled)
BRICKEND_PORT           # Server port (optional, default from config)
BRICKEND_HOST           # Server host (optional, default from config)
```

**Development vs Production**:
- Development: Use config file values as defaults
- Production: Environment variables override config file values
- All implementations MUST support both modes

### Testing Requirements

Every implementation MUST generate:

1. **Unit tests** for model validation
2. **Integration tests** for API endpoints  
3. **Test fixtures** for development data
4. **Test configuration** separate from production config

**Test Structure**:
```
tests/
├── unit/
│   ├── models/         # Model validation tests
│   └── schemas/        # Schema validation tests  
├── integration/
│   ├── endpoints/      # API endpoint tests
│   └── auth/          # Authentication tests
└── fixtures/
    └── [model].json   # Test data fixtures
```

---

## Compliance Validation

### Cross-Language Test Suite

The main Brickend repository MUST maintain a test suite that validates compliance across all language implementations:

**Functional Equivalence Tests**:
```yaml
# Test configuration
test_config: |
  project:
    name: "compliance-test"
  models:
    User:
      fields:
        id: "uuid, primary_key"
        email: "string, unique, required"
  endpoints:
    - name: "users"
      model: "User" 
      type: "crud"

# Expected behaviors that MUST be identical across all languages
expected_behaviors:
  - endpoint: "POST /api/users"
    input: {"email": "test@example.com"}
    expected_status: 201
    expected_response_format: "success_response"
    
  - endpoint: "POST /api/users"  
    input: {"email": "test@example.com"}  # Duplicate
    expected_status: 409
    expected_error_code: "CONFLICT"
```

### Compliance Checklist

Before releasing any language implementation:

- [ ] All CLI commands work as specified
- [ ] All universal types map correctly
- [ ] All constraint types are enforced
- [ ] Error responses follow exact format
- [ ] HTTP status codes match specification
- [ ] Authentication works identically
- [ ] Generated file structure matches pattern
- [ ] Cross-language test suite passes 100%

### Version Compatibility

**Semantic Versioning**:
- MAJOR: Breaking changes to configuration schema or API contract
- MINOR: New features that maintain backward compatibility  
- PATCH: Bug fixes and internal improvements

**Cross-Language Sync**:
- All language implementations MUST support the same MAJOR.MINOR version
- PATCH versions can vary between languages
- Breaking changes require coordination across all implementations

---

## Ecosystem Standards

### Repository Structure

**Main Repository** (`brickend/brickend`):
- Framework principles and specifications (this document)
- Cross-language compliance test suite
- Documentation and examples
- Release coordination

**Language Repositories** (`brickend/brickend-[language]`):
- Language-specific implementation
- Language-specific templates and examples
- Language-specific documentation
- Independent release cycle (following main version)

### Documentation Standards

**Required Documentation** (per language repo):
- `README.md`: Installation and quick start
- `CONFIGURATION.md`: Complete YAML reference  
- `EXAMPLES.md`: Common use cases and patterns
- `CONTRIBUTING.md`: Development setup and guidelines
- `CHANGELOG.md`: Version history and breaking changes

**Cross-References**:
- All language repos MUST link to main repo for principles
- Main repo MUST link to all language implementations  
- Version compatibility matrix MUST be maintained

### Community Standards

**Issue Tracking**:
- Language-specific issues go to language repos
- Cross-language issues and spec changes go to main repo
- Security issues follow coordinated disclosure

**Feature Requests**:
- New universal features require RFC in main repo
- Language-specific optimizations handled in language repos
- Breaking changes require approval from all language maintainers

---

## Conclusion

These principles form the foundation of the Brickend ecosystem. They ensure that regardless of implementation language, developers have a consistent, predictable experience when building backend applications.

Every language implementation is bound by these contracts. Deviations require explicit documentation and community approval through the RFC process.

The goal is simple: write configuration once, deploy anywhere.

---

**Document Version**: 1.0  
**Last Updated**: September 26, 2025  
**Next Review**: Quarterly