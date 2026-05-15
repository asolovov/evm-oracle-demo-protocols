# Agent Guide: Protocol Development

This guide provides instructions for AI agents and developers working with protocol definitions in this repository.

## Core Principles

1. **Consistency First**: All protocols must follow the same patterns and conventions
2. **Documentation Required**: Every message, field, enum, and service must have comments
3. **Breaking Changes**: Must be detected and handled carefully
4. **Backward Compatibility**: Maintain compatibility whenever possible
5. **Versioning**: Use semantic versioning for repository, package versioning for protos

## Protocol Naming Conventions

### Package Names

- Format: `<service>.v<version>`
- Examples: `user.v1`, `product.v2`, `common.v1`
- Always lowercase
- Must include version suffix (enforced by Buf lint)

### Service Names

- Format: `<Name>Service`
- Must end with "Service" suffix (enforced by Buf lint)
- PascalCase
- Examples: `UserService`, `ProductService`, `OrderService`

### Message Names

- PascalCase
- Request messages: Must end with `Request` (e.g., `GetUserRequest`)
- Response messages: Must end with `Response` (e.g., `GetUserResponse`)
- Descriptive of their purpose
- Examples: `User`, `CreateUserRequest`, `ListUsersResponse`

### RPC Method Names

- PascalCase
- Use standard CRUD verbs: `Get`, `List`, `Create`, `Update`, `Delete`
- Examples: `GetUser`, `ListUsers`, `CreateUser`, `UpdateUser`, `DeleteUser`
- Avoid: `Fetch`, `Retrieve`, `Remove`, `Insert`

### Field Names

- snake_case (lowercase with underscores)
- Descriptive and unambiguous
- Examples: `user_id`, `created_at`, `email_address`, `total_count`
- Avoid abbreviations unless universally understood (e.g., `id` is acceptable)

### Enum Names

- PascalCase for enum type
- SCREAMING_SNAKE_CASE for enum values
- Format: `<ENUM_NAME>_<VALUE>`
- Must include `_UNSPECIFIED` as value 0 (enforced by Buf lint)
- Examples:
  ```protobuf
  enum UserStatus {
    USER_STATUS_UNSPECIFIED = 0;
    USER_STATUS_ACTIVE = 1;
    USER_STATUS_INACTIVE = 2;
  }
  ```

## Documentation Requirements

Every protocol element must have documentation comments:

### Service Documentation

```protobuf
// UserService provides operations for managing user accounts.
service UserService {
  // GetUser retrieves a user by their unique identifier.
  // Returns NOT_FOUND if the user does not exist.
  rpc GetUser(GetUserRequest) returns (User);
  
  // CreateUser creates a new user.
  rpc CreateUser(CreateUserRequest) returns (User);
}
```

### Message Documentation

```protobuf
// User represents a user account in the system.
// Users can have different statuses and roles.
message User {
  // Unique user identifier (UUID format).
  string id = 1;
  
  // User's email address (must be unique and valid).
  string email = 2;
  
  // User's display name.
  string name = 3;
  
  // Current account status.
  common.v1.CommonStatus status = 4;
}
```

### Field Documentation

Each field must have:
- Description of what it contains
- Constraints (required/optional, validation rules, format)
- Special notes (e.g., "immutable after creation")

## Linting Rules (Buf)

The following rules are enforced via `buf.yaml`:

### Enabled Rules

- **DEFAULT**: Standard Google style guide rules
- **PACKAGE_DIRECTORY_MATCH**: Package name must match directory structure
- **PACKAGE_VERSION_SUFFIX**: Packages must have version suffix (v1, v2, etc.)

### Enum Rules

- **ENUM_ZERO_VALUE_SUFFIX**: First enum value must end with `_UNSPECIFIED`
- Zero value (0) is reserved and must not represent a valid state

### RPC Rules

- **RPC_REQUEST_STANDARD_NAME**: Request messages must end with `Request`
- **RPC_RESPONSE_STANDARD_NAME**: Response messages must end with `Response`
- Requests and responses should not use `google.protobuf.Empty`

### Service Rules

- **SERVICE_SUFFIX**: Services must end with `Service`

## Directory Structure Rules

Each service must follow this structure:

```
<service>/
├── v1/
│   └── <service>.proto    # Main proto file
└── README.md              # Service documentation
```

Example:
```
user/
├── v1/
│   └── user.proto
└── README.md
```

## Import Patterns

### Prefer Common Types

Use common proto definitions for shared concepts:

```protobuf
import "common/v1/status.proto";
import "common/v1/pagination.proto";

message User {
  string id = 1;
  common.v1.CommonStatus status = 2;  // Use common status
}

message ListUsersRequest {
  common.v1.PageRequest page = 1;  // Use common pagination
}
```

### Import Order

1. Well-known types (google/protobuf/*)
2. Common types (common/v1/*)
3. Other service types

```protobuf
import "google/protobuf/timestamp.proto";
import "common/v1/status.proto";
import "common/v1/pagination.proto";
```

## Breaking vs Non-Breaking Changes

### Non-Breaking Changes (Safe)

- Adding new fields to messages
- Adding new enum values (not at position 0)
- Adding new RPC methods
- Adding new services
- Adding new messages
- Deprecating fields (using `deprecated` option)

Example:
```protobuf
message User {
  string id = 1;
  string email = 2;
  string name = 3;
  string phone = 4;  // NEW: Non-breaking addition
}
```

### Breaking Changes (Dangerous)

- Removing fields
- Removing RPC methods
- Removing services
- Changing field types
- Changing field numbers
- Renaming fields, services, or methods
- Changing message structure

Example of breaking change:
```protobuf
message User {
  string id = 1;
  // BREAKING: Removed email field
  string name = 3;  // BREAKING: Changed from string to int32
}
```

### Deprecation Process

Instead of removing, deprecate:

```protobuf
message User {
  string id = 1;
  string email = 2 [deprecated = true];  // Deprecate, don't remove
  string email_address = 3;  // New field replacing email
}
```

## Field Numbering Strategy

1. **Reserve 1-15**: Most frequently used fields (single-byte encoding)
2. **Reserve 16-2047**: Less frequent fields
3. **Never reuse** field numbers
4. **Reserve deleted** field numbers:
   ```protobuf
   message User {
     reserved 2, 5, 9;  // Previously used field numbers
     reserved "old_field", "deprecated_field";
     
     string id = 1;
     string name = 3;
   }
   ```

## Common Proto Usage Patterns

### Status Fields

Use `common.v1.CommonStatus` for entity status:

```protobuf
import "common/v1/status.proto";

message User {
  string id = 1;
  common.v1.CommonStatus status = 2;
}
```

### Pagination

Use common pagination types:

```protobuf
import "common/v1/pagination.proto";

message ListUsersRequest {
  common.v1.PageRequest page = 1;
}

message ListUsersResponse {
  repeated User users = 1;
  common.v1.PageResponse page = 2;
}
```

### Error Handling

Use common error structures:

```protobuf
import "common/v1/errors.proto";

// Return error details in metadata or use gRPC status
rpc CreateUser(CreateUserRequest) returns (CreateUserResponse);
// On error, return gRPC status with ErrorDetail in metadata
```

### Timestamps

Always use `google.protobuf.Timestamp`:

```protobuf
import "google/protobuf/timestamp.proto";

message User {
  string id = 1;
  google.protobuf.Timestamp created_at = 2;
  google.protobuf.Timestamp updated_at = 3;
}
```

### Money

Use `common.v1.Money` for monetary values:

```protobuf
import "common/v1/types.proto";

message Product {
  string id = 1;
  common.v1.Money price = 2;
}
```

## Testing Proto Changes

Before committing, always run:

```bash
# 1. Lint your changes
make buf-lint

# 2. Check for breaking changes
make buf-breaking

# 3. Validate with protoc
make protoc-validate

# 4. Test code generation
make buf-generate PACKAGE=your-service
```

## Versioning Workflow

### Repository Versions

- Repository uses semantic versioning: v1.0.0, v1.1.0, v2.0.0
- Automated via GitHub Actions on push to main
- Breaking changes require major version bump

### Package Versions

- Proto packages use v1, v2, etc.: `user.v1`, `user.v2`
- When to bump package version:
  - Major breaking changes that can't be handled by deprecation
  - Complete redesign of service
  - Keep v1 alongside v2 during migration period

### When to Bump Package Version (v1 → v2)

Create a new package version when:
1. Making incompatible changes that break existing clients
2. Complete API redesign
3. Cannot maintain backward compatibility

Process:
```bash
# 1. Copy existing version
cp -r user/v1 user/v2

# 2. Update package in user/v2/user.proto
package user.v2;
option go_package = "github.com/andskur/protocols-template/user/v2;userv2";

# 3. Make breaking changes in v2

# 4. Update buf.yaml
# Add: - path: user/v2

# 5. Validate
make buf-lint
```

## Code Generation Expectations

Generated code should:
- Never be committed to this repository (see .gitignore)
- Be generated in consuming microservices
- Use `paths=source_relative` option
- Place generated files alongside .proto files

## CI/CD Workflow

### On Pull Request

CI automatically:
1. Runs `buf lint` on all files
2. Checks for breaking changes vs main branch
3. Validates with protoc
4. Tests code generation with both Buf and protoc

### On Push to Main

CI automatically:
1. Re-runs all validations
2. Generates documentation
3. Creates versioned release with changelog

### Automatic Versioning

- Patch version bumped by default on merge to main
- Manual major/minor bumps via workflow_dispatch
- Tags created automatically: v1.0.0, v1.1.0, etc.

## Common Patterns and Best Practices

### Simple CRUD Service

Keep services minimal in templates - return entities directly:

```protobuf
service UserService {
  // Return entity directly, not wrapped in a response
  rpc GetUser(GetUserRequest) returns (User);
  rpc CreateUser(CreateUserRequest) returns (User);
}
```

### List Operations

When adding list methods, use common pagination:

```protobuf
import "common/v1/pagination.proto";

message ListUsersRequest {
  common.v1.PageRequest page = 1;
  string filter = 2;  // Optional filter
}

message ListUsersResponse {
  repeated User users = 1;
  common.v1.PageResponse page = 2;
}
```

### Soft Delete

For delete operations, consider returning status:

```protobuf
import "common/v1/status.proto";

message DeleteUserRequest {
  string id = 1;
}

message DeleteUserResponse {
  bool success = 1;  // Or return updated user with DELETED status
}
```

### Partial Updates

Use field presence with optional:

```protobuf
message UpdateUserRequest {
  string id = 1;  // Required
  optional string email = 2;  // Only update if present
  optional string name = 3;   // Only update if present
}
```

### Batch Operations

```protobuf
message BatchCreateUsersRequest {
  repeated CreateUserRequest requests = 1;
}

message BatchCreateUsersResponse {
  repeated User users = 1;
  repeated common.v1.ErrorDetail errors = 2;  // Partial failures
}
```

## Quick Reference Checklist

Before committing protocol changes:

- [ ] All messages, fields, enums, and services have documentation comments
- [ ] Package name matches directory structure
- [ ] Service name ends with "Service"
- [ ] Request/Response messages properly named
- [ ] Enum zero value ends with "_UNSPECIFIED"
- [ ] Field names use snake_case
- [ ] No breaking changes (or intentional with version bump)
- [ ] Common types used where applicable
- [ ] Ran `make buf-lint` successfully
- [ ] Ran `make buf-breaking` successfully (on PR)
- [ ] Tested code generation with `make buf-generate`

## Getting Help

- Review existing services in `user/v1/user.proto`
- Check documentation in `docs/`
- Review common types in `common/v1/`
- Run `make help` for available commands
- See [Buf documentation](https://buf.build/docs)
- See [Protocol Buffers style guide](https://protobuf.dev/programming-guides/style/)
