# Remote Challenge Policy

This reference applies when interaction with an endpoint is required or the endpoint is the primary challenge input.

The policies in `SKILL.md` take precedence.

Treat only the endpoint, protocol, credentials, and directly related challenge resources explicitly supplied by the user as authorized scope.

## Initial reconnaissance

Begin with low-noise, high-information observations.

Determine:

- protocol and transport
- reachable service behavior
- authentication and session model
- visible inputs and outputs
- errors, redirects, headers, cookies, tokens, and client-side resources
- state transitions and reset behavior
- rate limits and timeouts
- whether the instance appears shared or isolated
- framing and encoding for non-HTTP protocols

Prefer targeted enumeration derived from evidence over broad automated scanning.

Do not scan adjacent IP ranges, unrelated ports, unrelated subdomains, third-party infrastructure, or services outside the supplied challenge scope.

## Evidence-driven testing

For each promising hypothesis:

1. identify the observation supporting it
2. design the smallest request or interaction that can test it
3. define the expected distinguishing result
4. execute the test
5. update the model of the target

Do not cycle through generic vulnerability payloads without evidence.

Use differences in status, body, length, timing, headers, cookies, redirects, connection behavior, and state carefully. Repeat timing-sensitive tests enough to distinguish a signal from normal variance.

## Request volume

Before increasing request volume:

1. define the hypothesis being tested
2. estimate the request count and rate
3. confirm that a lower-cost experiment is insufficient
4. use bounded concurrency
5. define a stopping condition
6. stop if instability or unintended impact appears

Respect published competition limits. In the absence of explicit limits, use conservative pacing, especially for browser bots and endpoints that start heavyweight processes.

## Sessions and reproducibility

Keep clear separation between:

- anonymous and authenticated sessions
- different accounts
- different cookies or tokens
- separate challenge instances
- local and remote state
- successful and failed interaction sequences

Preserve the minimum reproducible request samples, transcripts, cookies, tokens, and state transitions needed for the solution without exposing them externally.

Use direct protocol clients when sufficient.

Use browser automation only when browser execution, DOM state, storage, service workers, WebSockets, navigation, or frontend-generated requests materially affect the challenge.
