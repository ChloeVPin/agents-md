## Summary

Describe the behavior, documentation, or validation change in concrete terms.

## Evidence

Identify the failure mode, source, or maintenance problem that justifies the change. Link to the relevant research or issue when available.

## Validation

List the exact commands run and the observed result.

```text
git diff --check
```

GitHub Actions validation must also pass.

## Review checklist

- [ ] The diff is limited to the stated scope.
- [ ] New rules are tied to a concrete failure mode or maintenance need.
- [ ] Claims are supported by the repository or a traceable source.
- [ ] No generated scratch files, credentials, or unrelated cleanup are included.
- [ ] The validation output was read and reported accurately.
