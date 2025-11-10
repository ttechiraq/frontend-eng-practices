---
sidebar_position: 4
---

# Pull Requests Guide

## Key Features of the PR Template

- **Title**: The title of a Pull Request (PR) is crucial for quickly conveying the purpose and scope of the changes. Example:

  ```
  [JIRA-123] feat: add user profile page
  ```

- **Description**: Encourages a clear explanation of the changes and links to related issues.
- **Type of Change**: Helps categorize the PR for better tracking and release notes.
- **Checklist**: Ensures adherence to team standards, including Angular style guide compliance, testing, and documentation.
- **Screenshots**: Useful for UI-related changes to visually demonstrate the impact.
- **Additional Notes**: Provides space for any extra context or information for reviewers.

## Creating a PR template in GitHub

Create the `.md` file in the `.github/` folder of the project repo:

```
.github/PULL_REQUEST_TEMPLATE.md
```

## Example of the template

```markdown
## Description

<!-- Provide a brief description of the changes introduced by this PR. -->

**Related Issue:** <!-- Link to the related issue (e.g., `INJAZ-XXXX`) -->

[INJAZ-XXXX](https://ttechteam.atlassian.net/browse/INJAZ-XXXX)

<!-- Replace INJAZ-XXXX with the related issue key  -->

## Type of Change

<!-- Check the relevant box with an `x`. -->

- [ ] Bug fix (non-breaking change which fixes an issue)

- [ ] New feature (non-breaking change which adds functionality)

- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)

- [ ] Documentation update

- [ ] Refactor (code improvements without changing functionality)

- [ ] CI/CD changes

- [ ] Other (please describe):

## Checklist

<!-- Ensure the following tasks are completed before requesting a review. -->

- [ ] My code follows the Angular style guide and coding standards.

- [ ] I have followed the clean architecture.

- [ ] I have performed a self-review of my own code.

- [ ] I have commented my code, particularly in hard-to-understand areas.

- [ ] I have made corresponding changes to the documentation (if applicable).

- [ ] My changes generate no new errors.

- [ ] I have tested my changes in the supported browsers.

- [ ] I have updated the changelog (if applicable).

## Screenshots (if applicable)

<!-- Add screenshots or GIFs to visually demonstrate the changes. -->

## Additional Notes

<!-- Add any additional context or notes for the reviewers. -->
```

## External Links

- [Markdown Guide - Basic Syntax](https://www.markdownguide.org/basic-syntax/)
- [GitHub Docs - Creating a Pull Request Template](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository)


