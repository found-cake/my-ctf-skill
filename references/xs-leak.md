# XS-Leak and Browser Side-Channel Reference

Use this reference only when direct read or direct JavaScript exfiltration is unavailable and an authorized CTF challenge requires inferring secret-dependent browser or cross-origin state through an observable difference.

This is an oracle-selection and measurement workflow, not a fixed exploit recipe. Browser behavior changes across engines, versions, policies, and partitioning models. Treat every primitive as a hypothesis until reproduced in the exact challenge browser and context.

Before choosing an oracle, reconstruct only the browser facts that affect the proposed leak: exact browser build and flags, origin and site relationships, credential delivery, frame or popup context, relevant security headers, storage or cache partitioning, and page lifetime. A browser bot is an execution environment, not evidence that the vulnerability is an XS-Leak. For receiver setup and event identity, read `callback.md`. Font-based ordered-text extraction is isolated in `xs-leak-font.md` and must not be loaded unless its stricter prerequisites are met.

## 1. Define the secret question

Classify what must actually be learned before choosing an oracle:

| Secret shape | Desired predicate or observation |
|---|---|
| Existence or login state | Boolean |
| Role, branch, or small status set | Few-state classification |
| Result count or length | Count, range, or threshold |
| Equality with a candidate | Equality predicate |
| Correct prefix | Prefix predicate |
| Character membership | Set-membership predicate |
| Ordered multi-character text | Ordered extraction only if no stronger predicate exists |

Do not default to character-by-character extraction when the target exposes a boolean, count, range, equality, or state-transition predicate.

Write the complete causal chain:

```text
secret-bearing state -> attacker-controlled predicate -> target response or browser state
-> observable event -> measurement record -> decoder decision
```

For every arrow, record the exact source line, endpoint, DOM node, browser API, resource, timing boundary, or log field that supports it. Reject a proposed leak when a non-secret cause can produce the same observation and cannot be controlled or measured.

## 2. Confirm that side-channel inference is necessary

Do not use an XS-Leak when any supported direct path is available, including:

- JavaScript execution in the secret-bearing origin
- readable same-origin endpoint or redirect chain
- weak `postMessage` origin validation
- opener, navigation, or same-origin communication that returns the value
- deterministic server-side state transition or authorization bypass
- sanitizer or parser bypass reaching direct execution

If attacker control is limited to CSS or inert markup, record the exact surviving syntax and sink. If the attacker controls a separate page, record the exact frame or popup relationship, top-level site, origin relationship, and whether the secret-bearing credentials are actually sent.

## 3. Select the smallest oracle family

Choose from target evidence rather than familiarity. When the leading family depends on undocumented or version-sensitive behavior, keep at least one materially different fallback.

| Family | Secret prerequisite | Observation | Main limitation |
|---|---|---|---|
| Attribute selector | Secret is serialized in an attribute visible to CSS | Conditional style or resource request | Does not generally inspect a live DOM property |
| State selector | Secret affects `:checked`, `:valid`, `:target`, `:has()`, or related state | Conditional style or request | Markup and engine dependent |
| Character membership | Secret characters occur in rendered text | Conditional font or resource use | Usually reveals membership, not order |
| Navigation or framing | Secret changes a reachable page or browsing context | Redirect, frame count, focus, close, error, or timing | Isolation and frame policy may remove the signal |
| Resource interpretation | Secret changes status, MIME, redirect, body, or metadata | Load/error, dimensions, media metadata, style, or download behavior | Element-specific normalization varies |
| Cache or connection state | Secret changes a request or cached object | Timing or callback behavior | Partitioning and connection reuse vary |
| State-machine transition | Secret controls whether a branch or mutation occurs | Subsequent observable state | Requires reliable reset and session separation |
| Parser or sanitizer mutation | Secret-dependent markup reparses differently | Execution, DOM, or resource difference | Exact parser and sink must be reproduced |
| Execution cost | Secret changes computation or rendering cost | Timing distribution or event-loop delay | High noise and usually many samples |
| Geometry or layout | Secret changes dimensions, overflow, or visibility | Size, scroll, timeline, or conditional resource | Often version-sensitive and asynchronous |

Use the strongest predicate the target naturally provides. For example, prefer equality over prefix extraction, prefix over per-character membership, and range queries over linear counting when supported.

## 4. Design efficient predicates

Separate the browser oracle from the query strategy. Depending on the predicate exposed by the target, use:

- direct equality tests
- prefix tests
- range or threshold tests
- binary search over ordered values
- balanced partitions of a candidate set
- multi-state classification
- adaptive queries based on prior results
- independent replicas or voting for noisy channels

Predeclare the candidate grammar or range. A clever encoder does not justify an unconstrained alphabet or unknown-length extraction.

## 5. Validate the exact browser surface

Reproduce the exact browser build, engine version, launch flags, top-level site, origin relationships, and credential state. Record:

- CSP, CORS, COOP, COEP, CORP, X-Frame-Options, sandbox, referrer, and mixed-content behavior
- page lifetime, wait condition, background throttling, and bot concurrency
- cookies actually sent, storage state, service workers, and partition keys
- cache state, connection reuse, timer precision, and callback reachability
- source payload, transformed output, live DOM, and final insertion sink when parsing is involved

Create known-positive and known-negative fixtures that differ only in the tested predicate. Require the intended browser state or observation to flip while unrelated controls remain stable.

## 6. Test the oracle in isolation

### Window and frame state

Test only cross-origin properties and events exposed by the exact browser, such as frame count, `closed`, focus or blur, permitted navigation, and whether opening or embedding completes. Record the exact result or exception in both known states. Do not rely on historical browser behavior.

### Resource interpretation

Test each inclusion element independently. Determine which status, MIME, redirect, body, or metadata differences affect load/error behavior and whether a readable property changes. Do not generalize from image behavior to script, stylesheet, media, download, or frame behavior.

### Timing and cache

Treat observations as distributions:

- initialize positive and negative states identically
- randomize measurement order
- retain raw samples
- use robust summaries such as median and spread
- predeclare sample cap and classification margin
- reject overlapping distributions instead of forcing a bit

Account for cache partitioning by top-level site, DNS, redirects, service workers, connection reuse, timer precision, and background throttling.

### State-machine leaks

Model states and transitions explicitly. Reset between trials, verify the starting state, and distinguish a secret-dependent transition from session expiry, CSRF failure, retries, or a race on a shared instance.

### Geometry and rendering

Measure the exact geometry property that carries the predicate. Confirm that layout boxes exist, dependencies are ready, and known-positive and known-negative fixtures separate. Load `xs-leak-font.md` only when ordered rendered text and font shaping are genuinely required.

## 7. Calibrate identifiable experiments

Treat every observation as uncertain until calibrated. Collect:

- known-positive control
- known-negative control
- liveness or execution baseline independent of the secret
- normal callback counts, retries, timing, redirects, and cache behavior

Change one logical condition per probe. Label every event so it can be reconstructed without relying on arrival order. Use unique run and probe identifiers; receiver-side details belong in `callback.md`.

Apply only controls relevant to the channel:

- liveness control
- positive and negative fixtures
- unique nonces and resource URLs
- `Cache-Control: no-store` when cache is not the channel
- guard intervals for staged measurements
- bounded grace period before decoding

Prefer relative classification within the same run over a global threshold. Reject a run when controls do not separate. Never decode a missing callback as “false” without liveness evidence.

## 8. Avoid obsolete or unsupported assumptions

Treat these as hypotheses requiring direct proof, not reusable facts:

- reading arbitrary `:visited` styles or history state
- cross-origin DOM access or `getComputedStyle`
- using iframe `onload` alone to distinguish HTTP status
- assuming global, unpartitioned cache state
- assuming precise high-resolution timers
- assuming `window.length`, opener state, focus, or frame properties survive COOP, sandboxing, or isolation
- assuming third-party credentials are sent in an iframe
- assuming a single timing sample or callback count is a stable oracle
- assuming absence means the predicate was false

If the primitive does not separate two known states in the exact target browser, discard it regardless of historical writeups.

## 9. Multiplex only after correctness

Start with the least complex strategy:

1. one condition per visit
2. one independent predicate per visit
3. a small number of bounded parallel replicas
4. staged or aggregated measurements only after lower-level correctness is proven

Multiplexing introduces loading races, stage overlap, shared resource state, and harder error attribution. Build a full timeline budget and finish all measurements well before bot shutdown.

## 10. Verify the result

Do not accept a value only because it matches the flag format. Apply the validation preference order from `SKILL.md`'s flag validation section — an independent verifier or success state first, then a second oracle family, then an exact-candidate-plus-near-miss test, then a fresh independent reproduction.

Rerun controls in fresh sessions, and preserve the candidate, raw labeled observations, browser version, payload generator, decoder, and exact reproduction command.
