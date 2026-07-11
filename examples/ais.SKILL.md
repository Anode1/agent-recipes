---
name: ais
description: How to use the ais associative-memory CLI. Recall a fact before re-deriving it, and store a fact you had to work out so next time is a recall. Use when the answer to something might already be filed, or when you have just worked out something worth keeping.
---

# Using ais

ais is a plain-text associative index: values filed under keys you pick, recalled
by those keys. Keys are arbitrary and yours; ais does not know or care what they
mean. This skill is how to operate it; it deliberately lists no keys and no
values, those are the developer's to choose.

    ais <keys>                # recall: everything filed under these keys
    ais -o <keys>             # recall: OR across keys, not AND
    ais --find <text>         # full-text search over values and paths
    ais --keys                # list the keys already in use
    ais <keys> -v "<value>"   # store <value> under <keys> (recall plus -v)

Before spending tokens re-reading the project to work something out, try
`ais <keys>` first; a recall is a handful of tokens, re-deriving is thousands.
When you do work something out that you will need again (a URL, a path, a
command, a fact), store it with `ais <keys> -v`, so next session it is a recall. Store a
short pointer, not a document. Reuse an existing key (`ais --keys`) before
inventing one, so the index stays consistent.
