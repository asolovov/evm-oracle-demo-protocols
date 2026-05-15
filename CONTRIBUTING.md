# Contributing to protocols-template

Thank you for your interest in contributing! This guide will help you understand how to contribute to this repository.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How Can I Contribute?](#how-can-i-contribute)
- [Development Process](#development-process)
- [Pull Request Process](#pull-request-process)
- [Style Guidelines](#style-guidelines)
- [Commit Messages](#commit-messages)

## Code of Conduct

This project follows a simple code of conduct:

- Be respectful and inclusive
- Provide constructive feedback
- Focus on what is best for the community
- Show empathy towards other contributors

## How Can I Contribute?

### Reporting Bugs

Before submitting a bug report:
- Check existing issues to avoid duplicates
- Use the latest version of the repository

When submitting a bug report, include:
- Clear title and description
- Steps to reproduce
- Expected vs actual behavior
- Proto file examples (if applicable)
- Error messages

### Suggesting Enhancements

Enhancement suggestions are welcome! Please:
- Use a clear and descriptive title
- Provide detailed description of the enhancement
- Explain why this would be useful
- Provide examples if possible

### Adding New Services

To add a new example service:

1. Use the scaffolding script:
   ```bash
   make add-service NAME=yourservice
   ```

2. Customize the generated files
3. Add comprehensive documentation
4. Ensure it follows all style guidelines
5. Submit a pull request

### Improving Documentation

Documentation improvements are always welcome:
- Fix typos or unclear explanations
- Add examples
- Improve organization
- Add missing information

## Development Process

### 1. Fork and Clone

```bash
# Fork the repository on GitHub
# Then clone your fork
git clone https://github.com/yourusername/protocols-template.git
cd protocols-template
```

### 2. Create a Branch

```bash
git checkout -b feature/your-feature-name
# or
git checkout -b fix/your-bug-fix
```

Branch naming:
- `feature/` - New features or enhancements
- `fix/` - Bug fixes
- `docs/` - Documentation changes
- `refactor/` - Code refactoring
- `test/` - Test improvements

### 3. Make Changes

#### Adding/Modifying Proto Files

1. Edit proto files following [style guidelines](#proto-style-guidelines)
2. Validate changes:
   ```bash
   make buf-lint
   make buf-breaking
   make protoc-validate
   ```
3. Test code generation:
   ```bash
   make buf-generate-all
   make clean  # Don't commit generated files
   ```

#### Updating Documentation

1. Edit relevant markdown files
2. Ensure examples are accurate
3. Check for broken links
4. Verify formatting

#### Adding Scripts

1. Follow existing script patterns
2. Add comprehensive comments
3. Include usage examples
4. Test on both macOS and Linux if possible
5. Make executable: `chmod +x scripts/your-script.sh`

### 4. Test Your Changes

```bash
# Lint
make buf-lint

# Check breaking changes
make buf-breaking

# Validate with protoc
make protoc-validate

# Test code generation
make buf-generate-all
make protoc-generate-all

# Clean up
make clean
```

### 5. Commit Your Changes

Follow the [commit message guidelines](#commit-messages):

```bash
git add .
git commit -m "feat: add product service example"
```

### 6. Push and Create Pull Request

```bash
git push origin feature/your-feature-name
```

Then create a pull request on GitHub.

## Pull Request Process

### Before Submitting

- [ ] Code follows style guidelines
- [ ] All validation passes (`make validate`)
- [ ] Documentation is updated
- [ ] Commit messages follow conventions
- [ ] No generated files committed (*.pb.go)
- [ ] Branch is up to date with main

### PR Description

Include:
- Clear description of changes
- Motivation and context
- Related issue numbers (if applicable)
- Screenshots (for documentation changes)
- Breaking changes (if any)

### Review Process

1. Automated CI checks must pass
2. At least one maintainer approval required
3. All review comments addressed
4. No merge conflicts

### After Merge

- Delete your feature branch
- Update your fork's main branch
- Celebrate! 🎉

## Style Guidelines

### Proto Style Guidelines

#### Package Naming

```protobuf
// ✅ Good
package user.v1;
package product.v2;
package common.v1;

// ❌ Bad
package user;
package UserV1;
```

#### Service Naming

```protobuf
// ✅ Good
service UserService {
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
}

// ❌ Bad
service User {
  rpc Get(GetUserReq) returns (GetUserResp);
}
```

#### Message Naming

```protobuf
// ✅ Good
message User { }
message GetUserRequest { }
message ListUsersResponse { }

// ❌ Bad
message user { }
message GetUserReq { }
message UserListResp { }
```

#### Field Naming

```protobuf
// ✅ Good
message User {
  string user_id = 1;
  string email_address = 2;
  google.protobuf.Timestamp created_at = 3;
}

// ❌ Bad
message User {
  string userId = 1;
  string emailAddr = 2;
  int64 createdAt = 3;
}
```

#### Enum Naming

```protobuf
// ✅ Good
enum UserStatus {
  USER_STATUS_UNSPECIFIED = 0;
  USER_STATUS_ACTIVE = 1;
  USER_STATUS_INACTIVE = 2;
}

// ❌ Bad
enum UserStatus {
  ACTIVE = 0;
  INACTIVE = 1;
}
```

#### Documentation

All public elements must have documentation:

```protobuf
// ✅ Good
// UserService provides operations for managing user accounts.
service UserService {
  // GetUser retrieves a user by their unique identifier.
  // Returns NOT_FOUND if the user does not exist.
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
}

// User represents a user account in the system.
message User {
  // Unique user identifier (UUID format).
  string id = 1;
  
  // User's email address (must be unique and valid).
  string email = 2;
}

// ❌ Bad
service UserService {
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
}

message User {
  string id = 1;
  string email = 2;
}
```

### Markdown Style Guidelines

- Use ATX-style headers (`#`, `##`, not underlines)
- Use fenced code blocks with language tags
- Include blank lines around headers and code blocks
- Use relative links for internal references
- Keep lines under 120 characters when possible
- Use tables for comparisons
- Include examples for complex concepts

### Script Style Guidelines

- Include shebang: `#!/bin/bash`
- Use `set -e` for error handling
- Add comprehensive comments
- Include usage examples
- Validate inputs
- Provide clear error messages

## Commit Messages

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Formatting, missing semicolons, etc.
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance tasks

### Scope

Optional, indicates what is being changed:
- `common`: Common proto types
- `user`: User service
- `scripts`: Automation scripts
- `docs`: Documentation
- `ci`: CI/CD workflows

### Examples

```bash
# Feature
git commit -m "feat(common): add Money type for monetary values"

# Bug fix
git commit -m "fix(user): correct email validation in CreateUser"

# Documentation
git commit -m "docs(integration): add git subtree examples"

# Breaking change
git commit -m "feat(user)!: remove deprecated email field

BREAKING CHANGE: The email field has been removed. Use email_address instead."
```

### Guidelines

- Use imperative mood ("add" not "added")
- Don't capitalize first letter
- No period at the end
- Limit subject line to 72 characters
- Separate subject from body with blank line
- Use body to explain what and why vs how
- Reference issues: "Closes #123"

## Breaking Changes

Breaking changes require special handling:

1. **Document clearly** in PR description
2. **Use `!` in commit** or `BREAKING CHANGE:` in footer
3. **Update VERSIONING.md** if needed
4. **Consider alternatives**:
   - Can you deprecate instead?
   - Can you maintain backward compatibility?
   - Should this be a new package version (v2)?

5. **Get team approval** before merging

## Review Guidelines

When reviewing pull requests:

### Check

- [ ] Follows style guidelines
- [ ] Includes documentation
- [ ] No breaking changes (or properly documented)
- [ ] CI passes
- [ ] Commit messages follow conventions
- [ ] No generated files committed

### Provide

- Constructive feedback
- Specific suggestions
- Praise for good work
- Clear explanations for requested changes

### Approve when

- All checks pass
- Code quality is high
- Documentation is complete
- No outstanding concerns

## Questions?

- Check [README.md](README.md) first
- Review [docs/](docs/) for detailed guides
- Open an issue for questions
- Reach out to maintainers

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
