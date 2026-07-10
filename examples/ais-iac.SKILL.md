---
name: ais-iac
description: How to use the ais (associative memory) and iac (inter-agent messaging) CLIs. Recall a fact before re-deriving it, store a fact you had to work out so next time is a recall, and coordinate with other agents by waiting on a message instead of polling.
---

# Using ais and iac

Two small CLIs are available. This skill is how to operate them; it deliberately
lists no keys and no values, those are the developer's to choose.

## ais - recall before you re-derive

ais is a plain-text associative index: values filed under keys you pick, recalled
by those keys. Keys are arbitrary and yours; ais does not know or care what they
mean.

    ais <keys>                # recall: everything filed under these keys
    ais -o <keys>             # recall: OR across keys, not AND
    ais --find <text>         # full-text search over values and paths
    ais --keys                # list the keys already in use
    ais -v "<value>" <keys>   # store <value> under <keys>

Before spending tokens re-reading the project to work something out, try
`ais <keys>` first; a recall is a handful of tokens, re-deriving is thousands.
When you do work something out that you will need again (a URL, a path, a
command, a fact), store it with `ais -v`, so next session it is a recall. Store a
short pointer, not a document. Reuse an existing key (`ais --keys`) before
inventing one, so the index stays consistent.

## iac - wait, do not poll

iac is a shared channel for agents on one machine. To listen, park a blocking
`recv`; it returns when a message arrives and costs nothing while waiting. Do not
loop-check your inbox, each check is an inference.

    iac recv                  # block until a message arrives, then return it
    iac send <to> <message>   # send a message to another agent

When you have nothing to do until another agent acts, wait on `iac recv` rather
than polling.
