# Recipes for working with coding agents

Each recipe is one short thing to say, plus why it works. The prompt is the
payload: the agent works the rest out. The value is not in clever wording, it
is in knowing which one line unlocks a capability the agent already has but
does not use by default.

## 1. Make the agent look at the screen

For UI work the agent stops at "compiles and traced," because it assumes you
verify the screen yourself. One line changes that: tell it to render the page
and read the screenshot back before calling the work done. With a screenshot
harness wired in, the agent brings up the server, authenticates against local
dev, seeds a row if the page needs data, captures the screenshot, reads it
back, then cleans up after itself. Set it up once as a default skill and
self-verification becomes the norm; the one line still helps for one-offs.

## 2. Let many agents coordinate instead of poll

When a job needs several agents at once, give them a shared channel instead of
having each one poll. Point them at [iac](https://github.com/Anode1/iac), a
small inter-agent messaging tool over a shared append-only log: an agent joins
a room, parks a blocking `recv`, and wakes the instant a message addressed to
it lands. One line ("wait on `iac recv`") is enough; the agent works out the
room, its name, and the send/recv from the README. The point is that a parked
`recv` costs nothing while it waits, where a model polling its own inbox pays a
full inference per check.

## 3. Store procedures you repeat, recall them on demand

If you catch yourself re-explaining the same steps every session (a deploy
sequence, an env config, a lookup), write it down once into a memory index and
recall it with a short prompt. I use [ais](https://github.com/Anode1/ais), an
associative memory index: one line stores a procedure, a short recall pulls it
back. This saves tokens, the agent looks the procedure up on demand instead of
you re-typing it.

## 4. Define "done" so the agent enforces it

Give the agent an explicit definition of done and it holds the line without
being reminded. A useful rule for a new API endpoint: it is not done until a
test covers it and passes, and until it is written into the API docs in a
fixed, ordered place. That keeps the surface self-describing and
regression-guarded, so the next agent can test and orchestrate against it
automatically instead of rediscovering it each time.

---

Add new recipes at the bottom, same shape.
