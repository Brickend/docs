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
# Project definition - REQUIRED
project:
  name: string              # REQUIRED: Project identifier
  version: string           # OPTIONAL: Semantic version
  description: string       # OPTIONAL: Project description

# Database configuration - REQUIRED
database:
  type: enum                # REQUIRED: postgresql|mysql|sqlite
  host: string              # OPTIONAL: Default localhost  
  port: integer             # OPTIONAL: Default per DB type
  name: string              # REQUIRED: Database name
  
# Model definitions - REQUIRED
models:
  [ModelName]:              # REQUIRED: At least one model
    fields:
      [field_name]: string  # REQUIRED: Field definition string

# API configuration - REQUIRED  
apis:
  config:                   # OPTIONAL: Global API settings
    host: string            # OPTIONAL: Default 127.0.0.1
    port: integer           # OPTIONAL: Default 8000
    cors:                   # OPTIONAL: CORS configuration
      enabled: boolean
      origins: string[]
    auth:                   # OPTIONAL: Authentication
      enabled: boolean
      type: enum            # jwt|api_key|bearer_token
      secret_key: string    # REQUIRED if auth.enabled
      
  endpoints:                # REQUIRED: At least one endpoint
    - name: string          # REQUIRED: Endpoint identifier  
      model: string         # REQUIRED: Reference to model
      type: enum            # REQUIRED: crud|custom
      auth_required: boolean # OPTIONAL: Default false
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

**Architecture Validation**:
- `architecture.yaml` MUST exist as the base configuration file
- All referenced service files in `service_modules` MUST exist
- If `api.config_file` is specified, the referenced file MUST exist
- If `services.auth.config_file` is specified, the referenced file MUST exist

**Service File Validation**:
- Every service MUST have exactly one primary_key field per table
- Field names MUST be valid identifiers in all target languages
- Service names MUST be valid in all target languages
- Referenced files in `extends` section MUST exist

**Cross-File Validation**:
- Service names in `architecture.yaml` MUST match service names in individual service files
- If `security.yaml` exists, referenced roles in service files MUST be defined
- If `api.yaml` exists, endpoint configurations MUST be compatible

**Security Validation**:
- If authentication is enabled, at least one provider MUST be configured
- RLS policies MUST reference existing tables
- Permission strings MUST follow the pattern: `resource:action:scope`

**Database Validation**:
- Database provider MUST be supported by the target language implementation
- Table names MUST be unique across all service files
- Foreign key references MUST point to existing tables and fields

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
├── brickend/               # Source configuration (preserved)
│   ├── architecture.yaml   # Base configuration
│   ├── security.yaml       # Authentication and authorization (optional)
│   ├── api.yaml            # API configuration (optional)
│   └── [service].yaml      # Service definitions (one or more)
├── [entry-point]           # Language-specific entry point
├── [dependency-file]       # Language-specific dependencies  
├── database/               # Database configuration
│   ├── connection.[ext]    # Database connection setup
│   └── migrations/         # Database migrations (if supported)
├── models/                 # Data models
│   ├── [model].[ext]       # One file per model
│   └── [relationships].[ext] # Model relationships
├── schemas/                # Validation schemas  
│   ├── [service].[ext]     # Service-specific validation
│   └── [common].[ext]      # Shared validation logic
├── routes/                 # HTTP routes
│   ├── [service].[ext]     # One file per service
│   └── [auth].[ext]        # Authentication routes
├── middleware/             # HTTP middleware
│   ├── [auth].[ext]        # Authentication middleware
│   ├── [cors].[ext]        # CORS middleware
│   ├── [security].[ext]    # Security middleware
│   └── [validation].[ext]  # Request validation
└── utils/                  # Utility functions
    ├── [responses].[ext]   # Standard response helpers
    ├── [errors].[ext]      # Error handling
    └── [config].[ext]      # Configuration loaders
```

### Generation Contract

**Configuration Files** (user-owned, never overwritten):
- `brickend/architecture.yaml` - Base configuration
- `brickend/security.yaml` - Security and authentication settings
- `brickend/api.yaml` - API configuration and standards
- `brickend/[service].yaml` - Service definitions and tables

**Framework Files** (regenerated every time):
- Entry point and server configuration
- Route definitions and middleware setup  
- Base model and schema definitions from all service files
- Database connection and migration files
- Standard utility functions and configuration loaders
- Security middleware from security.yaml
- API middleware from api.yaml

**Business Logic Files** (generated once, user-modifiable):
- Service-specific route handler implementations
- Custom validation logic per service
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
brickend init [project-name]    # Create new project with brickend/ folder and default architecture.yaml
brickend init --template [name] # Create from template

# Code generation  
brickend generate               # Generate code from brickend/architecture.yaml and all service files
brickend generate --dry-run     # Show what would be generated
brickend generate --force       # Overwrite existing business logic files
brickend generate --service [name] # Generate specific service only
brickend generate --env [name]  # Use environment-specific configuration

# Validation
brickend validate               # Validate brickend/architecture.yaml and all referenced files
brickend validate --service [name] # Validate specific service file
brickend validate --security    # Validate security.yaml configuration
brickend validate --api         # Validate api.yaml configuration
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