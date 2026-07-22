# ctf-solver

A directly invoked Codex skill for solving explicitly authorized CTF challenges from the current working directory or a user-supplied endpoint. It prioritizes source-derived attack paths, bounded distinguishing experiments, evidence-gated broad search, and reproducible validation.

XS-Leak guidance is separated by scope:

- `references/xs-leak.md` selects and validates general XS-Leak and browser side-channel oracles only when direct access is unavailable.
- `references/xs-leak-font.md` contains specialized ordered-text font and geometry extraction and is loaded only under strict prerequisites, preventing font-specific techniques from dominating other XS-Leak paths.

## Requirements

**Core**

- A Codex-compatible agent harness that loads skills from `~/.agents/skills/`

**Strongly recommended when relevant**

- [Docker](https://docs.docker.com/get-docker/) — used for isolated analysis containers, unavailable runtimes, callback receivers, or local challenge reproduction when explicitly requested

**Nice to have, but not required** — if any of these are missing on the host, `references/docker.md` allows a temporary container to provide the missing runtime when appropriate:

- [`uv`](https://docs.astral.sh/uv/) — Python work
- Node.js with `npm`/`npx` — JavaScript and browser automation
- Go, Rust, or C toolchains — native or performance-sensitive bounded helpers
- Java — JVM-based tools such as Ghidra or jadx
- Domain-specific tools such as SageMath, Z3, pwntools, debuggers, decompilers, or browser automation frameworks

**Optional tools that require user-owned access or credentials**

- IDA Pro with a working [`ida-domain-api`](https://ida-domain.docs.hex-rays.com/) setup — optional; Ghidra and other tools remain valid alternatives
- An authenticated [ngrok](https://ngrok.com/) account — only when a challenge requires an externally reachable callback

## Installation

Copy the `ctf-solver` directory into:

```text
~/.agents/skills/
```

Result:

```text
~/.agents/skills/ctf-solver/
├── SKILL.md
├── README.md
├── agents/
│   └── openai.yaml
└── references/
    ├── callback.md
    ├── docker.md
    ├── remote.md
    ├── xs-leak.md
    └── xs-leak-font.md
```

## Invocation

This repository includes `agents/openai.yaml` with `policy.allow_implicit_invocation: false`, so the skill only runs when explicitly invoked with `$ctf-solver`.

```text
$ctf-solver

분야: REV
설명: ...
FLAG 형식: FLAG{...}
```

For a locally deployed web or system challenge:

```text
$ctf-solver

분야: WEB
설명: ...
로컬 접속 정보: http://127.0.0.1:18432
FLAG 형식: FLAG{...}
```

The flag format is only an example. The skill does not hardcode a prefix.

The current working directory is treated as the challenge workspace, so challenge files do not need to be listed individually.
