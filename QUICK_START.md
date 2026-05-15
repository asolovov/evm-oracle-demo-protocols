# Quick Start Guide

Get up and running with protocols-template in minutes.

## For Template Users

### 1. Create Your Protocols Repository

Click "Use this template" on GitHub or:

```bash
git clone https://github.com/andskur/protocols-template.git my-protocols
cd my-protocols
rm -rf .git
git init
git add .
git commit -m "Initial commit from protocols-template"
```

### 2. Install Tools

```bash
make install
```

This installs:
- Buf CLI
- protoc-gen-go
- protoc-gen-go-grpc

### 3. Validate Everything Works

```bash
make validate
```

### 4. Create Your First Service

```bash
make add-service NAME=product
```

This creates:
- `product/v1/product.proto`
- `product/README.md`
- Updates `buf.yaml`

### 5. Customize and Validate

Edit `product/v1/product.proto`, then:

```bash
make buf-lint
make buf-generate PACKAGE=product
```

## For Microservice Developers

### 1. Add Protocols to Your Microservice

```bash
# In your microservice repository
make proto-setup PROTO_REPO=https://github.com/yourorg/your-protocols.git
```

### 2. Generate Go Code

```bash
make proto-generate PROTO_PACKAGE=user
```

### 3. Import in Your Code

```go
import (
    userv1 "your-module/protocols/user/v1"
    commonv1 "your-module/protocols/common/v1"
)
```

### 4. Use the Generated Code

```go
// Create a user
user := &userv1.User{
    Id:     "123",
    Email:  "[email protected]",
    Name:   "John Doe",
    Status: commonv1.CommonStatus_COMMON_STATUS_ACTIVE,
}
```

### 5. Update Protocols When Needed

```bash
make proto-update
make proto-generate-all
```

## Common Commands

### Validation

```bash
make buf-lint              # Lint with Buf
make buf-breaking          # Check breaking changes
make protoc-validate       # Validate with protoc
make validate              # Run all validations
```

### Code Generation

```bash
make buf-generate PACKAGE=user        # Generate one package (Buf)
make buf-generate-all                 # Generate all packages (Buf)
make protoc-generate PACKAGE=user     # Generate one package (protoc)
make protoc-generate-all              # Generate all packages (protoc)
```

### Development

```bash
make add-service NAME=product         # Create new service
make clean                            # Remove generated files
make help                             # Show all commands
```

## File Locations

After integration, find files here:

```
your-microservice/
├── protocols/              # ← Protocols repository
│   ├── common/v1/
│   │   ├── status.proto
│   │   ├── status.pb.go      # ← Generated
│   │   └── ...
│   ├── user/v1/
│   │   ├── user.proto
│   │   ├── user.pb.go        # ← Generated
│   │   └── user_grpc.pb.go   # ← Generated
│   └── buf.yaml
└── ...
```

## Import Paths

```go
import (
    // User service
    userv1 "your-module/protocols/user/v1"
    
    // Common types
    commonv1 "your-module/protocols/common/v1"
    
    // Well-known types
    "google.golang.org/protobuf/types/known/timestamppb"
)
```

## Typical Workflow

### 1. Developer Updates Protocols

```bash
# In protocols repository
cd my-protocols

# Create new service or update existing
vim user/v1/user.proto

# Validate
make buf-lint
make buf-breaking

# Commit and push
git add .
git commit -m "Add phone field to User"
git push
```

### 2. Microservice Pulls Updates

```bash
# In microservice repository
cd my-microservice

# Pull latest protocols
make proto-update

# Regenerate code
make proto-generate-all

# Test
go test ./...

# Commit
git add .
git commit -m "Update protocols with phone field"
git push
```

## Troubleshooting

### "buf: command not found"

```bash
make buf-install
```

### "protoc: command not found"

**macOS:**
```bash
brew install protobuf
```

**Linux:**
```bash
sudo apt-get install protobuf-compiler
```

### Import errors in generated code

```bash
# Regenerate all packages
make clean
make buf-generate-all

# Tidy Go modules
go mod tidy
```

### Breaking change detected

Either:
1. Revert the breaking change
2. Deprecate instead of removing
3. Create new package version (v1 → v2)
4. Plan major version bump

See [docs/VERSIONING.md](docs/VERSIONING.md) for details.

## Common Patterns

### Adding a Field (Non-Breaking)

```protobuf
message User {
  string id = 1;
  string email = 2;
  string name = 3;
  string phone = 4;  // ✅ New field, safe to add
}
```

### Deprecating a Field

```protobuf
message User {
  string id = 1;
  string email = 2 [deprecated = true];  // ✅ Deprecate
  string email_address = 3;               // New replacement
}
```

### Using Common Types

```protobuf
import "common/v1/status.proto";
import "common/v1/pagination.proto";

message ListUsersRequest {
  common.v1.PageRequest page = 1;
  common.v1.CommonStatus status = 2;
}
```

## Next Steps

- **New to protocols?** Read [README.md](README.md)
- **Contributing?** See [docs/PROTOCOL_DEVELOPMENT.md](docs/PROTOCOL_DEVELOPMENT.md)
- **Using Buf?** Check [docs/BUF_GUIDE.md](docs/BUF_GUIDE.md)
- **Integrating?** Review [docs/INTEGRATION.md](docs/INTEGRATION.md)
- **Versioning?** Read [docs/VERSIONING.md](docs/VERSIONING.md)
- **AI Agent?** See [AGENTS.md](AGENTS.md)

## Getting Help

- Check documentation in `docs/`
- Review examples in `user/v1/`
- Run `make help` for available commands
- See existing proto files for patterns

## Quick Reference

| Task | Command |
|------|---------|
| Install tools | `make install` |
| Lint protos | `make buf-lint` |
| Check breaking | `make buf-breaking` |
| Generate code | `make buf-generate PACKAGE=user` |
| Create service | `make add-service NAME=product` |
| Update in microservice | `make proto-update` |
| Clean generated | `make clean` |
| Show help | `make help` |
