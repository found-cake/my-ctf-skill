# Docker Policy

This reference applies when Docker-related files exist or the user supplies a locally deployed challenge endpoint.

The policies in `SKILL.md` take precedence.

## Challenge deployment

Docker-related files are challenge source material and do not imply permission to deploy the challenge.

When the user supplies a local URL, host, or port, assume the challenge instance is already deployed.

- Use the supplied endpoint.
- Do not build, create, start, stop, restart, remove, or reconfigure challenge containers.
- Do not change their ports, names, networks, volumes, environment variables, or compose project names.
- Do not run `docker build`, `docker run`, `docker compose up`, or equivalent commands for the challenge service.
- Inspect `Dockerfile`, Compose files, entrypoints, environment files, and container configuration only as source material for understanding the challenge.

Do not use `docker exec`, `docker cp`, log/volume inspection, or any other form of container introspection to read a secret, hash, source file, or flag directly out of the running challenge container as a shortcut when an intended path is difficult or blocked. This does not satisfy the flag-validation requirement that acquisition be supported by target or challenge logic, and the resulting flag would not be reproducible through the actual exploit.

If no endpoint is supplied:

- continue with static and file-based analysis first
- do not automatically deploy the challenge
- request connection information only when a running instance is essential and progress is blocked
- deploy the challenge only when the user explicitly requests deployment

## Temporary analysis containers

Temporary Docker containers are allowed for isolated analysis when they do not interfere with the challenge deployment.

Use them for purposes such as:

- running untrusted binaries or tools
- providing unavailable runtimes or system libraries
- compiling or testing in a controlled environment
- reproducing a narrow component of the challenge
- running callback receivers as defined in `callback.md`

For temporary containers:

- prefer `--rm`
- use unique names only when a name is necessary
- avoid fixed host ports
- avoid persistent volumes and shared networks unless required
- do not reuse names, ports, volumes, or networks declared by the challenge
- do not join the challenge deployment network unless explicitly necessary and authorized
- mount the workspace read-only when writable access is unnecessary
- preserve only artifacts required for reproduction

Distinguish clearly between:

1. challenge-service containers, which require explicit user authorization to deploy or manage
2. temporary analysis or callback containers, which are allowed under the constraints above
