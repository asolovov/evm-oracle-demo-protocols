# Protocol Development Guide

This guide covers how to develop and maintain protocol definitions in this repository.

## Table of Contents

- [Adding a New Service](#adding-a-new-service)
- [Adding Fields to Existing Messages](#adding-fields-to-existing-messages)
- [Adding New Enums](#adding-new-enums)
- [Deprecating Fields and Services](#deprecating-fields-and-services)
- [Using Common Protos](#using-common-protos)
- [Message Design Best Practices](#message-design-best-practices)
- [Field Numbering Strategy](#field-numbering-strategy)
- [Testing Proto Changes](#testing-proto-changes)

## Adding a New Service

### Automated Approach (Recommended)

Use the scaffolding script:

```bash
make add-service NAME=product
```

This creates:
- `product/v1/product.proto` with basic service template
- `product/README.md` with documentation template

### Manual Approach

1. **Create directory structure**:
   ```bash
   mkdir -p product/v1
   ```

2. **Create proto file** (`product/v1/product.proto`):
   ```protobuf
   syntax = "proto3";
   
   package product.v1;
   
   import "google/protobuf/timestamp.proto";
   import "common/v1/status.proto";
   
   option go_package = "github.com/andskur/protocols-template/product/v1;productv1";
   
   // ProductService provides operations for managing products.
   service ProductService {
     // GetProduct retrieves a product by ID.
     rpc GetProduct(GetProductRequest) returns (GetProductResponse);
     
     // CreateProduct creates a new product.
     rpc CreateProduct(CreateProductRequest) returns (CreateProductResponse);
   }
   
   // Product represents a product entity.
   message Product {
     // Unique product identifier.
     string id = 1;
     
     // Product name.
     string name = 2;
     
     // Product description.
     string description = 3;
     
     // Product status.
     common.v1.CommonStatus status = 4;
     
     // Timestamp when the product was created.
     google.protobuf.Timestamp created_at = 5;
   }
   
   // GetProductRequest is the request message for GetProduct.
   message GetProductRequest {
     // ID of the product to retrieve.
     string id = 1;
   }
   
   // GetProductResponse is the response message for GetProduct.
   message GetProductResponse {
     // The requested product.
     Product product = 1;
   }
   
   // CreateProductRequest is the request message for CreateProduct.
   message CreateProductRequest {
     // Product name (required).
     string name = 1;
     
     // Product description (optional).
     string description = 2;
   }
   
   // CreateProductResponse is the response message for CreateProduct.
   message CreateProductResponse {
     // The newly created product.
     Product product = 1;
   }
   ```

3. **Create README** (`product/README.md`):
   Document the service, methods, and usage examples.

4. **Validate**:
   ```bash
   make buf-lint
   make protoc-validate PACKAGE=product
   ```

## Adding Fields to Existing Messages

### Non-Breaking Addition

Adding new fields is safe and non-breaking:

```protobuf
message User {
  string id = 1;
  string email = 2;
  string name = 3;
  
  // NEW: Adding phone is non-breaking
  string phone = 4;
  
  // NEW: Adding avatar_url is non-breaking
  string avatar_url = 5;
}
```

**Best Practices:**
- Always use the next available field number
- Document new fields with comments
- Consider using `optional` for truly optional fields
- Test with older clients to ensure compatibility

### Using Optional Fields

For partial updates:

```protobuf
message UpdateUserRequest {
  // User ID (required).
  string id = 1;
  
  // Email to update (optional, only updated if present).
  optional string email = 2;
  
  // Name to update (optional, only updated if present).
  optional string name = 3;
}
```

### Nested Messages

Add nested messages for complex structures:

```protobuf
message User {
  string id = 1;
  string name = 2;
  
  // NEW: Adding nested message
  UserProfile profile = 3;
}

message UserProfile {
  string bio = 1;
  string avatar_url = 2;
  string timezone = 3;
}
```

## Adding New Enums

### Creating a New Enum

Always include `_UNSPECIFIED` as value 0:

```protobuf
// UserRole represents different user access levels.
enum UserRole {
  // Default unspecified value (required).
  USER_ROLE_UNSPECIFIED = 0;
  
  // Regular user with standard permissions.
  USER_ROLE_USER = 1;
  
  // Administrator with full permissions.
  USER_ROLE_ADMIN = 2;
  
  // Moderator with limited admin permissions.
  USER_ROLE_MODERATOR = 3;
}
```

### Adding Enum Values

Adding new enum values is non-breaking (except at position 0):

```protobuf
enum UserRole {
  USER_ROLE_UNSPECIFIED = 0;
  USER_ROLE_USER = 1;
  USER_ROLE_ADMIN = 2;
  USER_ROLE_MODERATOR = 3;
  
  // NEW: Adding new role is safe
  USER_ROLE_SUPER_ADMIN = 4;
}
```

### Enum Naming Convention

- Enum type: PascalCase (e.g., `UserRole`)
- Enum values: SCREAMING_SNAKE_CASE with prefix (e.g., `USER_ROLE_ADMIN`)
- Prefix format: `<ENUM_NAME>_<VALUE>`

## Deprecating Fields and Services

### Deprecating Fields

Use the `deprecated` option instead of removing:

```protobuf
message User {
  string id = 1;
  
  // Deprecated: Use email_address instead.
  string email = 2 [deprecated = true];
  
  // Replacement for email field.
  string email_address = 3;
  
  string name = 4;
}
```

### Reserving Deleted Fields

If you must remove a field, reserve its number:

```protobuf
message User {
  reserved 2, 5, 9;  // Previously used field numbers
  reserved "old_field", "removed_field";
  
  string id = 1;
  string name = 3;
  string email = 4;
}
```

### Deprecating RPC Methods

```protobuf
service UserService {
  // Deprecated: Use GetUserV2 instead.
  rpc GetUser(GetUserRequest) returns (GetUserResponse) {
    option deprecated = true;
  };
  
  // Replacement for GetUser method.
  rpc GetUserV2(GetUserV2Request) returns (GetUserV2Response);
}
```

## Using Common Protos

### Available Common Types

Import and use common protos for consistency:

```protobuf
import "common/v1/status.proto";
import "common/v1/pagination.proto";
import "common/v1/errors.proto";
import "common/v1/types.proto";

message User {
  string id = 1;
  string name = 2;
  
  // Use common status enum
  common.v1.CommonStatus status = 3;
}

message ListUsersRequest {
  // Use common pagination
  common.v1.PageRequest page = 1;
}

message ListUsersResponse {
  repeated User users = 1;
  
  // Use common pagination response
  common.v1.PageResponse page = 2;
}
```

### When to Add to Common Protos

Add to `common/v1` when:
- Type is used by 3+ services
- Represents a universal concept (status, pagination, etc.)
- Should have consistent representation across services

Don't add to common:
- Service-specific types
- Experimental or unstable definitions
- Types that may evolve independently

## Message Design Best Practices

### Use Descriptive Names

```protobuf
// Good
message CreateUserRequest {
  string email_address = 1;
  string full_name = 2;
}

// Bad
message CreateUserReq {
  string email = 1;
  string name = 2;
}
```

### Group Related Fields

```protobuf
message User {
  // Identity fields
  string id = 1;
  string email = 2;
  
  // Profile fields
  string name = 3;
  string bio = 4;
  string avatar_url = 5;
  
  // Metadata fields
  google.protobuf.Timestamp created_at = 10;
  google.protobuf.Timestamp updated_at = 11;
  common.v1.CommonStatus status = 12;
}
```

### Use Well-Known Types

Prefer well-known types from `google/protobuf`:

```protobuf
import "google/protobuf/timestamp.proto";
import "google/protobuf/duration.proto";
import "google/protobuf/empty.proto";

message Event {
  string id = 1;
  
  // Use Timestamp, not int64 or string
  google.protobuf.Timestamp occurred_at = 2;
  
  // Use Duration, not int32 or string
  google.protobuf.Duration duration = 3;
}
```

### Avoid Primitive Obsession

```protobuf
// Good: Use message types for structure
message Money {
  int64 amount = 1;
  string currency = 2;
}

message Product {
  string id = 1;
  Money price = 2;
}

// Bad: Primitive types lose structure
message Product {
  string id = 1;
  int64 price_amount = 2;
  string price_currency = 3;
}
```

## Field Numbering Strategy

### Reserve 1-15 for Frequent Fields

Fields 1-15 use single-byte encoding:

```protobuf
message User {
  // Most frequently accessed fields (1-15)
  string id = 1;
  string email = 2;
  string name = 3;
  common.v1.CommonStatus status = 4;
  
  // Less frequent fields (16+)
  string bio = 16;
  string website = 17;
  string location = 18;
}
```

### Leave Gaps for Future Use

```protobuf
message User {
  // Identity (1-5)
  string id = 1;
  string email = 2;
  // Reserved for future identity fields: 3-5
  
  // Profile (6-10)
  string name = 6;
  string bio = 7;
  // Reserved for future profile fields: 8-10
  
  // Metadata (11-15)
  google.protobuf.Timestamp created_at = 11;
  common.v1.CommonStatus status = 12;
  // Reserved for future metadata fields: 13-15
}
```

### Never Reuse Field Numbers

```protobuf
message User {
  reserved 2, 5, 9;  // Previously used, never reuse
  reserved "old_email", "removed_field";
  
  string id = 1;
  string email_address = 3;  // New field, new number
  string name = 4;
}
```

## Testing Proto Changes

### Local Validation

Before committing:

```bash
# 1. Lint with Buf
make buf-lint

# 2. Check for breaking changes
make buf-breaking

# 3. Validate with protoc
make protoc-validate

# 4. Test code generation
make buf-generate PACKAGE=your-service

# 5. Clean up generated files
make clean
```

### Testing in Microservice

1. Update protocols in your microservice:
   ```bash
   cd your-microservice
   make proto-update
   ```

2. Generate code:
   ```bash
   make proto-generate PROTO_PACKAGE=your-service
   ```

3. Build and test:
   ```bash
   go build ./...
   go test ./...
   ```

### CI Validation

Pull requests automatically run:
- Buf lint
- Breaking change detection
- Protoc validation
- Code generation tests

Wait for CI to pass before merging.

## Common Patterns

### Standard CRUD Service

```protobuf
service ProductService {
  rpc GetProduct(GetProductRequest) returns (GetProductResponse);
  rpc ListProducts(ListProductsRequest) returns (ListProductsResponse);
  rpc CreateProduct(CreateProductRequest) returns (CreateProductResponse);
  rpc UpdateProduct(UpdateProductRequest) returns (UpdateProductResponse);
  rpc DeleteProduct(DeleteProductRequest) returns (DeleteProductResponse);
}
```

### Batch Operations

```protobuf
message BatchCreateProductsRequest {
  repeated CreateProductRequest requests = 1;
}

message BatchCreateProductsResponse {
  repeated Product products = 1;
  repeated common.v1.ErrorDetail errors = 2;
}
```

### Streaming

```protobuf
service EventService {
  // Server streaming
  rpc WatchEvents(WatchEventsRequest) returns (stream Event);
  
  // Client streaming
  rpc UploadData(stream DataChunk) returns (UploadResponse);
  
  // Bidirectional streaming
  rpc Chat(stream ChatMessage) returns (stream ChatMessage);
}
```

## Next Steps

- Read [VERSIONING.md](VERSIONING.md) for version management
- See [BUF_GUIDE.md](BUF_GUIDE.md) for Buf tooling
- Check [INTEGRATION.md](INTEGRATION.md) for microservice integration
