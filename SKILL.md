---
name: ctf-solver
description: Solve an explicitly authorized CTF challenge from the current workspace or a user-supplied endpoint, obtain a verified flag, and preserve a reproducible solution using evidence-gated triage, bounded experimentation, and reproducible validation. Use only when directly invoked by the user.
---

# CTF Solver

## Objective

Solve the explicitly authorized challenge, obtain a flag supported by target behavior or equivalent concrete evidence, and preserve the minimum reproducible artifacts.

Treat the current working directory and user-supplied connection details as the complete authorized scope. Determine the real challenge structure from source, runtime behavior, protocols, and observed state rather than trusting the stated category.

## Investigation workflow

For ambiguous, nontrivial, or multi-path challenges:

1. Inspect the supplied source, configuration, runtime versions, entrypoints, and reachable behavior.
2. Trace attacker-controlled data and state across memory, arithmetic, parser, protocol, cryptographic, execution, file, query, template, browser, authorization, privilege, concurrency, and serialization boundaries.
3. Build a short ranked attack-path inventory.
4. Test the highest-ranked path with the smallest experiment that clearly distinguishes it.
5. Update, reject, or park the hypothesis before spending more work on it.

When source evidence exposes a direct, low-cost path, test it first. Record alternatives only if the direct path fails, depends on an uncertain assumption, or cannot produce sufficient validation.

Do not choose a path merely because its tooling is available, its progress is measurable, or it can run unattended. A direct source-derived sink outranks guessed credentials or generic payload search unless evidence shows otherwise.

After a negative result, state exactly what input space or mechanism was excluded and select a materially different path unless new evidence reopens the same one. Stop immediately when a path's predeclared stopping condition is met.

### Attack-path inventory

Before brute force, large wordlists, broad fuzzing, generic payload cycling, or wide enumeration, record plausible source-derived paths with:

- source observation
- attacker-controlled input or state
- sink or security boundary
- required prerequisites
- smallest distinguishing test
- estimated cost and target impact
- supporting and contradicting evidence
- status: open, confirmed, rejected, or parked

Do not require a formal inventory for an obvious, bounded, source-derived test that can directly confirm or reject the leading path.

Treat titles, comments, unusual language constructs, pinned runtime versions, strict comparisons, and deliberately exposed sinks as evidence. Use them to rank hypotheses; do not treat them as guarantees.

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

- permit at most one sanity round
- use only explicitly source-derived candidates
- cap the round at 1,000 candidates
- do not use general password dictionaries, mangling rules, masks, or exhaustive character sets
- park the path immediately after a miss

Do not expand a failed search because the next space is still computationally affordable. Expansion requires new evidence that changes the candidate distribution. Record that evidence before reopening the path.

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

Read only the references whose conditions apply:

1. Read `references/docker.md` when Docker-related files exist or the user supplies a locally deployed endpoint.
2. Read `references/remote.md` when an endpoint is required or is the primary challenge input.
3. Read `references/xs-leak.md` when direct read or direct JavaScript exfiltration is unavailable and progress depends on inferring secret-dependent browser or cross-origin state through an observable difference. The presence of a browser bot alone is not sufficient.
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
