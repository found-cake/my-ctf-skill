---
name: ctf-solver
description: Solve an explicitly authorized CTF challenge from the current workspace or a user-supplied endpoint, obtain a verified flag, and preserve a reproducible solution using evidence-gated triage, bounded experimentation, and reproducible validation. Use only when directly invoked by the user.
---

# CTF Solver

## Objective

Solve the explicitly authorized challenge, obtain a flag supported by target behavior or equivalent concrete evidence, and preserve the minimum reproducible artifacts.

Treat the current working directory and user-supplied connection details as the complete authorized scope. Determine the real challenge structure from source, runtime behavior, protocols, and observed state rather than trusting the stated category.

Web search may be used to research general public techniques, language or runtime internals, library behavior, and CVEs. Do not search for a write-up or solution of this specific challenge or competition — by event name, challenge title, or a distinctive challenge string — and do not search for a target hash, secret, or flag value verbatim hoping to find it already published or leaked. This applies by default, whether or not the user says so in a given session. If such a search is already in progress, stop it immediately.

## Investigation workflow

For ambiguous, nontrivial, or multi-path challenges:

1. Inspect the supplied source, configuration, runtime versions, entrypoints, and reachable behavior.
2. Trace attacker-controlled data and state across memory, arithmetic, parser, protocol, cryptographic, execution, file, query, template, browser, authorization, privilege, concurrency, and serialization boundaries.
3. Build a short ranked attack-path inventory.
4. Test the highest-ranked path with the smallest experiment that clearly distinguishes it.
5. Update, reject, or park the hypothesis before spending more work on it.

When source evidence exposes a direct, low-cost path, test it first. Record alternatives only if the direct path fails, depends on an uncertain assumption, or cannot produce sufficient validation.

Do not choose a path merely because its tooling is available, its progress is measurable, or it can run unattended. A direct source-derived sink outranks guessed credentials or generic payload search unless evidence shows otherwise.

Before treating a cryptographic or hash comparison as the primary working path, enumerate every other reachable execution, deserialization, or parsing sink found in source, and record for each one whether its transformation chain has been traced to a conclusive result — works, or demonstrably blocked at an exact point — not merely attempted once or judged difficult at a glance. A single shallow check (such as one encoding or filter that appears to block a simple payload) is not conclusive. Only treat the cryptographic path as primary once this record shows every such sink is conclusively blocked, or has clearly lower expected value with evidence to support that.

A conclusion about what a target requires or registers, drawn only from reading the source, is a hypothesis, not a verified fact — this is especially true for runtime-specific semantics such as conditional declarations, dynamic dispatch, redeclaration, and hoisting, which can behave differently from a naive reading. Before concluding a path is blocked because "the code requires X," reproduce the exact source locally in an isolated container, exercise the actual branch or condition observed at the target (including the false/failure branch), and inspect the real runtime state. A generic clean environment of the same language and version is not a substitute for running the target's own conditional logic.

When a reproduction is used to confirm or rule out a hypothesis, verify it actually exercises the exact mechanism or identifier the hypothesis depends on, not a same-sounding proxy for it. Checking whether a plain name resolves is not the same test as checking whether an internal, mangled, or otherwise-derived identifier for the same underlying construct resolves — disproving one does not disprove the other, and treating it as if it does can close a path that was never actually tested.

After a negative result, state exactly what input space or mechanism was excluded and select a materially different path unless new evidence reopens the same one. Stop immediately when a path's predeclared stopping condition is met.

Closing or parking one path — including a search that the evidence gate below disallows — is not evidence that the challenge itself is blocked. Report the task as blocked only once every plausible path in the inventory has been tested or ruled out with evidence, not immediately after the first one closes.

### Attack-path inventory

Before brute force, large wordlists, broad fuzzing, generic payload cycling, or wide enumeration, record plausible source-derived paths with:

- source observation
- attacker-controlled input or state
- sink or security boundary
- required prerequisites
- smallest distinguishing test
- estimated cost and target impact
- supporting and contradicting evidence
- applicable reference in `references/`, if its conditions are met — read it before testing this path, not after
- status: open, confirmed, rejected, or parked

Do not assume a path's listed prerequisite is genuinely required just because another path appears to produce it. Test the dependency claim directly and cheaply — attempt the dependent path with the prerequisite absent, empty, or a placeholder value — before spending effort acquiring it. Mark it required only once the dependent path demonstrably fails without it.

Do not require a formal inventory for an obvious, bounded, source-derived test that can directly confirm or reject the leading path.

Treat titles, comments, unusual language constructs, pinned runtime versions, strict comparisons, and deliberately exposed sinks as a reason to test something, never as proof that a path is or is not a trap. A challenge author can plant a title, description, or comment to mislead just as easily as to hint, so weigh what the text says far less than what the code actually does — whether a sink is reachable, whether a transformation chain completes, whether a comparison is reachable with attacker-controlled input. When a comment's claim and the traced code behavior disagree, the traced behavior wins.

An unusual or old pinned runtime version (an outdated interpreter, engine, or library) is a different kind of signal — unlike text, an author cannot easily fake it, and it is rarely accidental. When it appears alongside an otherwise-unexplained code pattern (a conditional declaration, a deprecated construct, behavior that seems to require an explanation), research that exact version's own source, changelog, or internals for the relevant subsystem before concluding the pattern behaves as a newer or more familiar version would.

## Bounded experiments and search

### Lightweight distinguishing probes

A small probe does not require the full search gate when all of the following are true:

- inputs are derived directly from source, protocol, or an observed boundary
- the exact upper bound is small and declared before execution
- each result distinguishes a concrete hypothesis
- target impact is low and conservative pacing is used
- the probe stops immediately after its distinguishing condition is reached

Examples include a few parser-boundary cases, a short list of source-derived routes, or controlled positive/negative fixtures. Do not disguise generic payload cycling as a distinguishing probe.

### Broad or performance-intensive search evidence gate

This gate applies to password cracking, hash preimage guesses, general wordlists, rule expansion, masks, key search, generic payload cycling, broad directory enumeration, large fuzzing corpora, and other repetitive or performance-intensive searches.

Before executing such a search, record all of the following:

```text
Hypothesis:
Evidence constraining the candidate distribution or input grammar:
Exact candidate-generation rule and upper-bound count:
Measured or estimated rate and total runtime:
Why a smaller distinguishing experiment is insufficient:
Why higher-ranked source-derived paths are blocked or lower value:
Hard stopping condition:
Result that would justify any later expansion:
```

If any field is missing, do not run the search.

The following are not evidence that a candidate space is likely:

- the hash or verifier is visible
- the digest is fixed
- the search is offline
- a GPU or fast implementation is available
- the first portion of a search space is inexpensive
- a larger wordlist is already installed
- the search can run concurrently with analysis

Hardware speed changes only the cost of an already justified candidate space. It does not constrain entropy, make a cryptographic preimage attack plausible, or increase information gain from an arbitrary mask.

### Search without distribution evidence

When no source, generator, policy, hint, leak, partial value, or protocol constraint bounds the candidate distribution:

- permit at most one sanity round per target value (per hash, per verifier, per secret) — not one round per source text you happen to consult
- before running it, combine every source-derived candidate already available (title, description, comments, code strings, filenames, event name, expected format) into that single round, rather than trying a small new round each time a different piece of already-available text occurs to you
- use only explicitly source-derived candidates
- cap the round at 1,000 candidates
- do not use general password dictionaries, mangling rules, masks, or exhaustive character sets
- park the path immediately after a miss

Do not expand a failed search because the next space is still computationally affordable, and do not treat a different textual source you already had access to at the start (the title instead of the description, the event name instead of the filename) as new evidence — it is not. Expansion requires evidence discovered after the round that changes the candidate distribution. Record that evidence before reopening the path.

### Cryptographic checks

Identify whether the task is a preimage, second-preimage, collision, constrained password guess, nonce failure, key-reuse problem, algebraic weakness, or protocol misuse.

- Treat generic SHA-2 and comparable preimage search as infeasible.
- Compute dictionary or mask cost only after establishing why the secret belongs to that dictionary or mask.
- Prefer recovering generator state, exploiting nonce or key reuse, modeling algebraic constraints, bypassing the verifier, or reaching the protected operation through another call path.
- Report an exhausted arbitrary mask only as exclusion of that mask, never as evidence about the unconstrained secret.

## Scope and autonomy

Proceed autonomously for non-destructive work inside the explicit challenge scope, including:

- source and configuration inspection
- isolated static and dynamic analysis
- interaction with supplied local or remote endpoints
- debugging, disassembly, and decompilation
- bounded solver, exploit, browser, protocol, and recovery scripts
- evidence-supported fuzzing, constraint solving, and brute force
- flag validation and reproducible artifact creation

Never expand to unrelated hosts, accounts, IP ranges, ports, domains, or third-party infrastructure discovered during analysis. Do not perform persistence, denial-of-service, indiscriminate scanning, destructive actions, or unrelated data access.

## User interaction

Continue autonomously while a safe in-scope alternative exists. Ask only for missing essential input, ambiguous authorization, unavoidable destructive or availability-impacting work, an action only the user can perform, required paid access, or unavailable callback infrastructure as specified by the references.

Before asking, state what was tested, why alternatives are insufficient, and the exact required action.

## Workspace

Preserve original challenge files. Work on copies or in clearly named `work`, `analysis`, or `output` directories. Keep final solver artifacts separate from disposable caches, browser traces, generated corpora, and build products.

Do not recursively inspect unrelated parent directories or locations. Preserve only the minimum requests, transcripts, state, and recovered artifacts required for reproduction.

## Tool and environment policy

Use the simplest tool native to the challenge. Prefer existing tools and project-local dependencies, and use temporary Docker environments over host installation when isolation or unavailable runtimes are relevant. Never build, start, stop, or reconfigure the supplied challenge service unless the user explicitly requests deployment.

Do not use Homebrew.

### Python

Prefer Python for glue, protocol clients, automation, bounded solvers, and data processing when no domain-specific tool is clearly better. Use `uv`, prefer PEP 723 metadata for standalone scripts, and record exact reproduction dependencies. Do not modify the system Python environment.

### Domain-specific tools

Use domain-native tools when they materially simplify or strengthen the solution, such as SageMath or Z3 for algebra and constraints, pwntools and a debugger for binary exploitation, decompilers for reverse engineering, and browser automation for browser-dependent behavior.

### Node.js

Install packages only as local dependencies. Use `npx` or `npm exec` for one-off commands and preserve reproduction-critical package versions.

### Go and compiled helpers

Use Go, Rust, C, or another compiled helper when the target is native to that ecosystem or a bounded workload is large enough that Python would be a meaningful bottleneck.

### Performance-intensive work

After passing the broad-search evidence gate:

1. reduce the space using source and protocol constraints
2. estimate the exact upper bound
3. benchmark a representative slice
4. select the simplest sufficient implementation
5. use bounded concurrency only when it materially helps
6. stop at the declared boundary

## Conditional references

Check this list — and read any reference whose condition is met — before searching the web for how to implement a technique, and before writing the first payload for it. A web search is not a substitute for an applicable reference: reaching for one instead of the other risks reproducing exactly the calibration, timing, and environment pitfalls the reference exists to prevent.

These conditions can start applying mid-task as the working hypothesis changes — for example, after abandoning a direct-execution attempt in favor of inference through an observable difference. Re-check this list at every such pivot and before writing the first payload for a new technique, even if the inventory step above was skipped because the attempt looked like an obvious bounded test. If the session's context has been compacted or summarized since a reference was last read and the current work still depends on its content, re-read it — do not reconstruct its guidance from general knowledge or a fresh web search instead. Read only the references whose conditions apply:

1. Read `references/docker.md` when Docker-related files exist or the user supplies a locally deployed endpoint.
2. Read `references/remote.md` when an endpoint is required or is the primary challenge input.
3. Read `references/xs-leak.md` during initial reconnaissance (step 1 of the investigation workflow), as soon as source shows a browser bot or admin/visitor automation — do not wait until a side-channel technique is already judged necessary; the reference itself helps make that judgment.
4. Read `references/xs-leak-font.md` only when all of the following are supported by evidence:
   - attacker control is limited to CSS or inert markup
   - the secret is rendered as ordered text
   - direct JavaScript read or exfiltration is unavailable
   - a simpler boolean, few-state, navigation, resource, cache, or timing oracle is insufficient
   - the exact browser supports the required font-shaping and geometry behavior
5. Read `references/callback.md` when an externally reachable receiver is actually required.

Follow all applicable references. This file takes precedence when policies conflict. Do not load unrelated references.

## Flag validation and completion

Complete the task only when:

1. a final flag is obtained
2. challenge logic or target behavior supports it
3. acquisition is reproducible
4. the conclusion is not based only on format or one noisy observation

For deterministic paths, use a service success state, verifier, recovered artifact, or fresh reproduction. For noisy side channels, require independent measurements with valid controls and then use, in order of preference:

1. an independent target verifier or success state
2. a second oracle family
3. an exact-candidate predicate with at least one near miss, when the oracle supports it
4. a fresh independent reproduction reaching the same boundary

Do not repeatedly submit candidates to a competition platform unless the user explicitly supplies and authorizes submission access.

## Final response

Use this structure:

### FLAG

`<final flag>`

### 핵심 원리

### 획득 과정

### 검증 근거

### 재현 방법

### 생성 파일

If blocked, do not invent a flag. Report confirmed observations, supported and rejected hypotheses, exact stopping evidence, the blocking point, generated artifacts, and the highest-value next step.
