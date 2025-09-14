# Contributing to Cloudflare Email Routing

Thank you for your interest in contributing to this email forwarding project powered by Cloudflare Email Routing! We appreciate your help in making this project better. This document outlines the guidelines and processes for contributing.

## Project Overview

This is a self-service system for managing email forwarding rules through GitHub Issues, implemented via Cloudflare Email Routing. It uses GitHub Actions and the Cloudflare TypeScript SDK for automation.

## Types of Contributions

We welcome several types of contributions:

- **Bug fixes** - Fix issues with workflows, error handling, or documentation.
- **Feature enhancements** - Improve existing functionality or add new features.
- **Documentation improvements** - Better guides, examples, or troubleshooting.
- **Testing** - Additional test cases or validation scripts.
- **Tooling** - Setup scripts, validation tools, or development helpers.

## Getting Started

To get started with contributing, please follow these steps:

### Development Setup

1.  **Fork the Repository**

2.  **Understanding the System Architecture**

    The system uses a GitOps approach, with `email_forwards.json` as the single source of truth.

    ```mermaid
    graph TB
        A[GitHub Issue] --> B{GitHub Actions Bot};
        B --> |New Request| C{Collision Check};
        C --> |Pass| D[Cloudflare Verification];
        C --> |Fail| E[Auto-close Issue];
        D --> F[User Verifies Email];
        F --> G[Admin Approval];
        G --> H[Update `email_forwards.json`];
        H --> I[Cloudflare API Apply];
        I --> J[Email Forward Active];

        subgraph "Source of Truth"
            K[email_forwards.json];
        end

        subgraph "Infrastructure"
            L[Cloudflare API];
        end

        H --> K;
        K --> I;
        L --> D;
    ```

    #### Core Components
    - **`email_forwards.json`**: The central registry file that defines all forwarding rules.
    - **GitHub Actions Workflows**: A set of automated workflows that manage the entire lifecycle of a request.
    - **Cloudflare API**: Used for email verification and rule management.
    - **Issue Templates**: Structured forms for submitting new forward or removal requests.

3.  **Review Related Documentation**
    - Read the [Product Requirements Document](docs/PRODUCT_REQUIREMENTS.md) for system design.
    - Review the [Admin Guide](docs/ADMIN_GUIDE.md) for setup details.
    - Check the [Testing Guide](docs/TESTING.md) for validation procedures.

4.  **Set Up Test Environment**
    - Follow the setup guide to configure a test repository.
    - Use a test Cloudflare zone/account for development.
    - Create test email addresses for verification.

## Making Changes

### Pull Request Process

1.  **Create Feature Branch**

    ```bash
    git checkout -b feature/your-feature-name
    ```

2.  **Make Your Changes**
    - Follow the coding standards below.
    - Include appropriate tests.
    - Update documentation if needed.

3.  **Test Thoroughly**
    - Use the [Testing Guide](docs/TESTING.md).
    - Test both positive and negative cases.
    - Verify no regressions in existing functionality.

4.  **Submit Pull Request**
    - Use a clear, descriptive title.
    - Include a detailed description of changes.
    - Reference any related issues.
    - Include testing steps for reviewers.

### Specific Guidelines for GitHub Actions Workflows

1.  **Validate YAML Syntax**

    ```bash
    # Test YAML syntax
    python3 -c "import yaml; yaml.safe_load(open('.github/workflows/your-file.yml'))"
    ```

2.  **Test Workflow Logic**
    - Use a test repository to validate changes.
    - Test both happy path and error conditions.
    - Verify proper label transitions and bot comments.

3.  **Follow Security Best Practices**
    - Never expose secrets in logs.
    - Use least privilege for permissions.
    - Validate all user inputs.

### Specific Guidelines for Documentation

1.  **Keep It Current**
    - Update docs when changing functionality.
    - Include new configuration options.
    - Add troubleshooting for new issues.

2.  **Test Instructions**
    - Follow your own documentation step-by-step.
    - Verify commands and examples work.
    - Test from a fresh environment.

## Coding Standards

### GitHub Actions Workflows

- **Naming**: Use descriptive job and step names.
- **Error Handling**: Include failure cases and clear error messages.
- **Comments**: Document complex logic or business rules.
- **Permissions**: Use minimum required permissions.
- **Concurrency**: Use appropriate concurrency groups for shared resources.

```yaml
# Good example
- name: Validate email format and check for collisions
  id: validation
  run: |
    # Validate email format
    if ! echo "$email" | grep -E '^[a-zA-Z0-9._%-=]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'; then
      echo "error=invalid_email" >> $GITHUB_OUTPUT
      exit 0
    fi

    # Check for existing forward
    if grep -q "source: \"$email\"" email_forwards.yml; then
      echo "error=collision" >> $GITHUB_OUTPUT
      exit 0
    fi

    echo "valid=true" >> $GITHUB_OUTPUT
```

### Documentation

- **Structure**: Use clear headings and consistent formatting.
- **Examples**: Include working code examples.
- **Links**: Use relative links for internal documentation.
- **Maintenance**: Keep examples and instructions current.

## Testing

All changes should be thoroughly tested. This project includes a detailed **[Testing Guide](docs/TESTING.MD)** that covers:

- Test scenarios for all major features.
- A manual testing checklist.
- Instructions for troubleshooting common issues.

## Security Considerations

- **Secret Handling**: Never log or expose API tokens or credentials.
- **Input Validation**: Validate all user inputs (emails, usernames, etc.).
- **Authorization**: Verify user permissions before sensitive operations.
- **Audit Trail**: Maintain clear audit logs for all changes.

## Performance Guidelines

- **Workflow Efficiency**: Minimize API calls and processing time.
- **Resource Usage**: Use appropriate runner sizes and timeouts.
- **Registry Management**: Keep email_forwards.yml sorted for merge conflict prevention.
- **Concurrency**: Use concurrency groups to prevent race conditions.

## Getting Help

- **Issues**: Open a GitHub issue for bugs or feature requests.
- **Discussions**: Use GitHub Discussions for questions.
- **Documentation**: Check existing docs first.
- **Examples**: Look at existing workflows for patterns.

## Release Process

Changes are deployed through:

1.  **Pull Request Review** - ADMIN review and approval.
2.  **Testing** - Automated and manual testing in a staging environment.
3.  **Merge** - Merge to the main branch.
4.  **Documentation** - Update version information and changelog.

## Legal & Community

### Code of Conduct

This project and everyone participating in it is governed by the [Code of Conduct](docs/CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code. Please report unacceptable behavior to [github@weispot.com](mailto:github@weispot.com).

### License

By contributing, you agree that your contributions will be licensed under the same MIT License that covers the project.
