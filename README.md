# Recipes for working with coding agents

One line each; the prompt is the payload, unlocking a capability the agent has
but skips by default. Two tools do the work: [iac](https://github.com/Anode1/iac)
(inter-agent messaging) and [ais](https://github.com/Anode1/ais) (associative
memory). Cost claims link to a paper.

## 1. Test-driven development, UI included

**Say:** "write the test first, then make it pass"

Regression tests are the industry standard, and for an agent they are the
definition of done: a failing test is a checkable target it iterates against
alone, without you in the loop. UI is not the exception. A project-local screenshot harness makes the screen testable headlessly: a
headless browser (CDP) for a web UI, a virtual display (Xvfb) for a native GUI.
The agent brings up the app, auths and seeds any data it needs, renders, captures
the PNG, reads it back, tears down, and asserts the expected state, in CI like any
other test. Non-UI: a suite
the agent runs itself (iac's `make ut` + ASan/UBSan + CI).

## 2. Recall, don't re-derive

**Store:** "`ais put <keys> <thing>`"   **Recall:** "`ais <keys>`"

Stop re-explaining the same steps each session. Store once, recall by key.
Re-searching the project burns thousands of tokens; recall is a handful, zero on
the CLI.

![What it costs to find something you already saved](images/ais_recall_cost.png)

[*Compress the Access*](https://doi.org/10.5281/zenodo.20764255)

Wire it into the agent as a skill: [`examples/ais-iac.SKILL.md`](examples/ais-iac.SKILL.md) teaches the agent to operate ais and iac (recall before re-deriving, store what it worked out, wait on `iac recv` instead of polling). It teaches the mechanism, not any keys or values, those are yours to choose.

## 3. Coordinate, don't poll

**Say:** "wait on `iac recv`"

Several agents on one box: one shared channel, not N pollers. Each parks a
blocking `recv` and wakes when a message lands. A parked `recv` is free; a model
polling its own inbox pays an inference per check.

![What it costs to keep an agent waiting](images/iac_wakeup_cost.png)

[*A Wakeup, Not a Broker*](https://doi.org/10.5281/zenodo.21206970)

## 4. Done is code, then doc, then test

**Say:** "code, then the doc, then a test that replays it"

Code is ground truth: on any conflict, code wins. An endpoint is not done when it
runs; it is done in three steps, in order:

1. Write the code, the ground truth.
2. Write the doc to match, in the same change. Prose is the agent's cheapest
   context, far fewer tokens than code, but only while true: a stale doc is worse
   than none, the agent builds to it and its gaps leak into the code.
3. Replicate the endpoint as a unit test, so the behavior is pinned and the doc
   has a check.

Write docs an agent can act on: for an API, a compact reference plus a parallel
examples file, same order, updated together, and the examples double as the
test's request and response. See [`examples/RESTapi.txt`](examples/RESTapi.txt)
(the reference) and
[`examples/RESTapi-examples.txt`](examples/RESTapi-examples.txt) (matching
request and response examples).

## 5. Agents work by concepts, not syntax

**Say:** "plain C, no frameworks"

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

## 6. Build once, promote the same artifact

**Say:** "build once, promote the same artifact"

A web app running across several environments (dev, uat, prod): do not rebuild
from source per environment. Build one artifact, promote that same one
everywhere; config and secrets come from the environment at runtime, never baked
into the build. One approved artifact is what ships, so a rollback is just
promoting the previous one, and a source-control compromise cannot silently reach
production.

[*Artifact Promotion as a Control Model*](https://doi.org/10.5281/zenodo.20451078)

## 7. Fresh session beats a compacted one

**Do:** start a fresh session once the agent begins compacting.

The context window is working memory, not storage. Once the agent compacts, it
reasons over a lossy summary and quality drops. Start fresh: the project's real
state lives in its docs (recipe 4) and its ais index (recipe 2), so a new session
reconstitutes by recall, cheap and complete, not by replaying the whole history.
Keep the window short.

## 8. Thin backend: SQL to JSON, no layers between

**Say:** "return JSON straight from the query, one connection per request"

For a full-stack app, drop the object layers between the database and the wire.
No POJOs, no ORM, no DTO mapping: hand-written SQL returns the rows, and the
row-to-JSON transform happens on the fly, streamed to the response as the result
set is read, never materialized as an object graph first. One user action is one
request is one connection: prefer a single query with JOINs over a loop that
issues a query per row, and one bulk insert over inserting in a loop, reusing the
same connection for the whole request. SQL is the native language for relational
algebra and the most direct way to say what you need; an ORM is a human-facing
layer over it (recipe 5), and for an agent it is cost with no benefit it can feel.
Fewer layers: fewer tokens to hold, fewer places to be wrong.

## 9. MISRA-style C: stack-first, bounded, no heap

**Say:** "stack-first, no heap, bounded buffers"

When the agent writes C, hold it to the avionics and medical-device standard and
most of C's CVE classes cannot occur by construction. Process one record at a time
in automatic (stack) variables and fixed-size buffers; peak memory is a function
of the struct sizes, not the data size, so a 10 GB input and a 10 KB input run in
the same footprint. Avoid the heap; when it is truly unavoidable the allocation is
bounded, documented, and freed on every path. Bounded strings only (`snprintf`,
never `strcpy`/`strcat`/`sprintf`). Single exit via `goto cleanup` when a function
holds a resource. Return codes, not `exit()`, inside the modules. Build clean under
`-std=c99 -Wall -Wextra` (a warning is a defect) and run the suite under
AddressSanitizer/UBSan. This is NASA's Power of Ten (rule 3, no heap after init)
and MISRA C:2012 rule 21.3, the discipline that lets the code run unattended.

Full rules: ais's [`STYLE.md`](https://github.com/Anode1/ais/blob/main/doc/dev/STYLE.md).

## 10. Frontend in triples: one screen, html + js + css

**Say:** "html/js/css triple per screen, split by concept not line count"

No build step: the frontend is served exploded, so a saved file is picked up on
reload, no bundler, no transpile, no framework. Each screen is a same-named
triple: `foo.html`, `js/foo.js`, `css/foo.css`, with a shared `app.css` for what
is common. Line count is not a concern; concepts are. When a screen grows a
second cohesive concept (a distinct component, an isolatable behavior), give it
its own file rather than letting one file carry two jobs, the same
one-file-per-concept discipline as the C core (recipe 9). A reader finds a
screen's markup, behavior, and style by its name, and each file does one thing.

## 11. Plain build: Makefile or Ant, nothing generated

**Say:** "plain Makefile for C, ant for Java, no autotools, no Maven"

A build should be readable in one file, not generated by another tool. For native
code a plain `Makefile` and `cc` are enough: no autotools
(autoconf/automake/libtool), no CMake, nothing that writes your build for you. For
Java an `ant` file is enough: no Maven, no Gradle, no online dependency
resolution; jars are vendored in the repo. Every heavier alternative (CMake,
Maven, Gradle, autotools) is itself a dependency you must install and keep in
version-sync; make plus a compiler, and ant, are close to what a stock box
already has. The build is also a file the agent reads and edits, so keep it small,
explicit, and offline: a stock toolchain builds the project unchanged, with no
generated layer between the command and the compiler (recipe 5).

Templates: [`examples/build.xml`](examples/build.xml) (Java web app: compile,
jar with a version-stamped manifest, war, run, test) and
[`examples/Makefile`](examples/Makefile) (C: build, test, sanitizers, install).
The fuller C starter, with the one-file-per-concept layout and tests wired, is
[aisconfig](https://github.com/Anode1/aisconfig).
