# Versioning Guide

This guide explains the versioning strategy for both the repository and proto packages.

## Table of Contents

- [Repository Versioning](#repository-versioning)
- [Package Versioning](#package-versioning)
- [Breaking vs Non-Breaking Changes](#breaking-vs-non-breaking-changes)
- [Migration Strategy](#migration-strategy)
- [Release Process](#release-process)

## Repository Versioning

### Semantic Versioning

This repository uses [Semantic Versioning 2.0.0](https://semver.org/):

```
vMAJOR.MINOR.PATCH
```

- **MAJOR**: Breaking changes that require client updates
- **MINOR**: New features, backward-compatible additions
- **PATCH**: Bug fixes, documentation, non-breaking improvements

### Examples

- `v1.0.0` → `v1.0.1`: Fixed documentation, non-breaking
- `v1.0.1` → `v1.1.0`: Added new service, backward-compatible
- `v1.1.0` → `v2.0.0`: Removed field, breaking change

### Automated Versioning

Versions are automatically managed by GitHub Actions:

- **On merge to main**: Patch version bump (v1.0.0 → v1.0.1)
- **Manual trigger**: Choose major, minor, or patch bump

#### Automatic Patch Bump

Every merge to main creates a new patch version:

```bash
# After PR merge
v1.0.0 → v1.0.1 → v1.0.2 → ...
```

#### Manual Version Bump

For minor or major versions:

1. Go to Actions → Release workflow
2. Click "Run workflow"
3. Select version bump type (major/minor/patch)
4. Click "Run workflow"

### Version Tags

Git tags follow this format:

```bash
v1.0.0
v1.1.0
v2.0.0
```

Tags are automatically created and pushed by CI.

## Package Versioning

### Proto Package Versions

Proto packages use version suffixes (v1, v2, v3):

```protobuf
package user.v1;
package product.v2;
package common.v1;
```

### When to Bump Package Version

Create a new package version (v1 → v2) when:

1. **Breaking changes** that can't be handled with deprecation
2. **Complete API redesign**
3. **Cannot maintain backward compatibility**

### Package Version Process

#### Step 1: Copy Existing Version

```bash
cp -r user/v1 user/v2
```

#### Step 2: Update Package Declaration

Edit `user/v2/user.proto`:

```protobuf
syntax = "proto3";

package user.v2;  // Changed from user.v1

option go_package = "github.com/andskur/protocols-template/user/v2;userv2";
```

#### Step 3: Make Breaking Changes

Now you can make breaking changes in v2:

```protobuf
// user/v2/user.proto
message User {
  string id = 1;
  // BREAKING: Removed email field
  // BREAKING: Changed name to full_name
  string full_name = 2;
  // NEW: Split into structured name
  StructuredName structured_name = 3;
}

message StructuredName {
  string first_name = 1;
  string last_name = 2;
}
```

#### Step 4: Update buf.yaml

Add new module to `buf.yaml`:

```yaml
version: v2
modules:
  - path: common/v1
  - path: user/v1
  - path: user/v2  # Add new version
```

#### Step 5: Validate

```bash
make buf-lint
make buf-generate PACKAGE=user
```

#### Step 6: Maintain Both Versions

Keep v1 and v2 in parallel during migration:

```
user/
├── v1/
│   └── user.proto  # Keep for existing clients
├── v2/
│   └── user.proto  # New version
└── README.md       # Document both versions
```

## Breaking vs Non-Breaking Changes

### Non-Breaking Changes

These changes are **safe** and only require a patch/minor version bump:

#### Adding Fields

```protobuf
message User {
  string id = 1;
  string email = 2;
  string name = 3;
  string phone = 4;  // ✅ Safe to add
}
```

#### Adding RPC Methods

```protobuf
service UserService {
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
  rpc UpdateUser(UpdateUserRequest) returns (UpdateUserResponse);  // ✅ Safe to add
}
```

#### Adding Services

```protobuf
// ✅ Safe to add new service
service ProductService {
  rpc GetProduct(GetProductRequest) returns (GetProductResponse);
}
```

#### Adding Messages

```protobuf
// ✅ Safe to add new message type
message UserProfile {
  string bio = 1;
  string avatar_url = 2;
}
```

#### Adding Enum Values

```protobuf
enum UserRole {
  USER_ROLE_UNSPECIFIED = 0;
  USER_ROLE_USER = 1;
  USER_ROLE_ADMIN = 2;
  USER_ROLE_MODERATOR = 3;  // ✅ Safe to add (not at position 0)
}
```

#### Deprecating Fields

```protobuf
message User {
  string id = 1;
  string email = 2 [deprecated = true];  // ✅ Safe to deprecate
  string email_address = 3;
}
```

### Breaking Changes

These changes **require a major version bump** or new package version:

#### Removing Fields

```protobuf
message User {
  string id = 1;
  // ❌ BREAKING: Removed email field
  string name = 3;
}
```

#### Changing Field Types

```protobuf
message User {
  string id = 1;
  int32 email = 2;  // ❌ BREAKING: Changed from string to int32
}
```

#### Changing Field Numbers

```protobuf
message User {
  string id = 1;
  string email = 3;  // ❌ BREAKING: Changed from 2 to 3
}
```

#### Renaming Fields

```protobuf
message User {
  string id = 1;
  string email_address = 2;  // ❌ BREAKING: Renamed from email
}
```

#### Removing RPC Methods

```protobuf
service UserService {
  // ❌ BREAKING: Removed GetUser method
  rpc UpdateUser(UpdateUserRequest) returns (UpdateUserResponse);
}
```

#### Changing Message Structure

```protobuf
message GetUserRequest {
  repeated string ids = 1;  // ❌ BREAKING: Changed from string id
}
```

#### Changing Enum Values

```protobuf
enum UserRole {
  USER_ROLE_ADMIN = 0;  // ❌ BREAKING: Changed zero value
  USER_ROLE_USER = 1;
}
```

## Migration Strategy

### For Non-Breaking Changes

1. Make changes in existing package (v1)
2. Run `make buf-lint`
3. Run `make buf-breaking` (should pass)
4. Commit and merge
5. Automatic patch/minor version bump

### For Breaking Changes

Choose one of these strategies:

#### Strategy 1: Deprecation (Preferred)

Add new fields, deprecate old ones:

```protobuf
message User {
  string id = 1;
  
  // Deprecated: Use email_address instead.
  string email = 2 [deprecated = true];
  
  // New field replacing email.
  string email_address = 3;
}
```

**Timeline:**
1. Release with deprecated field
2. Migrate clients to new field
3. In next major version, remove deprecated field

#### Strategy 2: New Package Version

Create v2 package:

```bash
# Copy and update
cp -r user/v1 user/v2
# Update package name in user/v2/user.proto
# Make breaking changes
# Update buf.yaml
```

**Timeline:**
1. Release v2 alongside v1
2. Migrate clients incrementally
3. Deprecate v1 after migration period
4. Eventually remove v1 in repository v3.0.0

#### Strategy 3: Major Version Bump

For repository-wide breaking changes:

1. Create new package versions for all affected services
2. Trigger manual major version bump
3. Coordinate client migrations

## Release Process

### Automatic Release (Patch)

On every merge to main:

1. CI runs all validations
2. Determines next patch version (v1.0.0 → v1.0.1)
3. Generates changelog from commits
4. Creates git tag
5. Creates GitHub Release

### Manual Release (Minor/Major)

For intentional version bumps:

1. Go to GitHub Actions
2. Select "Release" workflow
3. Click "Run workflow"
4. Choose version bump type:
   - **patch**: Bug fixes, docs (v1.0.0 → v1.0.1)
   - **minor**: New features (v1.0.0 → v1.1.0)
   - **major**: Breaking changes (v1.0.0 → v2.0.0)
5. Click "Run workflow"

### Changelog Format

Changelogs are automatically generated from commit messages:

```markdown
## Changes

- Add phone field to User message (abc123)
- Update documentation for pagination (def456)
- Fix linting errors in product service (ghi789)
```

**Best Practice:** Use descriptive commit messages:

```bash
# Good
git commit -m "Add phone field to User message for contact info"

# Bad
git commit -m "Update user.proto"
```

## Version Compatibility Matrix

### Repository Versions

| Repository Version | Proto Package Versions | Breaking Changes |
|-------------------|------------------------|------------------|
| v1.0.0 - v1.x.x   | user.v1, common.v1    | None             |
| v2.0.0 - v2.x.x   | user.v2, common.v1    | User service     |
| v3.0.0+           | user.v2, common.v2    | Common types     |

### Migration Timeline Example

```
v1.0.0: Initial release (user.v1)
  ↓
v1.1.0: Add new fields (non-breaking)
  ↓
v1.2.0: Deprecate old fields
  ↓
v2.0.0: Release user.v2 alongside user.v1
  ↓
v2.1.0: Clients migrate to user.v2
  ↓
v3.0.0: Remove user.v1 (breaking for unmigrated clients)
```

## Best Practices

1. **Avoid breaking changes** when possible
2. **Use deprecation** instead of removal
3. **Communicate** breaking changes to consumers
4. **Maintain compatibility** for at least 2 minor versions
5. **Test migrations** before major version bumps
6. **Document changes** in commit messages and changelogs
7. **Coordinate with teams** when planning breaking changes

## Checking for Breaking Changes

### Local Check

```bash
make buf-breaking
```

This compares against the main branch or latest tag.

### CI Check

Pull requests automatically check for breaking changes:

- ✅ Pass: No breaking changes detected
- ❌ Fail: Breaking changes detected (requires major version or new package)

### Override Breaking Change Detection

If you intend to make a breaking change:

1. Create new package version (v1 → v2), OR
2. Plan for major repository version bump (v1.x → v2.0)
3. Document the breaking change in PR description
4. Communicate with consumers

## Next Steps

- Read [PROTOCOL_DEVELOPMENT.md](PROTOCOL_DEVELOPMENT.md) for development guidelines
- See [BUF_GUIDE.md](BUF_GUIDE.md) for Buf tooling
- Check [INTEGRATION.md](INTEGRATION.md) for microservice integration
