# Recipes for working with coding agents

One line each; the prompt is the payload, unlocking a capability the agent has
but skips by default. Two tools do the work: [iac](https://github.com/Anode1/iac)
(inter-agent messaging) and [ais](https://github.com/Anode1/ais) (associative
memory). Cost claims link to a paper.

## 1. Make it look at the screen

**Say:** "screenshot the page and read it back before calling this done"

The agent stops at "compiles" and leaves the screen to you. A project-local
screenshot harness lets it self-verify: bring up the server, auth against local
dev, seed a row if needed, shoot, read the PNG back, tear down. (Non-UI code, same
rule: a suite it runs itself, e.g. iac's `make ut` + ASan/UBSan + CI.)

## 2. Coordinate, don't poll

**Say:** "wait on `iac recv`" (point it at the iac repo)

Several agents on one box: one shared channel, not N pollers. Each parks a
blocking `recv` and wakes when a message lands. A parked `recv` is free; a model
polling its own inbox pays an inference per check.

![What it costs to keep an agent waiting](images/iac_wakeup_cost.png)

[*A Wakeup, Not a Broker*](https://doi.org/10.5281/zenodo.21206970)

## 3. Recall, don't re-derive

**Store:** "`ais put <keys> <thing>`"   **Recall:** "`ais <keys>`"

Stop re-explaining the same steps each session. Store once, recall by key.
Re-searching the project burns thousands of tokens; recall is a handful, zero on
the CLI.

![What it costs to find something you already saved](images/ais_recall_cost.png)

[*Compress the Access*](https://doi.org/10.5281/zenodo.20764255)

## 4. Make "done" executable

**Say:** "not done until a test covers it and it is in the API docs"

An explicit "done" holds without reminders: tested, and documented in a fixed,
ordered place, so the surface stays self-describing and regression-guarded. See
[`examples/RESTapi.txt`](examples/RESTapi.txt) for the shape.

## 5. Docs are ground truth

**Say:** "docs describe the code; code wins on conflict; update the doc in the same change"

Prose is the agent's cheapest context, far fewer tokens than code, but only while
it is true. A stale doc is worse than none: the agent builds to it and its gaps
leak into the code.

## 6. Tests drive the work, UI too

**Say:** "write the failing test first (unit or screenshot), then make it pass"

Verify-after (recipe 1) is good; test-first is better. A failing test is a
checkable "done" the agent iterates against alone; the screenshot becomes an
assertion, not a look.
