---
name: ctf-solver
description: Solve an explicitly authorized CTF challenge from the current working directory or a user-supplied endpoint, obtain a verified flag, and provide a reproducible solution. Use only when directly invoked by the user.
---

# CTF Solver

## Objective

Solve the explicitly authorized CTF challenge and obtain a verified flag.

Treat the current working directory as the challenge workspace. Inspect its relevant files and subdirectories without requiring the user to enumerate them. If the user supplies a URL, host, port, protocol, credentials, or other connection information, treat those inputs as part of the authorized challenge scope.

The stated category and description are hints, not guarantees. Determine the actual challenge structure from the files, code, protocols, and observed behavior, including mixed-category or intentionally misleading challenges.

Identify the shortest reliable path to the flag, execute it, verify the result, and preserve the scripts or artifacts needed to reproduce it.

Do not accept a candidate solely because it matches the expected flag format. Confirm it through challenge logic, actual target behavior, recovered data, a reproduced verifier, or equivalent concrete evidence.

## Scope and autonomy

Proceed autonomously for non-destructive work within the explicitly supplied challenge scope, including:

- inspecting workspace files and relevant subdirectories
- static analysis and isolated dynamic analysis
- interacting with a supplied local or remote endpoint
- debugging, disassembly, and decompilation
- writing and running solver, exploit, recovery, protocol, or browser-automation scripts
- bounded fuzzing, constraint solving, and brute force
- creating reproducible intermediate artifacts
- validating flag candidates

When an approach fails, diagnose the result and continue with another reasonable approach.

Never expand scope to unrelated hosts, services, accounts, IP ranges, domains, or infrastructure discovered during analysis.

Do not perform destructive actions, persistence, denial-of-service, indiscriminate high-volume scanning, or unrelated data access.

## User interaction

Continue autonomously whenever a reasonable alternative exists.

Ask the user only when meaningful progress is blocked by one of the following:

- missing essential challenge input, credentials, or connection information
- an action only the user can perform, such as MFA, CAPTCHA, or account configuration
- ambiguous authorization scope
- an unavoidable destructive or availability-impacting action
- an essential host installation with no reasonable Docker, portable, existing-tool, or project-local alternative
- unavailable or unauthenticated ngrok when an external callback is required
- a required paid service or API
- unavoidable disclosure of challenge data or credentials to an unrelated external service

Before asking, try reasonable alternatives first. State briefly what was attempted, why alternatives are insufficient, and exactly what information or action is required.

Do not ask merely because the preferred tool is unavailable. Continue with an equivalent approach when possible.

## Workspace

Preserve original challenge files.

When modification, patching, conversion, or writable execution is required, work on copies or use a clearly named subdirectory such as `work`, `analysis`, or `output`.

Do not recursively inspect unrelated parent directories or unrelated locations.

Keep solver scripts, exploit scripts, recovered files, transcripts, request samples, patched files, and important notes organized so the solution can be reproduced.

Separate disposable build products, dependency caches, and temporary outputs from useful final artifacts.

## Tool and environment policy

Use existing tools and project-local dependencies when they are sufficient.

Prefer isolated Docker environments over installing tools, runtimes, or system dependencies directly on the host.

Use Docker freely for temporary analysis isolation and callback receivers, but never build or start the challenge service itself unless the user explicitly requests deployment.

Before requesting a host installation, first consider:

1. existing tools
2. project-local dependencies
3. portable binaries
4. a temporary Docker analysis environment
5. an equivalent technique or tool

Do not use Homebrew.

### Python

Use `uv` for Python work.

- Do not modify the system Python environment.
- Prefer `uv run` for standalone scripts.
- Use `uv add`, a local project, or PEP 723 inline metadata for dependencies.
- Record dependencies required for reproduction.

### Node.js

Install Node.js packages only as local dependencies in the working directory.

- Never use npm global installation.
- Use `npx` or `npm exec` for one-off commands.
- Record reproducibility-critical packages in `package.json`.

### Go

Use Python by default for automation and solver development.

Use Go when a bounded workload is large enough that Python would be a meaningful bottleneck, such as high-volume hashing, parsing, search, or protocol work.

### Disassembler and decompiler

Before installing Ghidra or another disassembler, try IDA via `ida-domain-api` first, and consult:

`https://ida-domain.docs.hex-rays.com/llms.txt`

Fall back to Ghidra or another tool when `ida-domain-api` cannot handle the binary format, lacks a needed capability, or a specific task (such as headless batch scripting across many binaries) is a clearly better fit elsewhere.

Do not trust decompiler output, file extensions, error messages, or a single tool result without appropriate verification.

## Working method

Start with inexpensive, high-information inspection.

Build evidence-based hypotheses and test each with the smallest experiment that can clearly distinguish outcomes.

Avoid generic payload cycling, broad scanning, large wordlists, or brute force without a specific hypothesis, bounded cost, and a clear stopping condition.

Prioritize the minimum path required to obtain and validate the flag rather than fully understanding every component.

Automate repetitive or error-prone work with deterministic scripts.

For performance-intensive work:

1. reduce the search space using formats, constraints, partial outputs, verifier behavior, or protocol state
2. estimate the workload
3. choose the simplest suitable implementation
4. use concurrency only when it materially helps
5. stop when evidence shows the approach is unproductive

Continue until the flag is verified or progress is genuinely blocked by unavailable information or capabilities.

## Conditional references

Read only the references whose conditions apply.

1. Read `references/docker.md` first when Docker-related files exist or the user supplies a locally deployed endpoint.
2. Read `references/remote.md` when interaction with an endpoint is required or the endpoint is the primary challenge input.
3. Read `references/callback.md` only when an externally reachable receiver is actually required, such as for XSS, SSRF, webhook, bot, or out-of-band interaction.

When multiple references apply, follow all applicable references.

The scope, autonomy, user-interaction, and tool policies in this file take precedence over reference files.

Do not load unrelated references.

## Flag validation and completion

The task is complete only when:

1. a final flag is obtained
2. the flag is supported by challenge logic, actual target behavior, or equivalent concrete evidence
3. the acquisition process is reproducible
4. the conclusion is not based on speculation or a mere format match

Use at least one strong validation path, such as:

- the challenge service returns the flag after a successful exploit
- the challenge reaches an explicit success state
- a fresh session or instance reproduces the result
- a local verifier or reconstructed algorithm confirms the value
- a recovered artifact contains the flag in clear challenge context
- independent observations converge on the same flag

Do not repeatedly submit candidates to a competition platform unless the user explicitly supplies and authorizes submission access.

## Final response

Use the following structure.

### FLAG

`<final flag>`

### 핵심 원리

### 획득 과정

### 검증 근거

### 재현 방법

### 생성 파일

If the challenge could not be solved, do not invent a flag. Report:

- confirmed observations
- supported hypotheses
- attempted approaches and their results
- the exact blocking point
- generated scripts and artifacts
- the highest-value next step
