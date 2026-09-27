A stream that stops early is not a rare case. A proxy times out, a producer dies, someone
presses stop. The interesting part is not that the tail is lost — it is that the call which
was supposed to report the run often returns exactly what it returns for a run that
finished, so the transcript the user watched and the transcript the application saved
diverge with nothing raised anywhere.

I work on that layer: interruption, conformance and what a consumer is left holding.

### Record

Each of these is a link, and each says what it did **not** establish as well as what it did.

**[ag-ui-protocol/ag-ui#2354](https://github.com/ag-ui-protocol/ag-ui/pull/2354#issuecomment-5849020368)**
— a false positive in an open fix, found by probing the one branch its own test could not
reach.

The PR exempts deliberately cancelled runs from a new truncation check. `runAborted` is one
boolean per agent instance, cleared by the *next* `runAgent()` after an `await`, and read by
`isCancelled` when the stream ends — so a stop-and-resend clears the flag out from under the
run that was just cancelled, and it rejects with the PR's own error. Built the branch and its
base, reproduced on both, deterministic. The report also states the limit: **no transport
shipped in that repo was confirmed to reach it**, because their `HttpAgent` synthesizes a
`RUN_ERROR` on abort and short-circuits the check first.

**[vercel/ai#21209](https://github.com/vercel/ai/issues/21209)** — reported; one fix,
backported across three release branches, credited as co-author on each.

The AI SDK's middleware examples did not compile, and the guardrails one left `wrapStream`
unimplemented — the page said so in a comment, but a reader who copied it got redaction on
the generate path and none while streaming.
[#21210](https://github.com/vercel/ai/pull/21210) ·
[#21211](https://github.com/vercel/ai/pull/21211) ·
[#21212](https://github.com/vercel/ai/pull/21212)

**[ag-ui-protocol/ag-ui#2836](https://github.com/ag-ui-protocol/ag-ui/pull/2836)** — open,
awaiting review.

Three conformance fixtures for non-ASCII text split across streaming deltas, in the
protocol's own dual-lane format, which requires a fixture to pass the TypeScript and .NET
runners both. Green on both lanes locally; their CI has not run on it yet.

### Tools

**[cut-at-k](https://github.com/roshcompanylabs/cut-at-k)** — cut a stream at every point
and check what the consumer is left with.

Replays a stream through your own client once whole and once per prefix, then asks two
questions of each cut. Did the bytes before the cut change — truncation may lose the tail,
it must never alter the head. And does the client report the truncated run differently from
the complete one. The second is the one that is quiet when it is wrong, and a loop finds it
where reading the code does not.

Over AG-UI's published conformance corpus with `@ag-ui/client@1.0.0`, what it prints:

```
48 streams, 227 cuts, 18 streams excluded as invalid on purpose
4 cuts threw during replay

reported the same as the whole run    : 154
  of which something was actually lost : 54
  of those, no terminal event either   : 52
spread across                          : 34 streams
```

The recording is committed and names the client that produced it, so one command re-derives
every figure — and replaying the same corpus through an older client gives different
numbers, which is why the version is part of the result.

**[llm-stream-guardrails](https://github.com/roshcompanylabs/llm-stream-guardrails)** —
redact PII and secrets from an LLM response while it is still streaming.

Built around one invariant: for any chunking of the same input, the concatenated streamed
output is byte-identical to filtering the whole string at once, so a chunk boundary cannot
be the reason a secret gets through.
[Live demo](https://roshcompanylabs.github.io/llm-stream-guardrails/) — drag the chunk size
down to one character and watch the output stay the same. It also says what it does not do:
the detectors catch 47% to 81%, so it is defence in depth, not a compliance control.

Both MIT, zero runtime dependencies, published to npm with provenance attestations.

### How I work

Claims get checked against the thing itself, not against my memory of it — their test suite
run rather than read, the history searched before anything is called an oversight, the
reproduction driven over the real transport rather than a convenient mock. It is slower and
it keeps being worth it: a finding I was ready to file at AG-UI turned out to have been
reported 54 days earlier, and a measurement I was ready to publish turned out to be counted
over two different denominators. Neither went out.

I shipped an LLM feature into a live support workflow before I understood what a cut or a
crossed context does to one. That class of failure is why I went and learned this layer
instead of reading about it.

### Work

Open to contract and full-time work on streaming correctness, protocol conformance and
client SDKs. TypeScript and Node, SSE and Web Streams, and reading enough of a protocol's
other language lanes to write fixtures that satisfy them.

**roshcompanylabs@gmail.com** ·
[@roshcompanylabs.bsky.social](https://bsky.app/profile/roshcompanylabs.bsky.social)
