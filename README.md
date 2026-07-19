# ctf-solver

A directly invoked Codex skill for solving explicitly authorized CTF challenges from the current working directory or a user-supplied endpoint.

## Requirements

**Core**

- A Codex-compatible agent harness that loads skills from `~/.agents/skills/`
- [Docker](https://docs.docker.com/get-docker/) — used for isolated analysis containers and callback receivers; also for the challenge service itself if you explicitly ask for local deployment

**Nice to have, but not required** — if any of these are missing on the host, `docker.md` allows spinning up a temporary container to provide the missing runtime instead, so pre-installing them just saves that step:

- [`uv`](https://docs.astral.sh/uv/) — Python work
- Node.js (with `npm`/`npx`) — JavaScript/Node work
- Go — for bounded workloads large enough that Python would be a meaningful bottleneck
- Java (JRE) — needed by JVM-based tools such as Ghidra or jadx; Ghidra itself does not need to be pre-installed, since it can be fetched and run from a temporary location on demand

**Must be provided by you** — these can't be substituted with a temporary container, since they depend on something you personally own or hold credentials for:

- IDA Pro with a working [`ida-domain-api`](https://ida-domain.docs.hex-rays.com/) setup — tried first for reverse engineering; falls back to Java-based tools if unavailable
- An authenticated [ngrok](https://ngrok.com/) account — only needed when a challenge requires an externally reachable callback

## Installation

Copy the `ctf-solver` directory into:

```text
~/.agents/skills/
```

Result:

```text
~/.agents/skills/ctf-solver/
├── SKILL.md
└── references/
    ├── docker.md
    ├── remote.md
    └── callback.md
```

## Invocation

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

The flag format shown above is just an example. Any competition's actual prefix (e.g. `picoCTF{...}`, `CTF{...}`, a custom prefix) works without changes, since `SKILL.md` never hardcodes or depends on a specific flag format.

The current working directory is treated as the challenge workspace, so challenge files do not need to be listed individually.
