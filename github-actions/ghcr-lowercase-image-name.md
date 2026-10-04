# GHCR: "repository name must be lowercase"

## Problem

A Docker build in GitHub Actions failed with:

```
invalid tag "ghcr.io/ZarthVazer/cicd-pipeline-demo:sha-9a75c4f": repository name must be lowercase
```

## Cause

The image name was built from `${{ github.repository }}`, which keeps the original
casing of the owner (`ZarthVazer`). Docker / OCI registries only accept lowercase
repository names.

## Fix

Lowercase the name in a shell step and pass it on as a step output:

```yaml
- name: Set image name and tag
  id: vars
  run: |
    echo "image=ghcr.io/${GITHUB_REPOSITORY,,}" >> "$GITHUB_OUTPUT"
    echo "tag=sha-${GITHUB_SHA::7}" >> "$GITHUB_OUTPUT"

- uses: docker/build-push-action@v7
  with:
    tags: ${{ steps.vars.outputs.image }}:${{ steps.vars.outputs.tag }}
```

`${VAR,,}` is bash syntax for "convert to lowercase"; `${VAR::7}` takes the first 7 characters.

## Takeaway

Never build registry paths directly from `github.repository` or `github.actor`
without normalizing case. Works fine for lowercase usernames, breaks for everyone else.
