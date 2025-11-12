---
sidebar_position: 2
---

# Branch Naming Conventions

Adopting consistent naming patterns helps agile teams stay organized and productive:

```
[prefix]/[issue-ID]-[short-description]
```

- **Prefix**: Describes the type of branch (feature, bugfix, hotfix, release).
- **Issue or Story ID**: Links the branch to the ticket number or ID in your project management system (e.g., Jira or Azure Boards).
- **Short Description**: Briefly describes the purpose or content of the branch.

## Examples

```
feature/PROJ-123-login-improvements
bugfix/PROJ-456-fix-login-error
hotfix/PROJ-789-high-priority-server-fix
release/1.2.0
```

## Prefix Guidelines

- **`feature/`**: For new features or enhancements.
- **`bugfix/`**: For non-critical bug fixes.
- **`hotfix/`**: For urgent, critical fixes in production.
- **`release/`**: For branches preparing a new release.

## Issue or Story ID

- Use the exact ID from your project tracking tool (e.g., PROJ-123 or #456).
- This helps quickly look up the corresponding story or defect.

## Short Description

- Use lower-case, hyphen-separated words (kebab-case).
- Keep it concise but descriptive (e.g., login-error, authentication-service).
- Avoid spaces or underscores.


