# Brickend

> **Configuration-driven backend generation for any language**

Brickend transforms YAML configuration into production-ready backend code. Define your architecture once, generate it in Python, TypeScript, Go, Rust, or any supported language.

---

## Core Philosophy

**Configuration over Code**: Describe what you want, not how to build it.

**Language Agnostic**: The same YAML produces functionally identical backends across all implementations.

**Copy-Paste Ownership**: Generated code is yours. No runtime dependencies. No vendor lock-in.

**Progressive Complexity**: Start with a single file. Scale to enterprise architecture when needed.

---

## ✨ Features

**CLI-Driven Workflow**
- `brickend init` → Create project structure with configuration templates
- `brickend generate` → Generate models, routes, schemas, and migrations from YAML
- `brickend validate` → Validate configuration and detect issues
- `brickend upgrade` → Update framework code while preserving business logic

**Flexible Configuration**
- **Simple**: Single `brickend/brickend.yaml` for quick prototypes
- **Modular**: Split into `security.yaml`, `api.yaml`, and service files
- **Enterprise**: Full separation with environment-specific configs

**Universal Type System**
- Consistent types across all languages (string, integer, uuid, timestamp, etc.)
- Automatic mapping to native types (str→string→String)
- Same constraints work everywhere (required, unique, max_length, etc.)

**Smart Code Generation**
- Database models with proper types and constraints
- API routes with CRUD operations
- Validation schemas (runtime + compile-time where available)
- Authentication middleware
- Migration files

**Safe Customization**
- Framework files regenerated on changes
- Business logic files generated once, never touched
- Clear separation between framework and your code

**Automatic Migrations**
- Compare YAML changes to generate migrations
- Safe operations applied automatically
- Breaking changes require explicit confirmation

---

## Supported Implementations

| Language | Frameworks | Status |
|----------|------------|--------|
| **TypeScript** | Supabase Functions, Express, NestJS | ✅ Active Development |
| **Python** | FastAPI, Django, Flask | 🔜 Planned Q1 2026 |
| **Go** | Gin, Fiber, Echo | 🔜 Planned Q1 2026 |
| **Rust** | Axum, Actix | 🔜 Planned Q2 2026 |

All implementations:
- Use identical YAML configuration
- Generate functionally equivalent APIs
- Follow the same architectural patterns
- Support the same CLI commands

---

## 🚀 Quick Start

### Installation

```bash
# Install for your language
npm install -g @brickend/cli-typescript
pip install brickend-python
go install github.com/brickend/cli-go
cargo install brickend-cli
```

### Create Your First API

**1. Initialize Project**

```bash
brickend init my-api --language typescript
cd my-api
```

Creates:
```
my-api/
└── brickend/
    └── brickend.yaml
```

**2. Define Configuration** (`brickend/brickend.yaml`)

```yaml
project:
  name: "my-api"

database:
  type: postgresql
  name: my_api_db

models:
  User:
    fields:
      id: "uuid, primary_key"
      email: "string, unique, required, max_length=255"
      name: "string, required"
      created_at: "timestamp, auto_add"

endpoints:
  - name: "users"
    model: "User"
    type: "crud"
    auth_required: true
```

**3. Generate Code**

```bash
brickend generate
```

Generates:
- Database models
- CRUD endpoints
- Validation schemas
- Authentication middleware
- Complete project structure

**4. Run Your API**

```bash
# Language-specific commands
npm run dev        # TypeScript
python main.py     # Python
go run main.go     # Go
cargo run          # Rust
```

Your API is now running with:
- `GET /api/users` - List users
- `POST /api/users` - Create user
- `GET /api/users/:id` - Get user
- `PUT /api/users/:id` - Update user
- `DELETE /api/users/:id` - Delete user
- Authentication endpoints

---

## 📂 Project Structure

All languages generate the same logical structure:

```
my-api/
├── brickend/                    # Configuration (preserved)
│   ├── brickend.yaml            # Main config
│   ├── security.yaml            # Auth & permissions (optional)
│   ├── api.yaml                 # API standards (optional)
│   └── services/                # Service modules (optional)
│       ├── users.yaml
│       └── posts.yaml
│
├── [entry-point]                # Language-specific entry
├── [dependency-file]            # Package management
│
├── database/
│   ├── connection.[ext]         # DB connection
│   └── migrations/              # Migration files
│
├── models/                      # Data models
│   ├── user.[ext]
│   └── post.[ext]
│
├── schemas/                     # Validation
│   ├── user.[ext]
│   └── post.[ext]
│
├── routes/                      # HTTP routes
│   ├── users.[ext]
│   └── posts.[ext]
│
├── middleware/                  # HTTP middleware
│   ├── auth.[ext]
│   ├── cors.[ext]
│   └── validation.[ext]
│
└── utils/                       # Utilities
    ├── responses.[ext]
    ├── errors.[ext]
    └── config.[ext]
```

---

## Configuration Options

### Simple (Single File)

Everything in one place for quick projects:

```yaml
project:
  name: "simple-api"

database:
  type: postgresql
  name: simple_db

auth:
  enabled: true
  type: jwt
  secret_key: "${JWT_SECRET}"

api:
  host: "0.0.0.0"
  port: 8000
  cors:
    enabled: true
    origins: ["*"]

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

### Modular (Split Files)

Better organization for growing projects:

**`brickend/brickend.yaml`:**
```yaml
project:
  name: "modular-api"

database:
  type: postgresql
  name: modular_db

imports:
  security: "security.yaml"
  api: "api.yaml"

services:
  - name: "users"
    file: "services/users.yaml"
  - name: "posts"
    file: "services/posts.yaml"
```

**`brickend/security.yaml`:**
```yaml
auth:
  enabled: true
  type: jwt
  secret_key: "${JWT_SECRET}"

roles:
  - name: "admin"
    permissions: ["*"]
  - name: "user"
    permissions: ["users:read:own", "posts:*:own"]
```

**`brickend/api.yaml`:**
```yaml
global:
  version: "v1"
  base_path: "/api"

pagination:
  default_page_size: 20
  max_page_size: 100

cors:
  enabled: true
  origins: ["https://app.example.com"]
```

**`brickend/services/users.yaml`:**
```yaml
service:
  name: "users"

models:
  User:
    fields:
      id: "uuid, primary_key"
      email: "string, unique, required"
      name: "string, required"
      created_at: "timestamp, auto_add"

endpoints:
  - name: "users"
    model: "User"
    type: "crud"
    auth_required: true
```

### Convert Between Structures

```bash
# Convert standalone to modular
brickend split

# Convert modular to standalone
brickend merge
```

---

## Universal Type System

Types map consistently across all languages:

| Universal | Python | Go | TypeScript | Rust |
|-----------|--------|-----|-----------|------|
| `string` | `str` | `string` | `string` | `String` |
| `integer` | `int` | `int32` | `number` | `i32` |
| `float` | `float` | `float64` | `number` | `f64` |
| `boolean` | `bool` | `bool` | `boolean` | `bool` |
| `uuid` | `UUID` | `uuid.UUID` | `string` | `Uuid` |
| `timestamp` | `datetime` | `time.Time` | `Date` | `DateTime` |
| `date` | `date` | `time.Time` | `Date` | `NaiveDate` |
| `json` | `Dict` | `map[string]interface{}` | `Record` | `Value` |

### Field Definition Syntax

```yaml
models:
  Product:
    fields:
      # Basic types
      name: "string, required"
      description: "text"
      price: "float, required"
      in_stock: "boolean, default=true"
      
      # With constraints
      email: "string, unique, max_length=255"
      age: "integer, min_value=0, max_value=150"
      
      # Enums
      status: "string, enum=draft|published|archived"
      
      # Auto-generated
      id: "uuid, primary_key"
      created_at: "timestamp, auto_add"
      updated_at: "timestamp, auto_update"
```

---

## API Contract

All implementations produce identical API behavior:

### Standard Responses

**Success:**
```json
{
  "data": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "email": "user@example.com",
    "name": "John Doe"
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "request_id": "req_abc123"
  }
}
```

**Error:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": {
      "field": "email",
      "constraint": "unique"
    },
    "timestamp": "2024-01-15T10:30:00Z",
    "request_id": "req_abc123"
  }
}
```

### Standard Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `VALIDATION_ERROR` | 400 | Request validation failed |
| `UNAUTHORIZED` | 401 | Authentication required |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `CONFLICT` | 409 | Resource conflict |
| `UNPROCESSABLE_ENTITY` | 422 | Business logic validation failed |
| `INTERNAL_ERROR` | 500 | Unexpected server error |

### Standard Routes

```
GET    /api/users              # List all
POST   /api/users              # Create
GET    /api/users/:id          # Get one
PUT    /api/users/:id          # Update
DELETE /api/users/:id          # Delete
```

---

## Advanced Features

### Authentication

```yaml
auth:
  enabled: true
  type: jwt
  secret_key: "${JWT_SECRET}"
  expire_minutes: 30
```

Automatically generates:
- `POST /auth/login`
- `POST /auth/register`
- `GET /auth/me`
- Token validation middleware

### Relations

```yaml
models:
  User:
    fields:
      id: "uuid, primary_key"
      email: "string, unique, required"
      
  Post:
    fields:
      id: "uuid, primary_key"
      title: "string, required"
      author_id: "uuid, required"
    relations:
      - type: "many_to_one"
        model: "User"
        field: "author_id"
```

### Query Parameters

```yaml
endpoints:
  - name: "posts"
    model: "Post"
    type: "list"
    query_params:
      - page: "integer, optional, default=1"
      - limit: "integer, optional, default=20, max=100"
      - status: "string, optional, enum=draft|published"
      - search: "string, optional"
      - sort_by: "string, optional, enum=created_at|title"
      - order: "string, optional, enum=asc|desc"
```

### Custom Endpoints

```yaml
endpoints:
  - name: "publish-post"
    type: "custom"
    method: "POST"
    path: "/posts/:id/publish"
    input:
      - scheduled_at: "timestamp, optional"
    output:
      - published: "boolean"
      - published_at: "timestamp"
```

---

## Code Generation Examples

The same YAML generates equivalent code in each language:

**TypeScript:**
```typescript
// models/user.ts
export interface User {
  id: string;
  email: string;
  name: string;
  created_at: Date;
}

// routes/users.ts
router.post('/users', async (req, res) => {
  const user = await createUser(req.body);
  res.json({ data: user });
});
```

**Python:**
```python
# models/user.py
class User(Base):
    id: UUID
    email: str
    name: str
    created_at: datetime

# routes/users.py
@router.post("/users")
async def create_user(data: CreateUserInput) -> User:
    user = await db.users.create(data)
    return user
```

**Go:**
```go
// models/user.go
type User struct {
    ID        uuid.UUID `json:"id"`
    Email     string    `json:"email"`
    Name      string    `json:"name"`
    CreatedAt time.Time `json:"created_at"`
}

// routes/users.go
func CreateUser(c *gin.Context) {
    var input CreateUserInput
    c.BindJSON(&input)
    user := db.CreateUser(input)
    c.JSON(200, gin.H{"data": user})
}
```

All three APIs behave identically.

---

## CLI Commands

### Project Management

```bash
# Initialize new project
brickend init [name]
brickend init my-api --language python
brickend init my-api --template modular

# Available templates
simple      # Single file configuration
modular     # Split configuration files
enterprise  # Full modular structure
```

### Code Generation

```bash
# Generate all code
brickend generate

# Preview changes (dry run)
brickend generate --dry-run

# Generate specific service
brickend generate --service users

# Use environment config
brickend generate --env production
```

### Validation

```bash
# Validate configuration
brickend validate

# Validate specific service
brickend validate --service users

# Detailed output
brickend validate --verbose
```

### Configuration Management

```bash
# Convert structures
brickend split    # Standalone → Modular
brickend merge    # Modular → Standalone

# Show configuration
brickend config show
```

---

## Database Migrations

Brickend automatically generates migrations:

```bash
brickend generate
```

Creates migration files in `database/migrations/`:
```
001_create_users_table.sql
002_add_posts_table.sql
003_add_email_index.sql
```

### Migration Strategy

**Safe Changes (automatic):**
- Adding new tables
- Adding new fields with defaults
- Creating indexes
- Adding constraints

**Breaking Changes (requires confirmation):**
- Dropping tables
- Dropping fields
- Changing field types
- Modifying constraints

```bash
# Review breaking changes
brickend generate --dry-run

# Apply with confirmation
brickend generate --allow-breaking
```

---

## Testing

All implementations generate test scaffolding:

```
tests/
├── unit/
│   ├── models/
│   └── schemas/
├── integration/
│   ├── endpoints/
│   └── auth/
└── fixtures/
    └── data.json
```

**Python (pytest):**
```python
def test_create_user(client):
    response = client.post('/api/users', json={
        'email': 'test@example.com',
        'name': 'Test User'
    })
    assert response.status_code == 201
```

**TypeScript (Jest):**
```typescript
test('create user', async () => {
  const response = await request(app)
    .post('/api/users')
    .send({ email: 'test@example.com' });
  expect(response.status).toBe(201);
});
```

**Go (testing):**
```go
func TestCreateUser(t *testing.T) {
    user, err := CreateUser(input)
    assert.Nil(t, err)
    assert.Equal(t, "test@example.com", user.Email)
}
```

---

## Deployment

### Environment Variables

All implementations use consistent variables:

```bash
DATABASE_URL=postgresql://user:pass@localhost/db
JWT_SECRET=your-secret-key
API_PORT=8000
ENVIRONMENT=production
```

### Docker Support

Auto-generated Dockerfiles:

```dockerfile
# TypeScript
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 8000
CMD ["npm", "start"]

# Python
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
EXPOSE 8000
CMD ["python", "main.py"]

# Go
FROM golang:1.21-alpine
WORKDIR /app
COPY . .
RUN go build -o app
EXPOSE 8000
CMD ["./app"]
```

### Cloud Platforms

Brickend-generated projects work with:
- Vercel / Netlify
- AWS Lambda / Google Cloud Run
- Heroku / Railway / Fly.io
- Docker / Kubernetes

---

## Upgrade Process

Preserve your customizations during upgrades:

```bash
brickend upgrade
```

**What Gets Updated:**
- Framework files (routes, utilities, middleware)
- Configuration schema (new features)
- Generated contracts and types

**What's Preserved:**
- Your business logic
- Configuration files
- Custom code
- Database data

---

## Multi-Language Example

The same configuration works everywhere:

**`brickend/brickend.yaml`:**
```yaml
project:
  name: "blog-api"

database:
  type: postgresql

models:
  Post:
    fields:
      id: "uuid, primary_key"
      title: "string, required"
      content: "text, required"
      published: "boolean, default=false"

endpoints:
  - name: "posts"
    model: "Post"
    type: "crud"
```

**Generate in any language:**
```bash
# TypeScript
brickend init blog-api --language typescript
brickend generate

# Python
brickend init blog-api --language python
brickend generate

# Go
brickend init blog-api --language go
brickend generate
```

All three produce functionally identical APIs.

---

## Principles

Brickend follows strict principles:

1. **Configuration Determinism** - Same config = same behavior
2. **Minimal Abstraction** - Only universal concepts
3. **Error Surface Minimization** - Clear error messages
4. **Idempotent Generation** - Reproducible builds
5. **Non-Destructive Updates** - Your code is safe

Read full principles: [PRINCIPLES.md](./PRINCIPLES.md)

---

## Roadmap

**Current (MVP)**
- TypeScript implementation (Supabase, Express, NestJS)
- YAML configuration system
- CRUD generation
- Authentication & authorization
- Database migrations
- Type-safe contracts

**Q1 2026**
- Python implementation (FastAPI, Django, Flask)
- Go implementation (Gin, Fiber, Echo)
- Relations and foreign keys
- Advanced filtering

**Q2 2026**
- Rust implementation (Axum, Actix)
- GraphQL support
- Real-time subscriptions
- Event-driven architecture

**Future**
- AI-powered optimization
- Multi-database support
- Microservices orchestration
- Frontend integration

---

## Community

- **Docs**: [docs.brickend.dev](https://docs.brickend.dev)
- **Discord**: [discord.gg/brickend](https://discord.gg/brickend)
- **GitHub**: [github.com/brickend/brickend](https://github.com/brickend/brickend)

---

## Contributing

Want to add support for a new language?

1. Fork `brickend/brickend-[language]`
2. Implement the language adapter
3. Pass compliance test suite
4. Submit for review

See [CONTRIBUTING.md](./CONTRIBUTING.md) for details.

---

## License

MIT License - Generated code is yours, no attribution required.

---

**Write once, deploy anywhere.**
