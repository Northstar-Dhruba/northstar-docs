# ES-006: Git Workflow

## Purpose

This standard defines the required branch, review, merge, and release practices for Northstar.

## Scope

This standard applies to all repositories and contributors in the Northstar platform.

## Mandatory Rules

- The main branch MUST represent the stable release line.
- The develop branch MUST represent the integration line for active work.
- Feature work MUST be completed in short-lived feature branches.
- Changes MUST be reviewed through pull requests before merge.
- Direct commits to protected branches MUST NOT be used.
- Releases MUST be tagged and traceable.

## Recommended Practices

- Feature branches SHOULD be created from develop unless a release or hotfix branch is required.
- Branch names SHOULD be descriptive and aligned to the work being completed.
- Commit messages SHOULD be concise, specific, and structured.
- Hotfixes SHOULD be handled through a dedicated branch and reviewed with the same rigor as regular changes.
- Branch protection SHOULD be enforced for main and develop.

## Examples

### Feature Branch

A feature branch is used for isolated implementation work, for example feature/foundation-package.

### Review

A pull request MUST be created for review before changes are merged into develop.

### Merge

Merges SHOULD be performed through pull requests with the required review and test evidence.

### Release

A release SHOULD be prepared from a reviewed and tested state and tagged consistently.

### Hotfix

A hotfix branch MAY be created from main when a production issue requires urgent correction.

## Common Mistakes

- Merging directly to main or develop without review.
- Leaving feature branches open for long periods without clear purpose.
- Failing to tag releases or hotfixes.
- Using unclear branch naming that obscures intent.

## Review Checklist

- [ ] The branch is named appropriately.
- [ ] The change is reviewed before merge.
- [ ] Tests are present and passing.
- [ ] Documentation is updated when required.
- [ ] Release or hotfix handling is appropriate.

## Related Standards

- ES-005: Code Review Checklist
- ES-007: Documentation Standards

## References

- Repository branch policy
- Release management practice
- Git commit conventions
