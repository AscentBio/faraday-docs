---
title: Docker
description: Run agent-generated code in an isolated Docker container.
---

## Quickstart

Build the sandbox image and add the sandbox block to your `faraday.yaml`:

```bash
docker build -f Dockerfile.sandbox -t faraday-code-sandbox .
```

```yaml
sandbox:
  backend: docker
  workspace:
    container_path: /workspace
```

That's it — Faraday runs on the host, code executes in a `faraday-code-sandbox` container.

## Custom sandbox image

Use your own Dockerfile to control what's installed in the sandbox:

```yaml
sandbox:
  backend: docker
  dockerfile: ./Dockerfile.my-sandbox
  docker_image: my-custom-sandbox
```

Faraday auto-builds the image when it isn't found locally. If you omit `docker_image`, it tags the build as `faraday-code-sandbox`.

To use a pre-built image instead:

```yaml
sandbox:
  docker_image: my-org/science-sandbox:latest
```

## Run the app in Docker

Use `--use-docker` to run Faraday itself inside a container:

```bash
faraday --use-docker "Your query here"
```

This builds `faraday-oss` if needed, bind-mounts your cwd at `/workspace`, and mounts the Docker socket for sandbox sidecars. See [examples/configs/docker.yaml](https://github.com/faraday-oss/faraday/blob/main/examples/configs/docker.yaml) for a full app-in-Docker config.

## Build the images

```bash
docker build -f Dockerfile.main    -t faraday-oss           .
docker build -f Dockerfile.sandbox -t faraday-code-sandbox .
```

## Further reading

- [YAML guidance](/reference/yaml-guidance) — mode combinations, workspace copies, and common patterns
- [Runtime config reference](/reference/runtime-config) — full field-by-field schema
- [Agent outputs](/reference/agent-outputs) — where run artifacts land
