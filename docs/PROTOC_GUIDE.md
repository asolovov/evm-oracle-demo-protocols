# Protoc Guide

This guide covers using traditional `protoc` (Protocol Buffer Compiler) for protocol development.

## Table of Contents

- [What is Protoc?](#what-is-protoc)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Code Generation](#code-generation)
- [Validation](#validation)
- [Comparison with Buf](#comparison-with-buf)
- [Troubleshooting](#troubleshooting)

## What is Protoc?

`protoc` is the official Protocol Buffer compiler from Google. It:

- Compiles `.proto` files into language-specific code
- Validates proto syntax
- Supports plugins for different languages
- Is the traditional tool for protobuf development

## Installation

### macOS

```bash
brew install protobuf
```

### Linux (Ubuntu/Debian)

```bash
sudo apt-get update
sudo apt-get install -y protobuf-compiler
```

### Linux (Fedora/RHEL)

```bash
sudo dnf install protobuf-compiler
```

### Windows

Download from [GitHub Releases](https://github.com/protocolbuffers/protobuf/releases):

1. Download `protoc-<version>-win64.zip`
2. Extract to `C:\protoc`
3. Add `C:\protoc\bin` to PATH

### From Source

```bash
git clone https://github.com/protocolbuffers/protobuf.git
cd protobuf
./autogen.sh
./configure
make
make install
```

### Verify Installation

```bash
protoc --version
# Output: libprotoc 3.21.0 (or similar)
```

## Installing Plugins

### Go Plugins

Install Go protobuf and gRPC plugins:

```bash
# Using make
make protoc-install

# Or manually
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

Verify plugins are in PATH:

```bash
which protoc-gen-go
which protoc-gen-go-grpc
```

### Other Language Plugins

For other languages, install respective plugins:

```bash
# Python
pip install grpcio-tools

# TypeScript
npm install -g protoc-gen-ts

# C++, Java, etc. (usually included with protoc)
```

## Basic Usage

### Compile a Single Proto

```bash
protoc \
  --proto_path=. \
  --go_out=. \
  --go_opt=paths=source_relative \
  user/v1/user.proto
```

Options:
- `--proto_path` or `-I`: Import path for proto files
- `--go_out`: Output directory for Go code
- `--go_opt`: Options for Go plugin

### Compile Multiple Protos

```bash
protoc \
  --proto_path=. \
  --go_out=. \
  --go_opt=paths=source_relative \
  --go-grpc_out=. \
  --go-grpc_opt=paths=source_relative \
  user/v1/*.proto
```

### Using Makefile Targets

This repository includes convenient Makefile targets:

```bash
# Validate specific package
make protoc-validate PACKAGE=user

# Validate all packages
make protoc-validate

# Generate code for specific package
make protoc-generate PACKAGE=user

# Generate code for all packages
make protoc-generate-all
```

## Code Generation

### Go Code Generation

Generate Go protobuf and gRPC code:

```bash
find user/v1 -name "*.proto" -exec protoc \
  --proto_path=. \
  --go_out=. \
  --go_opt=paths=source_relative \
  --go-grpc_out=. \
  --go-grpc_opt=paths=source_relative \
  {} \;
```

This generates:
- `user.pb.go`: Message definitions
- `user_grpc.pb.go`: Service definitions

### Output Path Modes

#### Source Relative (Recommended)

```bash
protoc \
  --go_opt=paths=source_relative \
  --go_out=. \
  user/v1/user.proto
```

Generates: `user/v1/user.pb.go` (next to source file)

#### Module Path

```bash
protoc \
  --go_opt=module=github.com/myorg/myrepo \
  --go_out=. \
  user/v1/user.proto
```

Generates: `github.com/myorg/myrepo/user/v1/user.pb.go`

#### Import Path

```bash
protoc \
  --go_opt=Muser/v1/user.proto=github.com/myorg/myrepo/user/v1 \
  --go_out=. \
  user/v1/user.proto
```

Maps proto files to specific Go import paths.

### Multiple Languages

Generate for multiple languages at once:

```bash
protoc \
  --proto_path=. \
  --go_out=. \
  --go_opt=paths=source_relative \
  --python_out=. \
  --java_out=. \
  user/v1/user.proto
```

## Validation

### Syntax Validation

Validate proto syntax without generating code:

```bash
protoc \
  --proto_path=. \
  --descriptor_set_out=/dev/null \
  user/v1/user.proto
```

### Using Makefile

```bash
make protoc-validate PACKAGE=user
```

### Validate All Protos

```bash
find . -name "*.proto" -exec protoc \
  --proto_path=. \
  --descriptor_set_out=/dev/null \
  {} \;
```

Or using make:

```bash
make protoc-validate
```

## Working with Imports

### Local Imports

For protos that import other protos:

```protobuf
import "common/v1/status.proto";
import "common/v1/pagination.proto";
```

Use `--proto_path` to specify search locations:

```bash
protoc \
  --proto_path=. \
  --proto_path=./common \
  --go_out=. \
  user/v1/user.proto
```

### Well-Known Types

Google's well-known types are included with protoc:

```protobuf
import "google/protobuf/timestamp.proto";
import "google/protobuf/duration.proto";
import "google/protobuf/empty.proto";
```

No special configuration needed - protoc knows where to find them.

### External Imports

For external proto dependencies:

1. Download dependency protos to a directory
2. Add to `--proto_path`:

```bash
protoc \
  --proto_path=. \
  --proto_path=./third_party \
  --go_out=. \
  user/v1/user.proto
```

## Comparison with Buf

| Feature | protoc | Buf |
|---------|--------|-----|
| **Speed** | Slower on large codebases | Very fast |
| **Linting** | Requires additional tools | Built-in, comprehensive |
| **Breaking Detection** | Manual scripts needed | Built-in |
| **Configuration** | Command-line flags | Simple YAML |
| **Error Messages** | Sometimes cryptic | Clear and actionable |
| **Dependency Management** | Manual | Automated (buf.lock) |
| **Maturity** | Very mature, stable | Newer, rapidly improving |
| **Ecosystem** | Larger (older) | Growing quickly |

### When to Use Protoc

Use protoc when:
- You're more comfortable with traditional tools
- You need maximum compatibility
- You're working with legacy projects
- Your CI/CD is already protoc-based

### When to Use Buf

Use Buf when:
- You want modern, fast tooling
- You need built-in linting and breaking detection
- You're starting a new project
- You want better developer experience

## Troubleshooting

### Protoc Not Found

```bash
protoc: command not found
```

**Solution:** Install protoc:

```bash
# macOS
brew install protobuf

# Linux
sudo apt-get install protobuf-compiler
```

### Plugin Not Found

```bash
protoc-gen-go: program not found or is not executable
```

**Solution:** Install and add to PATH:

```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
export PATH="$PATH:$(go env GOPATH)/bin"
```

### Import Not Found

```bash
user/v1/user.proto:3:8: common/v1/status.proto: File not found.
```

**Solution:** Add correct proto path:

```bash
protoc --proto_path=. --proto_path=./common ...
```

### Version Mismatch

```bash
WARNING: protoc is version 3.12.0, but generated code expects version 3.21.0
```

**Solution:** Update protoc:

```bash
brew upgrade protobuf  # macOS
```

Or update plugins:

```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
```

### Permission Denied

```bash
protoc: error while loading shared libraries
```

**Solution:** Check installation permissions and PATH.

## Best Practices

### 1. Use Makefile Targets

Instead of complex protoc commands:

```bash
make protoc-generate PACKAGE=user
```

### 2. Validate Before Generating

```bash
make protoc-validate
make protoc-generate-all
```

### 3. Keep Plugins Updated

```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

### 4. Use Source Relative Paths

```bash
--go_opt=paths=source_relative
```

Keeps generated files next to proto files.

### 5. Version Control

Don't commit generated files to this repository. Generate them in consuming services.

## Advanced Usage

### Descriptor Sets

Generate binary descriptor sets:

```bash
protoc \
  --descriptor_set_out=user.desc \
  --include_imports \
  user/v1/user.proto
```

Useful for:
- Runtime reflection
- Schema validation
- Documentation generation

### Custom Options

Define custom options:

```protobuf
import "google/protobuf/descriptor.proto";

extend google.protobuf.MessageOptions {
  string my_option = 50000;
}

message User {
  option (my_option) = "custom value";
  string id = 1;
}
```

### Plugins with Options

Pass options to plugins:

```bash
protoc \
  --go_out=. \
  --go_opt=paths=source_relative \
  --go_opt=Muser/v1/user.proto=github.com/myorg/myrepo/user/v1 \
  user/v1/user.proto
```

## Migration from Protoc to Buf

If you want to switch from protoc to Buf:

1. **Install Buf:**
   ```bash
   make buf-install
   ```

2. **Create buf.yaml:**
   ```bash
   buf config init
   ```

3. **Test validation:**
   ```bash
   buf lint
   buf breaking --against '.git#branch=main'
   ```

4. **Update CI/CD:**
   Replace protoc commands with buf commands

5. **Keep protoc for compatibility:**
   You can use both - Buf for development, protoc for CI if needed

## Additional Resources

- [Protoc Documentation](https://protobuf.dev/programming-guides/)
- [Protocol Buffers Language Guide](https://protobuf.dev/programming-guides/proto3/)
- [Go Generated Code Guide](https://protobuf.dev/reference/go/go-generated/)

## Next Steps

- Read [BUF_GUIDE.md](BUF_GUIDE.md) to learn about Buf (recommended)
- See [PROTOCOL_DEVELOPMENT.md](PROTOCOL_DEVELOPMENT.md) for development guidelines
- Check [INTEGRATION.md](INTEGRATION.md) for microservice integration
