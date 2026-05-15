# Integration Guide

This guide explains how to integrate this protocols repository with your microservices.

## Table of Contents

- [Integration Methods](#integration-methods)
- [Git Subtree Integration](#git-subtree-integration-recommended)
- [Git Submodule Integration](#git-submodule-integration)
- [Direct Go Module Import](#direct-go-module-import)
- [Version Pinning](#version-pinning)
- [Development Workflow](#development-workflow)
- [Troubleshooting](#troubleshooting)

## Integration Methods

Three main approaches for integrating protocols:

| Method | Pros | Cons | Use Case |
|--------|------|------|----------|
| **Git Subtree** | • Simple workflow<br>• No submodule complexity<br>• Files committed to repo | • Larger repo size<br>• Updates require subtree pull | **Recommended** for most projects |
| **Git Submodule** | • Small repo size<br>• Clear dependency tracking | • More complex workflow<br>• Easy to forget updates | Advanced users, multiple proto repos |
| **Go Module** | • Go native<br>• Standard dependency | • Requires public repo<br>• Go-specific | Go-only microservices |

## Git Subtree Integration (Recommended)

Git subtree embeds the protocols repository into your microservice repository.

### Initial Setup

In your microservice repository:

```bash
# Add protocols repository as a subtree
make proto-setup PROTO_REPO=https://github.com/andskur/protocols-template.git
```

This:
1. Adds protocols repository as a remote
2. Pulls protocols into `protocols/` directory
3. Commits to your repository

### Manual Setup

If your microservice doesn't have the Makefile targets:

```bash
# Add remote
git remote add protocols https://github.com/andskur/protocols-template.git

# Pull as subtree
git subtree add \
  --prefix protocols \
  protocols main \
  --squash

# Commit
git commit -m "Add protocols via git subtree"
```

### Directory Structure

After setup, your microservice will have:

```
your-microservice/
├── cmd/
├── internal/
├── protocols/              # ← Protocols repository
│   ├── common/v1/
│   │   ├── status.proto
│   │   ├── pagination.proto
│   │   └── ...
│   ├── user/v1/
│   │   └── user.proto
│   ├── buf.yaml
│   └── README.md
├── go.mod
└── README.md
```

### Updating Protocols

Pull latest changes from protocols repository:

```bash
make proto-update
```

Or manually:

```bash
git subtree pull \
  --prefix protocols \
  protocols main \
  --squash
```

### Generating Code

Generate Go code from protocols:

```bash
# Generate for specific package
make proto-generate PROTO_PACKAGE=user

# Generate for all packages
make proto-generate-all
```

Generated files go to: `protocols/<package>/v1/*.pb.go`

### Example Makefile Targets

Add to your microservice's `Makefile`:

```makefile
PROTO_DIR ?= protocols
PROTO_REPO ?= https://github.com/andskur/protocols-template.git
PROTO_PACKAGE ?=

.PHONY: proto-setup
proto-setup:
ifndef PROTO_REPO
	@echo "Error: PROTO_REPO parameter is required"
	@exit 1
endif
	@echo "Setting up protocols from $(PROTO_REPO)..."
	@git remote add protocols $(PROTO_REPO) 2>/dev/null || true
	@git subtree add --prefix $(PROTO_DIR) protocols main --squash

.PHONY: proto-update
proto-update:
	@echo "Updating protocols..."
	@git subtree pull --prefix $(PROTO_DIR) protocols main --squash

.PHONY: proto-generate
proto-generate:
ifndef PROTO_PACKAGE
	@echo "Error: PROTO_PACKAGE parameter is required"
	@exit 1
endif
	@cd $(PROTO_DIR)/$(PROTO_PACKAGE) && buf generate

.PHONY: proto-generate-all
proto-generate-all:
	@cd $(PROTO_DIR) && buf generate
```

## Git Submodule Integration

Git submodules link to external repositories without copying files.

### Initial Setup

```bash
# Add protocols as submodule
git submodule add \
  https://github.com/andskur/protocols-template.git \
  protocols

# Initialize and update
git submodule update --init --recursive

# Commit
git commit -m "Add protocols submodule"
```

### Cloning Repository with Submodules

```bash
# Clone with submodules
git clone --recursive https://github.com/yourorg/your-microservice.git

# Or if already cloned
git submodule update --init --recursive
```

### Updating Submodule

```bash
# Update to latest
cd protocols
git pull origin main
cd ..
git add protocols
git commit -m "Update protocols submodule"
```

### Generating Code

```bash
cd protocols
buf generate
```

### Pros and Cons

**Pros:**
- Smaller repository size
- Clear version tracking
- Easy to see which version is used

**Cons:**
- More complex workflow
- Team members may forget to update submodules
- CI/CD needs special handling

### Makefile Targets for Submodules

```makefile
.PHONY: proto-init
proto-init:
	@git submodule update --init --recursive

.PHONY: proto-update
proto-update:
	@git submodule update --remote protocols
	@git add protocols
	@git commit -m "Update protocols submodule" || true

.PHONY: proto-generate
proto-generate:
	@cd protocols && buf generate
```

## Direct Go Module Import

Import pre-generated Go code directly (requires public repository).

### Setup

In your `go.mod`:

```go
require (
    github.com/andskur/protocols-template v1.0.0
)
```

### Import in Code

```go
import (
    userv1 "github.com/andskur/protocols-template/user/v1"
    commonv1 "github.com/andskur/protocols-template/common/v1"
)

func main() {
    user := &userv1.User{
        Id:     "123",
        Email:  "[email protected]",
        Name:   "John Doe",
        Status: commonv1.CommonStatus_COMMON_STATUS_ACTIVE,
    }
}
```

### Updating

```bash
go get github.com/andskur/protocols-template@latest
```

### Pros and Cons

**Pros:**
- Standard Go workflow
- Easy dependency management
- Automatic updates via `go get`

**Cons:**
- Requires generated files in repo (violates best practice)
- Go-specific (not for other languages)
- Less control over code generation

**Note:** This method requires committing generated `.pb.go` files to the protocols repository, which is not recommended in this template.

## Version Pinning

### Pin to Specific Version

#### Git Subtree

```bash
# Pull specific tag
git subtree pull \
  --prefix protocols \
  protocols v1.2.0 \
  --squash
```

#### Git Submodule

```bash
cd protocols
git checkout v1.2.0
cd ..
git add protocols
git commit -m "Pin protocols to v1.2.0"
```

#### Go Module

```bash
go get github.com/andskur/protocols-template@v1.2.0
```

### Version Tracking

Document which version you're using in `README.md`:

```markdown
## Protocol Dependencies

- protocols-template: v1.2.0
```

## Development Workflow

### Full Integration Workflow

1. **Initial Setup**
   ```bash
   make proto-setup PROTO_REPO=https://github.com/andskur/protocols-template.git
   ```

2. **Generate Code**
   ```bash
   make proto-generate PROTO_PACKAGE=user
   ```

3. **Import in Your Code**
   ```go
   import userv1 "your-module/protocols/user/v1"
   
   func GetUser(ctx context.Context, id string) (*userv1.User, error) {
       // Implementation
   }
   ```

4. **Build and Test**
   ```bash
   go build ./...
   go test ./...
   ```

5. **Update Protocols (When Needed)**
   ```bash
   make proto-update
   make proto-generate-all
   go mod tidy
   ```

### Example: Using User Service

1. **Import Generated Code**
   ```go
   package main
   
   import (
       "context"
       "log"
       
       userv1 "your-module/protocols/user/v1"
       "google.golang.org/grpc"
   )
   
   func main() {
       conn, err := grpc.Dial("localhost:50051", grpc.WithInsecure())
       if err != nil {
           log.Fatal(err)
       }
       defer conn.Close()
       
       client := userv1.NewUserServiceClient(conn)
       
       resp, err := client.GetUser(context.Background(), &userv1.GetUserRequest{
           Id: "user-123",
       })
       if err != nil {
           log.Fatal(err)
       }
       
       log.Printf("User: %s (%s)", resp.User.Name, resp.User.Email)
   }
   ```

2. **Implement Server**
   ```go
   package server
   
   import (
       "context"
       
       userv1 "your-module/protocols/user/v1"
       commonv1 "your-module/protocols/common/v1"
       "google.golang.org/protobuf/types/known/timestamppb"
   )
   
   type UserServer struct {
       userv1.UnimplementedUserServiceServer
   }
   
   func (s *UserServer) GetUser(ctx context.Context, req *userv1.GetUserRequest) (*userv1.GetUserResponse, error) {
       user := &userv1.User{
           Id:        req.Id,
           Email:     "[email protected]",
           Name:      "John Doe",
           Status:    commonv1.CommonStatus_COMMON_STATUS_ACTIVE,
           CreatedAt: timestamppb.Now(),
           UpdatedAt: timestamppb.Now(),
       }
       
       return &userv1.GetUserResponse{User: user}, nil
   }
   ```

### CI/CD Integration

#### GitHub Actions Example

```yaml
name: Build

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive  # If using submodules
      
      - uses: actions/setup-go@v5
        with:
          go-version: '1.23'
      
      - name: Install Buf
        run: |
          go install github.com/bufbuild/buf/cmd/buf@latest
      
      - name: Generate protocols
        run: |
          make proto-generate-all
      
      - name: Build
        run: go build ./...
      
      - name: Test
        run: go test ./...
```

## Troubleshooting

### Subtree Pull Conflicts

```
CONFLICT (content): Merge conflict in protocols/...
```

**Solution:**

```bash
# Resolve conflicts manually
git mergetool

# Or abort and try with --squash
git merge --abort
git subtree pull --prefix protocols protocols main --squash
```

### Submodule Not Initialized

```
fatal: not a git repository: protocols/.git
```

**Solution:**

```bash
git submodule update --init --recursive
```

### Import Path Issues

```go
import "your-module/protocols/user/v1"  // Not found
```

**Solution:** Check `go.mod` module name matches import path:

```go
module your-module  // Must match import prefix

require (
    google.golang.org/grpc v1.60.0
    google.golang.org/protobuf v1.31.0
)
```

### Generated Files Missing

```
user/v1/user.pb.go: no such file or directory
```

**Solution:**

```bash
make proto-generate PROTO_PACKAGE=user
```

### Buf Not Found in CI

```
buf: command not found
```

**Solution:** Install in CI:

```yaml
- name: Install Buf
  run: go install github.com/bufbuild/buf/cmd/buf@latest
```

## Best Practices

1. **Pin to Versions:** Use specific versions (tags) in production

2. **Document Integration:** Add to README which method you use

3. **Automate Generation:** Include in build scripts/CI

4. **Test After Updates:** Run tests after updating protocols

5. **Version in README:** Document which protocol version you're using

6. **Use Make Targets:** Standardize commands across team

## Next Steps

- Read [PROTOCOL_DEVELOPMENT.md](PROTOCOL_DEVELOPMENT.md) for contributing protocols
- See [BUF_GUIDE.md](BUF_GUIDE.md) for Buf usage
- Check [VERSIONING.md](VERSIONING.md) for version management
