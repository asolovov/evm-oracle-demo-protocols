# Protocol Definitions Template

A comprehensive template repository for managing protobuf protocol definitions with both Buf and protoc workflows. Designed for seamless integration into Go microservices via git subtree.

> **Built to work with [go-microservice-template](https://github.com/andskur/go-microservice-template)** - This repository provides the protocol definitions layer for microservices built with the go-microservice-template.

## Features

- **Dual Workflow Support**: Use modern Buf tooling or traditional protoc
- **Comprehensive Linting**: Enforces best practices and consistency with Buf
- **Breaking Change Detection**: Automated validation against previous versions
- **Common Proto Library**: Reusable types for status, pagination, errors, and more
- **Automated CI/CD**: GitHub Actions for validation, documentation, and releases
- **Easy Service Scaffolding**: Scripts to quickly create new service definitions
- **Semantic Versioning**: Automated version management and changelog generation

## Quick Start

### Using as a Template

1. Click "Use this template" on GitHub to create your own protocols repository
2. Clone your new repository:
   ```bash
   git clone https://github.com/yourusername/your-protocols.git
   cd your-protocols
   ```

3. Install required tools:
   ```bash
   make install
   ```

4. Validate the proto files:
   ```bash
   make validate
   ```

### Integrating with Microservices

This repository is designed to work seamlessly with [go-microservice-template](https://github.com/andskur/go-microservice-template).

**Using with go-microservice-template:**

```bash
# In your microservice created from go-microservice-template
make proto-setup PROTO_REPO=https://github.com/andskur/protocols-template.git

# Update protocols
make proto-update

# Generate Go code
make proto-generate PROTO_PACKAGE=user
```

**Using with any Go microservice:**

Add this protocols repository using git subtree (see [docs/INTEGRATION.md](docs/INTEGRATION.md) for detailed instructions).

## Repository Structure

```
protocols-template/
├── .github/workflows/    # CI/CD automation
│   ├── ci.yml           # Linting, validation, code generation
│   ├── docs.yml         # Auto-generate documentation
│   └── release.yml      # Semantic versioning releases
│
├── common/v1/           # Shared protocol definitions
│   ├── status.proto     # Common status enums
│   ├── pagination.proto # Pagination patterns
│   ├── errors.proto     # Error structures
│   └── types.proto      # Common types (UUID, Money, Address)
│
├── user/v1/             # Example user service
│   └── user.proto       # User service definition
│
├── docs/                # Comprehensive documentation
│   ├── PROTOCOL_DEVELOPMENT.md
│   ├── VERSIONING.md
│   ├── BUF_GUIDE.md
│   ├── PROTOC_GUIDE.md
│   └── INTEGRATION.md
│
├── scripts/             # Automation scripts
│   ├── add-service.sh
│   ├── validate-buf.sh
│   └── validate-protoc.sh
│
├── buf.yaml            # Buf workspace configuration
├── buf.gen.yaml        # Code generation config
├── Makefile            # Build targets
└── AGENTS.md           # AI agent development guide
```

## Workflows

### Buf Workflow (Recommended)

Modern, fast, and feature-rich workflow using [Buf](https://buf.build):

```bash
# Install Buf
make buf-install

# Lint proto files
make buf-lint

# Check for breaking changes
make buf-breaking

# Generate Go code for a specific package
make buf-generate PACKAGE=user

# Generate for all packages
make buf-generate-all
```

### Protoc Workflow (Traditional)

Compatible with traditional protoc tooling:

```bash
# Install protoc plugins
make protoc-install

# Validate proto files
make protoc-validate

# Generate Go code for a specific package
make protoc-generate PACKAGE=user

# Generate for all packages
make protoc-generate-all
```

See [docs/BUF_GUIDE.md](docs/BUF_GUIDE.md) and [docs/PROTOC_GUIDE.md](docs/PROTOC_GUIDE.md) for detailed usage.

## Common Proto Definitions

This template includes reusable proto definitions in the `common/v1` package:

- **status.proto**: Standard entity states (Active, Inactive, Deleted)
- **pagination.proto**: Both offset/limit and cursor-based pagination
- **errors.proto**: Structured error responses with field-level details
- **types.proto**: Common types like UUID, Money, and Address

Import these in your services:

```protobuf
import "common/v1/status.proto";
import "common/v1/pagination.proto";

message User {
  string id = 1;
  string name = 2;
  common.v1.CommonStatus status = 3;
}
```

See [common/README.md](common/README.md) for details.

## Creating New Services

Use the scaffolding script to create a new service:

```bash
make add-service NAME=product
```

This creates:
- `product/v1/product.proto` with a basic service template
- `product/README.md` with documentation
- Updates `buf.yaml` to include the new module

Then customize the generated files and run:

```bash
make buf-lint
make buf-generate PACKAGE=product
```

See [docs/PROTOCOL_DEVELOPMENT.md](docs/PROTOCOL_DEVELOPMENT.md) for detailed guidance.

## Versioning

This repository follows semantic versioning:

- **Patch** (v1.0.X): Non-breaking changes like new fields, new services
- **Minor** (v1.X.0): New features, reserved for significant additions
- **Major** (vX.0.0): Breaking changes like removing fields or changing types

Proto package versions (v1, v2) are separate from repository versions.

Breaking changes are automatically detected in CI. See [docs/VERSIONING.md](docs/VERSIONING.md) for the complete policy.

## Available Make Targets

Run `make help` to see all available targets:

```bash
make help                    # Show all available targets
make install                 # Install all required tools
make validate               # Run all validation checks
make buf-lint               # Lint with Buf
make buf-breaking           # Check breaking changes
make buf-generate PACKAGE=x # Generate code with Buf
make protoc-validate        # Validate with protoc
make protoc-generate PACKAGE=x # Generate code with protoc
make add-service NAME=x     # Scaffold new service
make clean                  # Remove generated files
```

## CI/CD

GitHub Actions automatically:

1. **On Pull Requests**:
   - Lint all proto files
   - Detect breaking changes
   - Validate with both Buf and protoc
   - Test code generation

2. **On Push to Main**:
   - Run all validations
   - Generate API documentation
   - Create versioned releases with changelogs

3. **Documentation**:
   - Auto-generate protocol docs
   - Publish to GitHub Pages

## Documentation

Detailed guides are available in the `docs/` directory:

- **[PROTOCOL_DEVELOPMENT.md](docs/PROTOCOL_DEVELOPMENT.md)**: Adding services, fields, and enums
- **[VERSIONING.md](docs/VERSIONING.md)**: Semantic versioning and breaking changes
- **[BUF_GUIDE.md](docs/BUF_GUIDE.md)**: Using Buf for linting and generation
- **[PROTOC_GUIDE.md](docs/PROTOC_GUIDE.md)**: Using protoc (traditional workflow)
- **[INTEGRATION.md](docs/INTEGRATION.md)**: Integrating with microservices
- **[AGENTS.md](AGENTS.md)**: AI agent development guidelines

## Contributing

1. Create a feature branch from `main`
2. Add or modify proto definitions
3. Run `make validate` to check your changes
4. Commit with descriptive messages (follows conventional commits)
5. Open a pull request

CI will automatically validate your changes and check for breaking changes.

## License

MIT License - see [LICENSE](LICENSE) for details.

## Related Projects

- **[go-microservice-template](https://github.com/andskur/go-microservice-template)** - Production-ready Go microservice template with built-in support for this protocols repository. Use them together for a complete microservices solution.
- [Buf](https://buf.build) - Modern Protobuf tooling
- [gRPC](https://grpc.io) - High-performance RPC framework

## Support

For issues, questions, or suggestions:
- Open an issue on GitHub
- Check existing documentation in `docs/`
- Review examples in `user/v1/`
