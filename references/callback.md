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

## Event identity and logging

Design callbacks as measurements, not console messages — a receiver can see zero, one, or many requests for one logical attempt because of retries, browser request coalescing, caches, redirects, and tunnel behavior.

- Assign a unique `run_id` to every attempt or bot visit, and a stable `probe_id` to each tested condition; add `stage_id`/`replica_id` when multiplexing or repeating probes.
- Identify events by these tokens (in the path, a header, or the body), never by arrival order — concurrent connections and HTTP/2 do not preserve it.
- Include an unconditional baseline event so a missing result can be distinguished from "condition false" versus "payload never ran."
- Write events append-only and keep duplicates rather than collapsing them during collection.
- Decode from a snapshot taken after the attempt finishes and a bounded grace period expires, not from a live log.
- Use a fresh resource URL per run/replica and `Cache-Control: no-store` where caching isn't part of the channel itself, so a cache hit doesn't masquerade as a missing or duplicate event.

General browser-oracle measurement design is in `xs-leak.md`; specialized font bit encoding and prefix extraction are in `xs-leak-font.md`. This section covers only receiver-side event handling, which applies to any callback-based challenge.

## Ngrok

Ngrok usage for an authorized CTF callback is pre-approved and does not require additional confirmation by itself. The tool-installation rules in `SKILL.md` apply to installing ngrok itself.

If ngrok is unavailable or unauthenticated and this blocks progress, follow the user-interaction policy in `SKILL.md`.

Do not silently substitute another tunneling provider without verifying its behavior and disclosing the substitution.

An unauthenticated ngrok tunnel shows a browser warning interstitial page to visitors before reaching the actual endpoint. This silently blocks any browser-driven subresource request routed through it (font loads, images, fetches used as an oracle) — verify with an authenticated tunnel or a header/flag that skips the interstitial, and confirm with a real subresource request, not just a direct top-level visit, that requests actually reach the receiver.

## Data minimization

Receive and retain only data required to solve and verify the challenge.

- Do not collect unrelated user, administrator, or third-party data.
- Avoid exposing flags, cookies, tokens, or credentials in query strings when a request body or safer transport is practical.
- Store callback logs only in the local challenge workspace, and keep sensitive bodies separate from routine logs.
- Avoid printing sensitive values into unrelated logs or shell history.
- Do not send challenge files or source code to ngrok or unrelated external services.

The public endpoint is authorized only as a receiver for interactions caused by the explicitly supplied CTF target. After validation, remove disposable logs, stop tunnels, and remove temporary containers without touching the challenge service.

## Verification

Before delivering the final payload, verify:

1. the receiver is reachable through the ngrok URL, and a baseline event arrives for a known execution
2. the intended route and HTTP method work, and known-positive/known-negative controls are distinguishable
3. required parameters or request bodies are recorded correctly
4. malformed, duplicate, or repeated requests do not break the receiver or its decoder
5. no unnecessary local service is exposed
6. the receiver survives the expected concurrency, duration, and grace period

If a callback provider changes, repeat these checks — reachability alone is not sufficient.
