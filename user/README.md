# User Service Protocol

This directory contains the protocol buffer definitions for the User Service.

## Overview

The User Service provides a minimal example of a gRPC service for managing user accounts. This is a template - extend it with additional methods as needed for your use case.

## Service Definition

```protobuf
service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc CreateUser(CreateUserRequest) returns (User);
}
```

## Message Definitions

### User

Represents a user account in the system.

```protobuf
message User {
  common.v1.UUID id = 1;                   // Unique user identifier (UUID)
  string email = 2;                        // User's email address
  string name = 3;                         // User's display name
  common.v1.CommonStatus status = 4;       // Account status
  google.protobuf.Timestamp created_at = 5; // Creation timestamp
  google.protobuf.Timestamp updated_at = 6; // Last update timestamp
}
```

**Field Details:**

- `id`: Unique identifier (UUID, 16 bytes)
- `email`: Must be unique across all users
- `name`: Display name shown in UI
- `status`: Uses common status enum (ACTIVE, INACTIVE, DELETED)
- `created_at`: Immutable creation timestamp
- `updated_at`: Updated on every modification

## Available Methods

### GetUser

Retrieves a single user by ID.

**Request:**

```protobuf
message GetUserRequest {
  common.v1.UUID id = 1;  // User ID to retrieve (UUID)
}
```

**Response:**

Returns the created `User` directly.

**Example:**

```go
user, err := client.CreateUser(ctx, &userv1.CreateUserRequest{
    Email: "[email protected]",
    Name:  "John Doe",
})
if err != nil {
    // Handle error (NOT_FOUND, INTERNAL, etc.)
}

fmt.Printf("User: %s (%s)\n", user.Name, user.Email)
```

**Errors:**

- `NOT_FOUND`: User with given ID does not exist
- `INVALID_ARGUMENT`: ID is empty or malformed
- `INTERNAL`: Server error

### CreateUser

Creates a new user account.

**Request:**

```protobuf
message CreateUserRequest {
  string email = 1;  // User's email (required)
  string name = 2;   // User's name (required)
}
```

**Response:**

Returns the created `User` directly.

**Example:**

```go
user, err := client.CreateUser(ctx, &userv1.CreateUserRequest{
    Email: "[email protected]",
    Name:  "John Doe",
})

if err != nil {
    // Handle error (ALREADY_EXISTS, INVALID_ARGUMENT, etc.)
}

fmt.Printf("Created user: %s (ID: %s)\n", user.Name, user.Id)
```

**Validation:**

- `email`: Required, must be valid email format, must be unique
- `name`: Required, min 1 character

**Errors:**

- `ALREADY_EXISTS`: Email already registered
- `INVALID_ARGUMENT`: Missing or invalid email/name
- `INTERNAL`: Server error

## Extending This Template

This service is intentionally minimal. Add more methods as needed:

```protobuf
service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc CreateUser(CreateUserRequest) returns (User);
  
  // Add your own methods:
  // rpc UpdateUser(UpdateUserRequest) returns (User);
  // rpc DeleteUser(DeleteUserRequest) returns (DeleteUserResponse);
  // rpc ListUsers(ListUsersRequest) returns (ListUsersResponse);
}
```

For list operations, consider using common pagination types:

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

## Integration

### Generating Go Code

```bash
# In your microservice repository
make proto-generate PROTO_PACKAGE=user
```

This generates:
- `protocols/user/v1/user.pb.go`: Message definitions
- `protocols/user/v1/user_grpc.pb.go`: Service client and server stubs

### Importing in Your Code

```go
import (
    userv1 "your-module/protocols/user/v1"
    commonv1 "your-module/protocols/common/v1"
)
```

### Implementing the Server

```go
package server

import (
    "context"
    "time"
    
    userv1 "your-module/protocols/user/v1"
    commonv1 "your-module/protocols/common/v1"
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
    "google.golang.org/protobuf/types/known/timestamppb"
)

type UserServer struct {
    userv1.UnimplementedUserServiceServer
    // Add your dependencies (DB, cache, etc.)
}

func (s *UserServer) GetUser(ctx context.Context, req *userv1.GetUserRequest) (*userv1.User, error) {
    if req.Id == "" {
        return nil, status.Error(codes.InvalidArgument, "user ID is required")
    }
    
    // Fetch user from database
    user, err := s.fetchUserFromDB(ctx, req.Id)
    if err != nil {
        return nil, status.Error(codes.NotFound, "user not found")
    }
    
    return user, nil
}

func (s *UserServer) CreateUser(ctx context.Context, req *userv1.CreateUserRequest) (*userv1.User, error) {
    // Validate
    if req.Email == "" {
        return nil, status.Error(codes.InvalidArgument, "email is required")
    }
    if req.Name == "" {
        return nil, status.Error(codes.InvalidArgument, "name is required")
    }
    
    // Create user
    user := &userv1.User{
        Id:        generateID(),
        Email:     req.Email,
        Name:      req.Name,
        Status:    commonv1.CommonStatus_COMMON_STATUS_ACTIVE,
        CreatedAt: timestamppb.Now(),
        UpdatedAt: timestamppb.Now(),
    }
    
    // Save to database
    if err := s.saveUserToDB(ctx, user); err != nil {
        if isDuplicateEmail(err) {
            return nil, status.Error(codes.AlreadyExists, "email already registered")
        }
        return nil, status.Error(codes.Internal, "failed to create user")
    }
    
    return user, nil
}
```

### Creating a Client

```go
package main

import (
    "context"
    "log"
    
    userv1 "your-module/protocols/user/v1"
    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials/insecure"
)

func main() {
    // Connect to server
    conn, err := grpc.Dial(
        "localhost:50051",
        grpc.WithTransportCredentials(insecure.NewCredentials()),
    )
    if err != nil {
        log.Fatal(err)
    }
    defer conn.Close()
    
    // Create client
    client := userv1.NewUserServiceClient(conn)
    
    // Call methods
    ctx := context.Background()
    
    // Create user
    user, err := client.CreateUser(ctx, &userv1.CreateUserRequest{
        Email: "[email protected]",
        Name:  "Alice Smith",
    })
    if err != nil {
        log.Fatal(err)
    }
    
    log.Printf("Created user: %s (ID: %s)", user.Name, user.Id)
    
    // Get user
    retrievedUser, err := client.GetUser(ctx, &userv1.GetUserRequest{
        Id: user.Id,
    })
    if err != nil {
        log.Fatal(err)
    }
    
    log.Printf("Retrieved user: %s", retrievedUser.Name)
}
```

## Best Practices

1. **Always validate inputs** on the server side
2. **Use appropriate gRPC status codes** for errors
3. **Include trace IDs** for distributed tracing
4. **Return entities directly** instead of wrapper responses when possible
5. **Use common types** for consistency (status, pagination, etc.)
6. **Add comprehensive field documentation** to help API consumers
7. **Add metrics and logging** for observability

## Versioning

This is version 1 (`user.v1`) of the User Service protocol.

For breaking changes, create `user/v2/` following the [versioning guide](../docs/VERSIONING.md).

## Related Documentation

- [Common Types](../common/README.md): Shared types used by this service
- [Protocol Development](../docs/PROTOCOL_DEVELOPMENT.md): How to modify protocols
- [Integration Guide](../docs/INTEGRATION.md): Using protocols in microservices
