# Pin and update third-party actions

## Problem

A pipeline that used to work started failing with:

```
Unable to resolve action `aquasecurity/trivy-action@0.24.0`, unable to find version `0.24.0`
```

## Cause

The action's maintainers changed their tag format (now `v0.x.y`), and the old tag
was no longer resolvable. Third-party actions are just Git refs: they can be moved,
renamed or deleted.

## Fix

Check which tags actually exist before pinning:

```bash
git ls-remote --tags https://github.com/aquasecurity/trivy-action | tail
```

and update the reference:

```yaml
- uses: aquasecurity/trivy-action@v0.36.0
```

## How to avoid it

- Let **Dependabot** keep actions up to date:

  ```yaml
  # .github/dependabot.yml
  version: 2
  updates:
    - package-ecosystem: github-actions
      directory: /
      schedule:
        interval: weekly
  ```

- For security-sensitive workflows pin to a full commit SHA instead of a tag,
  so the code you run can't change underneath you:

  ```yaml
  - uses: actions/checkout@<full-40-char-sha> # v7.0.1
  ```
