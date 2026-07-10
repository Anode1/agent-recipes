# Recipes for working with coding agents

Each recipe is one short thing to say, plus why it works. The prompt is the
payload: the agent works the rest out. The value is not in clever wording, it is
in knowing which one line unlocks a capability the agent already has but does not
use by default.

The examples point at two small public tools built for exactly these recipes:
[iac](https://github.com/Anode1/iac) (inter-agent messaging) and
[ais](https://github.com/Anode1/ais) (an associative memory index). Where a cost
is claimed, the measurement is in a linked paper.

## 1. Make the agent look at the screen

For UI work the agent stops at "compiles and traced," because it assumes you
verify the screen yourself. One line changes that: tell it to render the page and
read the screenshot back before calling the work done. A small project-local
screenshot harness is enough - the agent brings up the server, authenticates
against local dev, seeds a row if the page needs data, captures the screenshot,
reads it back, then cleans up after itself. Wire it in once as a default skill
and self-verification becomes the norm; the one line still helps for one-offs.
(For the same discipline on non-UI code - a real test suite the agent runs
itself - see iac's `make ut`, AddressSanitizer/UBSan lanes, and CI.)

## 2. Let many agents coordinate instead of poll

When a job needs several agents at once, give them a shared channel instead of
having each one poll. Point them at [iac](https://github.com/Anode1/iac): an
agent joins a room, parks a blocking `recv`, and wakes the instant a message
addressed to it lands. One line - "wait on `iac recv`" - is enough; the agent
works out the room, its name, and the send/recv from the README. A parked `recv`
costs nothing while it waits, where a model polling its own inbox pays a full
inference per check:

![What it costs to keep an agent waiting for a message](images/iac_wakeup_cost.png)

Measured in the paper: [*A Wakeup, Not a Broker*](https://doi.org/10.5281/zenodo.21206970).

## 3. Store procedures you repeat, recall them on demand

If you catch yourself re-explaining the same steps every session (a deploy
sequence, an env config, a lookup), write it down once into a memory index and
recall it with a short prompt. Use [ais](https://github.com/Anode1/ais), an
associative memory index: one line stores a procedure, a short recall pulls it
back. An agent re-searching the project for an answer burns thousands of tokens;
recalling it from a key index is a handful - or zero on the command line:

![What it costs to find something you already saved](images/ais_recall_cost.png)

Measured in the paper: [*Compress the Access*](https://doi.org/10.5281/zenodo.20764255).

## 4. Define "done" so the agent enforces it

Give the agent an explicit definition of done and it holds the line without being
reminded. A useful rule for a new API endpoint: it is not done until a test
covers it and passes, and until it is written into the API docs in a fixed,
ordered place. That keeps the surface self-describing and regression-guarded, so
the next agent can test and orchestrate against it automatically instead of
rediscovering it each time. See [`examples/RESTapi.txt`](examples/RESTapi.txt)
for the shape a compact, agent-readable API doc can take.

## 5. Keep the docs true - they are the agent's cheapest context

The strongest context you can give an agent is the whole project directory, and
in it the prose documentation - reading prose costs far fewer tokens than reading
the code for the same understanding. But that leverage holds only while the docs
are current. A stale doc is worse than none: the agent trusts it, builds to it,
and the doc's deficiencies migrate into the code. So set one standing rule - the
code is ground truth: where a doc and the code disagree, the code wins and the
doc is stale; and the doc is refreshed in the same iteration the code changes,
not "later." Cheap, true docs are the highest-leverage context an agent has.

## 6. Let tests drive the work - UI included

Verifying after the fact (recipe 1) is good; driving with tests is better. Have
the agent work test-first: write the test that defines the change, watch it fail,
then make it pass. A failing test is a precise, self-checkable definition of done
the agent iterates against without you in the loop - it cannot declare
victory until a test it can run says so. Extend it to the UI: the screenshot
harness from recipe 1 becomes an assertion, not just a look - the agent
renders the page and checks the expected element or state is present, so a UI
change is test-driven too, not eyeballed once. Unit tests for logic, screenshot
assertions for the screen: either way, "done" is executable.

--

Add new recipes at the bottom, same shape.
