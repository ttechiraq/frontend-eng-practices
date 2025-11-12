---
sidebar_position: 3
---

# Git Commit Messages

Writing clear and consistent Git commit messages is crucial for maintaining a clean and understandable project history. Below are the conventions and guidelines your team should follow:

## Commit Message Structure

Each commit message should consist of a header, an optional body, and an optional footer, separated by blank lines:

```
<type>(<scope>): <subject>

<BLANK LINE>

<body>

<BLANK LINE>

<footer>
```

## Header

The header is a single line that summarizes the commit. It has three parts:

- **Type**: Describes the purpose of the commit (e.g., feat, fix, docs, etc.).
- **Scope (optional)**: Specifies the part of the codebase affected (e.g., auth, navbar, api).
- **Subject**: A concise description of the change (50 characters or less).

**Example:**

```
feat(auth): add password reset functionality
```

## Types

Use one of the following types to categorize your commit:

- **`feat`**: A new feature.
- **`fix`**: A bug fix.
- **`docs`**: Documentation changes.
- **`style`**: Code style improvements (e.g., formatting, linting).
- **`refactor`**: Code changes that neither fix a bug nor add a feature.
- **`perf`**: Performance improvements.
- **`test`**: Adding or modifying tests.
- **`chore`**: Maintenance tasks (e.g., dependency updates, build scripts).
- **`ci`**: Changes to CI/CD pipelines.
- **`revert`**: Reverting a previous commit.

## Scope (Optional)

The scope specifies the part of the codebase affected by the commit. It should be short and descriptive. Examples:

- `auth`
- `navbar`
- `api`
- `database`

If the change affects multiple areas or is too broad, you can omit the scope.

## Subject

The subject should:

- Be written in the imperative mood (e.g., "add" instead of "added" or "adds").
- Be concise (50 characters or less).
- Not end with a period.

**Good:**

```
fix(api): handle null values in response
```

**Bad:**

```
fixed the issue with null values in the api response.
```

## Body (Optional)

The body provides additional context about the changes. It should:

- Explain what and why (not how, as the code shows that).
- Be wrapped at 72 characters per line.
- Use bullet points or paragraphs for clarity.

**Example:**

```
feat(auth): add password reset functionality

- Added a new endpoint `/auth/reset-password`.
- Integrated email service to send reset links.
- Updated user schema to include reset token and expiry.
```

## Footer (Optional)

The footer is used for:

- **Breaking changes**: Indicate if the commit introduces breaking changes with `BREAKING CHANGE:` followed by a description.
- **Issue references**: Link to related issues or tickets using `Closes`, `Fixes`, or `Resolves`.

**Example:**

```
fix(api): handle null values in response

Fixes #123
```

**Breaking Change Example:**

```
feat(api): refactor user authentication

BREAKING CHANGE: The `login` endpoint now requires an additional `device_id` parameter.
```

## Additional Tips

- **Be consistent**: Follow the same format across all commits.
- **Use a linter**: Consider using a tool like commitlint to enforce these conventions.
- **Squash commits**: If a feature or fix involves multiple small commits, squash them into a single meaningful commit before merging.
- **Avoid vague messages**: Messages like "update" or "fix bug" are not helpful.

## Example Commit Messages

**Feature:**

```
feat(profile): add profile picture upload functionality

- Added a new endpoint `/profile/upload-picture`.
- Integrated with AWS S3 for image storage.
- Updated user schema to include profile picture URL.
```

**Bug Fix:**

```
fix(navbar): resolve mobile menu overflow issue

- Adjusted CSS for mobile menu items.
- Added overflow handling for smaller screens.

Fixes #45
```

**Documentation:**

```
docs(readme): update installation instructions

- Added steps for setting up the development environment.
- Included troubleshooting tips for common issues.
```

**Refactor:**

```
refactor(auth): simplify token generation logic

- Removed redundant code in token generation.
- Improved error handling for invalid tokens.
```

By following these guidelines, your team will maintain a clean, readable, and professional Git history, making it easier to collaborate and understand the project's evolution.
