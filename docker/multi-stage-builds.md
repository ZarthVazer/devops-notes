# Multi-stage builds and smaller images

## The idea

Build tools (compilers, pip caches, dev headers) are needed to **build** the app,
not to **run** it. A multi-stage build installs them in one stage and copies only
the result into a clean final image.

```dockerfile
# build stage
FROM python:3.12-slim AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# runtime stage
FROM python:3.12-slim
RUN useradd --system --no-create-home app
WORKDIR /app
COPY --from=builder /install /usr/local
COPY app/ ./app/
USER app
CMD ["gunicorn", "--bind", "0.0.0.0:8080", "app.main:app"]
```

## Checklist for a good image

- Copy dependency files **before** source code so the dependency layer is cached
- Use a `.dockerignore` (`.git`, tests, local venvs) to keep the build context small
- Run as a **non-root** user
- Pin base image versions (`python:3.12-slim`, not `python:latest`)
- Scan the result: `trivy image <image>`

## Check the size and layers

```bash
docker images <image>
docker history <image>
```
