# Common Protocol Definitions

This directory contains reusable protocol buffer definitions that are shared across all services.

## Overview

The `common/v1` package provides standard types for:

- **Status**: Entity lifecycle states (active, inactive, deleted)
- **Pagination**: Both offset-based and cursor-based pagination
- **Errors**: Structured error responses with field-level details
- **Types**: Common data types (UUID, Money, Address)

## Available Types

### Status (status.proto)

Standard entity states for consistent status representation across services.

```protobuf
enum CommonStatus {
  COMMON_STATUS_UNSPECIFIED = 0;  // Default/unknown
  COMMON_STATUS_ACTIVE = 1;       // Active and available
  COMMON_STATUS_INACTIVE = 2;     // Temporarily inactive
  COMMON_STATUS_DELETED = 3;      // Soft-deleted
}
```

**Usage:**

```protobuf
import "common/v1/status.proto";

message User {
  string id = 1;
  string name = 2;
  common.v1.CommonStatus status = 3;
}
```

### Pagination (pagination.proto)

Two pagination patterns for different use cases.

#### Offset-Based Pagination

Traditional page-based pagination:

```protobuf
message PageRequest {
  int32 page = 1;       // Page number (1-indexed)
  int32 page_size = 2;  // Items per page
}

message PageResponse {
  int32 total_count = 1;
  int32 page = 2;
  int32 page_size = 3;
  int32 total_pages = 4;
}
```

**Usage:**

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

**Example:**

```go
req := &userv1.ListUsersRequest{
    Page: &commonv1.PageRequest{
        Page:     1,
        PageSize: 20,
    },
}
```

#### Cursor-Based Pagination

Efficient pagination for large datasets:

```protobuf
message CursorPageRequest {
  string cursor = 1;  // Opaque cursor from previous response
  int32 limit = 2;    // Max items to return
}

message CursorPageResponse {
  string next_cursor = 1;  // Cursor for next page
  bool has_more = 2;       // More results available
}
```

**Usage:**

```protobuf
import "common/v1/pagination.proto";

message ListEventsRequest {
  common.v1.CursorPageRequest page = 1;
}

message ListEventsResponse {
  repeated Event events = 1;
  common.v1.CursorPageResponse page = 2;
}
```

**Example:**

```go
// First page
req := &eventv1.ListEventsRequest{
    Page: &commonv1.CursorPageRequest{
        Cursor: "",  // Empty for first page
        Limit:  50,
    },
}

resp, _ := client.ListEvents(ctx, req)

// Next page
if resp.Page.HasMore {
    nextReq := &eventv1.ListEventsRequest{
        Page: &commonv1.CursorPageRequest{
            Cursor: resp.Page.NextCursor,
            Limit:  50,
        },
    }
}
```

### Errors (errors.proto)

Structured error information for rich error responses.

```protobuf
message ErrorDetail {
  string code = 1;                     // Error code
  string message = 2;                  // Human-readable message
  string field = 3;                    // Field that caused error
  google.protobuf.Any metadata = 4;    // Additional context
}

message ErrorResponse {
  repeated ErrorDetail errors = 1;     // List of errors
  string trace_id = 2;                 // Distributed tracing ID
}
```

**Usage:**

Use in gRPC error details or application-level error responses:

```go
import (
    commonv1 "your-module/protocols/common/v1"
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
)

// Validation errors
errDetails := &commonv1.ErrorResponse{
    Errors: []*commonv1.ErrorDetail{
        {
            Code:    "INVALID_EMAIL",
            Message: "Email format is invalid",
            Field:   "email",
        },
        {
            Code:    "REQUIRED_FIELD",
            Message: "Name is required",
            Field:   "name",
        },
    },
    TraceId: "trace-123",
}

// Return as gRPC error
st := status.New(codes.InvalidArgument, "Validation failed")
st, _ = st.WithDetails(errDetails)
return st.Err()
```

### Types (types.proto)

Common data types used across services.

#### UUID

Binary UUID representation:

```protobuf
message UUID {
  bytes value = 1;  // 16-byte UUID
}
```

**Usage:**

```protobuf
import "common/v1/types.proto";

message User {
  common.v1.UUID uuid = 1;  // Binary UUID
  string id = 2;             // Or string representation
}
```

#### Money

Avoid floating-point precision issues:

```protobuf
message Money {
  int64 amount = 1;     // Amount in smallest unit (cents)
  string currency = 2;  // ISO 4217 code (USD, EUR, etc.)
}
```

**Usage:**

```protobuf
import "common/v1/types.proto";

message Product {
  string id = 1;
  string name = 2;
  common.v1.Money price = 3;
}
```

**Example:**

```go
price := &commonv1.Money{
    Amount:   1050,    // $10.50 in cents
    Currency: "USD",
}
```

#### Address

Standardized address format:

```protobuf
message Address {
  string street = 1;
  string city = 2;
  string state = 3;
  string postal_code = 4;
  string country = 5;  // ISO 3166-1 alpha-2 (US, GB, etc.)
}
```

**Usage:**

```protobuf
import "common/v1/types.proto";

message User {
  string id = 1;
  string name = 2;
  common.v1.Address address = 3;
}
```

**Example:**

```go
address := &commonv1.Address{
    Street:     "123 Main St",
    City:       "San Francisco",
    State:      "CA",
    PostalCode: "94105",
    Country:    "US",
}
```

## When to Use Common Types

### Use Common Types When

- Type is used by 3+ services
- Represents a universal concept (status, pagination, money)
- Should have consistent representation across services
- Benefits from standardization

### Don't Use Common Types When

- Type is service-specific
- Type is experimental or may evolve independently
- Type has unique validation rules per service

## Adding New Common Types

Before adding to `common/v1`:

1. **Verify Reusability**: Used by 3+ services?
2. **Check Stability**: Is the type stable and well-defined?
3. **Review with Team**: Get consensus on design
4. **Document Thoroughly**: Add comprehensive comments

Process:

```bash
# 1. Edit common/v1/types.proto (or create new file)
# 2. Add comprehensive documentation
# 3. Validate
make buf-lint

# 4. Test with consumer services
make buf-generate PACKAGE=common

# 5. Create PR and get team review
```

## Import Patterns

### Importing Common Types

```protobuf
syntax = "proto3";

package user.v1;

import "google/protobuf/timestamp.proto";
import "common/v1/status.proto";
import "common/v1/pagination.proto";
import "common/v1/types.proto";

option go_package = "github.com/andskur/protocols-template/user/v1;userv1";
```

### Import Order

Follow this order for consistency:

1. Well-known types (`google/protobuf/*`)
2. Common types (`common/v1/*`)
3. Other service types

## Code Generation

Generate Go code for common types:

```bash
make buf-generate PACKAGE=common
```

Or using protoc:

```bash
make protoc-generate PACKAGE=common
```

## Examples

### Complete Service Example

```protobuf
syntax = "proto3";

package order.v1;

import "google/protobuf/timestamp.proto";
import "common/v1/status.proto";
import "common/v1/pagination.proto";
import "common/v1/types.proto";

option go_package = "github.com/andskur/protocols-template/order/v1;orderv1";

service OrderService {
  rpc ListOrders(ListOrdersRequest) returns (ListOrdersResponse);
  rpc CreateOrder(CreateOrderRequest) returns (CreateOrderResponse);
}

message Order {
  string id = 1;
  string user_id = 2;
  
  // Use common types
  common.v1.Money total = 3;
  common.v1.CommonStatus status = 4;
  common.v1.Address shipping_address = 5;
  
  google.protobuf.Timestamp created_at = 6;
}

message ListOrdersRequest {
  // Use common pagination
  common.v1.PageRequest page = 1;
  
  // Optional filters
  common.v1.CommonStatus status = 2;
}

message ListOrdersResponse {
  repeated Order orders = 1;
  
  // Use common pagination response
  common.v1.PageResponse page = 2;
}
```

## Best Practices

1. **Always Import from common/v1**: Don't duplicate these types
2. **Use Appropriate Status**: Use CommonStatus for entity lifecycle
3. **Choose Right Pagination**: Offset for UI, cursor for large datasets
4. **Validate Currency Codes**: Use ISO 4217 for Money.currency
5. **Validate Country Codes**: Use ISO 3166-1 for Address.country
6. **Include Trace IDs**: Always set trace_id in ErrorResponse

## Versioning

Common types follow the same versioning as the repository:

- `common/v1`: Current stable version
- Breaking changes require `common/v2` (rare)

## Related Documentation

- [PROTOCOL_DEVELOPMENT.md](../docs/PROTOCOL_DEVELOPMENT.md): Adding new services
- [VERSIONING.md](../docs/VERSIONING.md): Version management
- [User Service README](../user/README.md): Example service using common types
