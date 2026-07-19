# Callback Receiver Policy

This reference applies only when the authorized challenge requires an externally reachable receiver, such as for XSS, SSRF, webhook, administrator-bot, or other out-of-band interaction.

The policies in `SKILL.md` and `docker.md` take precedence.

## Receiver setup

When a callback receiver is necessary:

1. create a minimal temporary receiver in Docker
2. use a unique container name when needed
3. use an available local port rather than assuming a fixed port
4. expose the receiver using the existing ngrok CLI
5. verify the public endpoint before delivering the payload
6. preserve the minimal receiver source, run command, and relevant logs
7. stop the ngrok tunnel and remove the receiver container after use

Do not run the receiver or its dependencies directly on the host when Docker is a reasonable option.

Do not use the challenge-service container as the receiver. The temporary-container constraints in `docker.md`, including the network-isolation rule, apply to callback receivers.

## Ngrok

Ngrok usage for an authorized CTF callback is pre-approved and does not require additional confirmation by itself. The tool-installation rules in `SKILL.md` apply to installing ngrok itself.

If ngrok is unavailable or unauthenticated and this blocks progress, follow the user-interaction policy in `SKILL.md`.

Do not silently substitute another tunneling provider.

## Data minimization

Receive and retain only data required to solve and verify the challenge.

- Do not collect unrelated user, administrator, or third-party data.
- Avoid exposing flags, cookies, tokens, or credentials in query strings when a request body or safer transport is practical.
- Store callback logs only in the local challenge workspace.
- Avoid printing sensitive values into unrelated logs or shell history.
- Do not send challenge files or source code to ngrok or unrelated external services.

The public endpoint is authorized only as a receiver for interactions caused by the explicitly supplied CTF target.

## Verification

Before delivering the final payload, verify:

1. the receiver is reachable through the ngrok URL
2. the intended route and HTTP method work
3. required parameters or request bodies are recorded correctly
4. malformed or repeated requests do not break the receiver
5. no unnecessary local service is exposed
