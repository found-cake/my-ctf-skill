# XS-Leak Font and Geometry Reference

Use this reference only when all of the following are supported by evidence:

- attacker control is limited to CSS or inert markup
- direct JavaScript read or exfiltration is unavailable
- the secret is rendered as ordered, multi-character text
- simpler boolean, few-state, navigation, resource, cache, state-machine, or timing oracles are insufficient
- the exact challenge browser supports the required font-loading, shaping, and geometry behavior

This is a specialized extension of `xs-leak.md`, not the default XS-Leak workflow.

## 1. Prove the rendering pipeline

Before generating extraction fonts, capture in the exact browser:

- the live text node and its surrounding DOM
- computed `font-family` and relevant CSS
- `document.fonts` or equivalent font-load evidence in a local reproduction
- font and callback request timestamps
- `clientWidth`, `scrollWidth`, `clientHeight`, and `scrollHeight`
- whether the text produces layout boxes
- sanitizer output and the final insertion sink

Create known-positive and known-negative text fixtures and require the selected geometry predicate to flip reliably.

`unicode-range` may reveal which code points are needed for rendered text, but does not by itself reveal order or repeated positions. Do not claim ordered extraction from membership-only behavior.

## 2. Build an ordered-text predicate

When order is required, use a verified font transformation such as a GSUB ligature whose input represents `known-prefix + candidate-next-character`. Give a match a measurably different advance width and convert the width difference into a separately validated geometry predicate, such as overflow.

Keep shaping context bounded:

- prefer a short unique suffix or delimiter
- verify uniqueness against all rendered text
- avoid unnecessarily long ligatures or large substitution tables
- test the maximum supported prefix length locally
- confirm that browser, shaper, and font-table limits do not silently disable matching

Do not assume a generated font is correct merely because it compiles or downloads.

## 3. Use paired-complement predicates

For each encoded bit, build two complementary predicates:

- `bit-i-1`: candidates whose encoded index has bit `i` set
- `bit-i-0`: the complement

Measure both under the same conditions:

```text
bit-i-1 positive, bit-i-0 negative -> bit 1
bit-i-1 negative, bit-i-0 positive -> bit 0
both similar or controls fail       -> discard the run
```

Add independent known-positive, known-negative, and liveness controls. Configure a separation margin before extraction. Do not infer a bit from a raw callback-count threshold without paired controls.

## 4. Load fonts deliberately

Preload each measurement font through rendered but visually hidden text. `display:none` may prevent the font from being needed; off-screen positioning, clipping, or `opacity:0` may preserve layout, but must be verified in the exact browser.

Budget time for:

```text
page and dependency load + sanitizer/frontend initialization + font download
+ shaping and layout + measurement stages + guard intervals + receiver grace period
```

Do not infer font readiness from wall-clock delay alone. Use a local instrumented reproduction to identify readiness and then leave a conservative margin in the bot.

## 5. Choose the least complex visit strategy

Establish correctness before optimization:

1. one candidate predicate per visit
2. one paired bit per visit
3. several independent replicas with bounded concurrency
4. staged probes in one visit
5. channel-specific aggregation only after the exact browser proves it reliable

Multiplexing can reduce visits but adds font-loading races, stage overlap, cache interactions, and difficult error attribution. Keep concurrency conservative because multiple browser instances may contend for CPU, network, font caches, or server workers.

## 6. Prevent prefix poisoning

Prefix extraction is stateful: one wrong character can make all later ligatures miss. Use this acceptance protocol:

1. Measure each character at least twice with unique runs.
2. Accept only identical high-confidence results with valid controls.
3. On disagreement, perform a third independent measurement and require a majority.
4. Reject results whose complementary predicates lack the configured separation margin.
5. Persist each accepted prefix atomically and retain raw per-bit observations.
6. When the next position becomes all-zero, all-one, repeated, or low-confidence, revalidate the previous character before continuing.

Parity, repeated code words, or error-correcting codes may supplement controls when the bot lifetime permits. They do not replace liveness or a functioning oracle.

## 7. Diagnose common failures

| Symptom | Likely cause | Distinguishing check |
|---|---|---|
| Adjacent printable character | One low-order bit flipped | XOR encoded indices and repeat only the differing bit pair |
| Every result is identical | Wrong DOM target, inactive channel, unconditional prefetch, or wrong prefix | Revalidate fixtures and inspect computed state |
| Random text after one plausible error | Prefix poisoning plus timing noise | Roll back to the last independently confirmed prefix |
| Early stage missing | Fonts or dependencies not ready | Instrument readiness locally and increase the pre-measurement margin |
| Late stage missing | Bot closes or navigates first | Shorten the schedule and add an end margin |
| All predicates positive | Geometry threshold always matches or CSS composition overrides the probe | Test known-negative text and inspect exact dimensions |
| All predicates negative | Font did not load, shaping failed, text is not rendered, or timeline is inactive | Verify font request, substitution, layout boxes, and positive fixture |
| Callback order changes | Normal browser or network scheduling | Decode labeled routes, never arrival order |
| Local browser works but bot fails | Browser version, headers, origin, network, or lifetime differs | Reproduce the exact bot configuration |
| Plausible prefix collapses to one character | Earlier accepted error poisoned later predicates | Independently remeasure the last confirmed boundary |

After two unchanged noisy runs, do not blindly increase repetitions or delays. Instrument font loading, substitution, geometry, animation state, and lifecycle timestamps; then change the measurement design or isolate one paired bit in one visit.

## 8. Verify the extracted candidate

When the font oracle supports a full-candidate predicate:

1. rerun known-positive, known-negative, and liveness controls
2. test the exact full candidate
3. test at least one near miss with a changed character
4. repeat both in fresh bot sessions
5. require the candidate and near miss to separate under valid controls

Prefer an independent target verifier or a second oracle family when available. Preserve raw labeled observations, accepted-prefix checkpoints, generated fonts, payload generator, receiver code, browser version, and reproduction commands.
