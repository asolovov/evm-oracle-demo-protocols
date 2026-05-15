# Buf Guide

This guide covers using [Buf](https://buf.build) for protocol buffer development in this repository.

## Table of Contents

- [What is Buf?](#what-is-buf)
- [Installation](#installation)
- [Configuration](#configuration)
- [Linting](#linting)
- [Breaking Change Detection](#breaking-change-detection)
- [Code Generation](#code-generation)
- [Dependency Management](#dependency-management)
- [Troubleshooting](#troubleshooting)

## What is Buf?

Buf is a modern toolkit for working with Protocol Buffers. It provides:

- **Fast linting** with sensible defaults
- **Breaking change detection** against previous versions
- **Efficient code generation** with remote plugins
- **Dependency management** for proto files
- **Schema registry** for sharing protos (optional)

### Why Use Buf?

Compared to protoc:

| Feature | Buf | protoc |
|---------|-----|--------|
| Speed | ⚡ Very fast | Slower |
| Linting | Built-in, extensive | Requires plugins |
| Breaking detection | Built-in | Manual/custom scripts |
| Configuration | Simple YAML | Complex commands |
| Error messages | Clear, actionable | Often cryptic |
| Module system | Native support | Manual management |

## Installation

### Using Make (Recommended)

```bash
make buf-install
```

This installs the latest version of Buf via Go.

### Manual Installation

#### macOS

```bash
brew install bufbuild/buf/buf
```

#### Linux

```bash
# Binary install
BIN="/usr/local/bin" && \
VERSION="1.28.1" && \
  curl -sSL \
    "https://github.com/bufbuild/buf/releases/download/v${VERSION}/buf-$(uname -s)-$(uname -m)" \
    -o "${BIN}/buf" && \
  chmod +x "${BIN}/buf"
```

#### Using Go

```bash
go install github.com/bufbuild/buf/cmd/buf@latest
```

### Verify Installation

```bash
buf --version
```

## Configuration

### buf.yaml

The main configuration file for Buf:

```yaml
version: v2
lint:
  use:
    - DEFAULT
    - PACKAGE_DIRECTORY_MATCH
    - PACKAGE_VERSION_SUFFIX
  except:
    - PACKAGE_AFFILIATION_SUFFIX
  enum_zero_value_suffix: _UNSPECIFIED
  rpc_allow_same_request_response: false
  rpc_allow_google_protobuf_empty_requests: false
  rpc_allow_google_protobuf_empty_responses: false
  service_suffix: Service
breaking:
  use:
    - FILE
  ignore_unstable_packages: false
```

This configuration uses a single-module workspace, which allows packages to import from each other (e.g., `user.v1` can import from `common.v1`).

#### Lint Rules

- `DEFAULT`: Google's style guide rules
- `PACKAGE_DIRECTORY_MATCH`: Package name must match directory
- `PACKAGE_VERSION_SUFFIX`: Packages must end with v1, v2, etc.
- Custom rules for enums, services, and RPCs

#### Breaking Rules

- `FILE`: Check for breaking changes at file level
- Compares against previous git commits or tags

### buf.gen.yaml

Configuration for code generation:

```yaml
version: v2
plugins:
  - remote: buf.build/protocolbuffers/go
    out: .
    opt:
      - paths=source_relative
  - remote: buf.build/grpc/go
    out: .
    opt:
      - paths=source_relative
```

#### Plugins

- `protocolbuffers/go`: Generate Go protobuf code
- `grpc/go`: Generate Go gRPC service code

#### Options

- `paths=source_relative`: Generate files next to .proto files
- `out`: Output directory for generated code

## Linting

### Run Linting

```bash
# Lint all modules
make buf-lint

# Or directly
buf lint
```

### Common Lint Errors

#### Package Version Suffix

```
user/v1/user.proto:3:1:Package name "user" should be suffixed with a correctly formed version, such as "user.v1".
```

**Fix:** Add version suffix to package name:

```protobuf
package user.v1;  // Not just "user"
```

#### Enum Zero Value

```
user/v1/user.proto:10:3:Enum zero value name "ACTIVE" should be suffixed with "_UNSPECIFIED".
```

**Fix:** Use `_UNSPECIFIED` for zero value:

```protobuf
enum UserStatus {
  USER_STATUS_UNSPECIFIED = 0;  // Required
  USER_STATUS_ACTIVE = 1;
}
```

#### Service Suffix

```
user/v1/user.proto:15:1:Service name "User" should be suffixed with "Service".
```

**Fix:** Add "Service" suffix:

```protobuf
service UserService {  // Not just "User"
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
}
```

#### Request/Response Names

```
user/v1/user.proto:20:1:RPC request type "GetUserReq" should be named "GetUserRequest" or "<Service>GetUserRequest".
```

**Fix:** Use standard naming:

```protobuf
message GetUserRequest {  // Not "GetUserReq"
  string id = 1;
}
```

### Disabling Lint Rules

To disable specific rules (not recommended):

```yaml
lint:
  use:
    - DEFAULT
  except:
    - ENUM_ZERO_VALUE_SUFFIX  # Disable specific rule
```

## Breaking Change Detection

### Check for Breaking Changes

```bash
# Check against main branch
make buf-breaking

# Or directly
buf breaking --against '.git#branch=origin/main'
```

### Against Specific Tag

```bash
buf breaking --against '.git#tag=v1.0.0'
```

### Common Breaking Changes

#### Removed Field

```
user/v1/user.proto:15:3:Previously present field "3" with name "email" on message "User" was deleted.
```

**Solution:** Deprecate instead of removing:

```protobuf
message User {
  string id = 1;
  string name = 2;
  string email = 3 [deprecated = true];  // Don't remove
}
```

#### Changed Field Type

```
user/v1/user.proto:15:3:Field "2" on message "User" changed type from "string" to "int32".
```

**Solution:** Add new field, deprecate old one:

```protobuf
message User {
  string id = 1;
  string name = 2 [deprecated = true];
  int32 name_id = 3;  // New field
}
```

#### Changed Field Number

```
user/v1/user.proto:15:3:Field "email" on message "User" changed number from "3" to "4".
```

**Solution:** Never change field numbers. Add new field if needed.

### Intentional Breaking Changes

If you need to make breaking changes:

1. **Option 1:** Create new package version (v1 → v2)
   ```bash
   cp -r user/v1 user/v2
   # Update package name in v2
   ```

2. **Option 2:** Plan major version bump
   - Document breaking changes
   - Coordinate with consumers
   - Merge and trigger major release

## Code Generation

### Generate for Specific Package

```bash
make buf-generate PACKAGE=user
```

This generates Go code in the package directory.

### Generate for All Packages

```bash
make buf-generate-all
```

### Custom Generation

Edit `buf.gen.yaml` to customize:

```yaml
version: v2
plugins:
  - remote: buf.build/protocolbuffers/go
    out: gen/go  # Custom output directory
    opt:
      - paths=source_relative
      - Muser/v1/user.proto=github.com/myorg/myrepo/gen/go/user/v1
```

### Local Plugins

To use locally installed protoc plugins:

```yaml
version: v2
plugins:
  - local: protoc-gen-go
    out: .
    opt:
      - paths=source_relative
  - local: protoc-gen-go-grpc
    out: .
    opt:
      - paths=source_relative
```

Then ensure plugins are in PATH:

```bash
export PATH="$PATH:$(go env GOPATH)/bin"
buf generate
```

## Dependency Management

### buf.lock

Buf automatically manages dependencies in `buf.lock`:

```yaml
version: v2
deps:
  - remote: buf.build
    owner: googleapis
    repository: googleapis
    commit: 12345abcdef
```

This file is auto-generated and should be committed.

### Updating Dependencies

```bash
buf mod update
```

### Adding Dependencies

If you import external protos:

1. Add dependency to `buf.yaml`:
   ```yaml
   deps:
     - buf.build/googleapis/googleapis
   ```

2. Update lock file:
   ```bash
   buf mod update
   ```

## Troubleshooting

### Buf Not Found

```bash
buf: command not found
```

**Solution:** Install Buf:

```bash
make buf-install
```

### Import Not Found

```
user/v1/user.proto:5:8:common/v1/status.proto: does not exist
```

**Solution:** Check import path and module configuration:

```protobuf
import "common/v1/status.proto";  // Must match module path
```

### Module Not Found

```
Failure: module "common/v1" not found in buf.yaml
```

**Solution:** Add module to `buf.yaml`:

```yaml
modules:
  - path: common/v1  # Add missing module
```

### Lint Failures After Update

If lint suddenly fails after Buf update:

1. Check Buf version:
   ```bash
   buf --version
   ```

2. Update configuration if needed:
   ```bash
   buf config migrate-v1beta1
   ```

3. Review new lint rules:
   ```bash
   buf lint --error-format=json
   ```

### Breaking Change False Positives

If breaking detection flags a non-breaking change:

1. Verify the change is truly non-breaking
2. Check against correct baseline:
   ```bash
   buf breaking --against '.git#tag=v1.0.0' --debug
   ```

3. If necessary, exclude specific files:
   ```yaml
   breaking:
     ignore:
       - common/v1/experimental.proto
   ```

## Best Practices

1. **Run lint before committing:**
   ```bash
   make buf-lint
   ```

2. **Check breaking changes on PRs:**
   ```bash
   make buf-breaking
   ```

3. **Keep buf.lock committed:** Ensures reproducible builds

4. **Use remote plugins:** Faster than local plugins

5. **Follow lint rules:** Ensures consistency across team

6. **Update Buf regularly:**
   ```bash
   go install github.com/bufbuild/buf/cmd/buf@latest
   ```

## Additional Resources

- [Buf Documentation](https://buf.build/docs)
- [Buf CLI Reference](https://buf.build/docs/reference/cli)
- [Buf Style Guide](https://buf.build/docs/best-practices/style-guide)
- [Buf Breaking Change Rules](https://buf.build/docs/breaking/rules)

## Next Steps

- Read [PROTOC_GUIDE.md](PROTOC_GUIDE.md) for protoc workflow
- See [PROTOCOL_DEVELOPMENT.md](PROTOCOL_DEVELOPMENT.md) for development guidelines
- Check [VERSIONING.md](VERSIONING.md) for version management
