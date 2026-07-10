# Recipes for working with coding agents

One line each; the prompt is the payload, unlocking a capability the agent has
but skips by default. Two tools do the work: [iac](https://github.com/Anode1/iac)
(inter-agent messaging) and [ais](https://github.com/Anode1/ais) (associative
memory). Cost claims link to a paper.

## 1. Coordinate, don't poll

**Say:** "wait on `iac recv`" (point it at the iac repo)

Several agents on one box: one shared channel, not N pollers. Each parks a
blocking `recv` and wakes when a message lands. A parked `recv` is free; a model
polling its own inbox pays an inference per check.

![What it costs to keep an agent waiting](images/iac_wakeup_cost.png)

[*A Wakeup, Not a Broker*](https://doi.org/10.5281/zenodo.21206970)

## 2. Recall, don't re-derive

**Store:** "`ais put <keys> <thing>`"   **Recall:** "`ais <keys>`"

Stop re-explaining the same steps each session. Store once, recall by key.
Re-searching the project burns thousands of tokens; recall is a handful, zero on
the CLI.

![What it costs to find something you already saved](images/ais_recall_cost.png)

[*Compress the Access*](https://doi.org/10.5281/zenodo.20764255)

## 3. Test-driven development, UI included

**Say:** "write the failing test first (unit or screenshot), then make it pass; UI goes through CI too"

Regression tests are the industry standard, and for an agent they are the
definition of done: a failing test is a checkable target it iterates against
alone, without you in the loop. UI is not the exception. A project-local
screenshot harness makes the screen testable: the agent brings up the server,
auths against local dev, seeds a row if needed, shoots, reads the PNG back, tears
down, and asserts the expected state, in CI like any other test. Non-UI: a suite
the agent runs itself (iac's `make ut` + ASan/UBSan + CI).

## 4. Docs are ground truth, keep them true

**Say:** "docs describe the code; code wins on conflict; update the doc in the same change"

Prose is the agent's cheapest context, far fewer tokens than code, but only while
true; a stale doc is worse than none, the agent builds to it and its gaps leak
into the code. So code wins on conflict, and the doc is updated in the same change
that touches the code. Write docs an agent can act on: for an API, a compact
reference plus a parallel examples file, same order, updated together. See
[`examples/RESTapi.txt`](examples/RESTapi.txt) (the reference) and
[`examples/RESTapi-examples.txt`](examples/RESTapi-examples.txt) (matching
request and response examples).

## 5. Agents work by concepts, not syntax

**Say:** "write plain, readable C; avoid frameworks when you can, skip C++ sugar"

Syntactic sugar and frameworks exist to help *people* manage complexity; an agent
works by concepts, not surface syntax, so they do not help it, they cost it. Its
native reasoning is a continuous latent representation we cannot read, where a
model can reason more efficiently than through our tokens
([Coconut](https://arxiv.org/abs/2412.06769)); the whole stack of languages,
syntax, and frameworks is a human-facing layer. C++ piles on overloads, templates,
and implicit conversions: ambiguity and ways to be wrong with no benefit the agent
can feel. Default to no framework; reach for one only when it plainly earns its
keep. If the agent writes it and you can read plain C, prefer plain C: less to
hold in context, fewer failure modes, cheaper to verify.

(In practice the author reaches for the most natural language per domain: Java for
web apps, C for systems and native code, SQL for relational-algebra engines, and
plain text for data, auditable and universally readable. Natural fits, not
framework layers.)
