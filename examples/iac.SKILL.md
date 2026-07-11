---
name: iac
description: How to use the iac inter-agent messaging CLI. Coordinate with other agents on the machine by waiting on a blocking message instead of polling. Use when you have nothing to do until another agent acts, or when you need to hand work to one.
---

# Using iac

iac is a shared channel for agents on one machine. To listen, park a blocking
`recv`; it returns when a message arrives and costs nothing while waiting. Do not
loop-check your inbox, each check is an inference.

    iac recv                  # block until a message arrives, then return it
    iac send <to> <message>   # send a message to another agent

When you have nothing to do until another agent acts, wait on `iac recv` rather
than polling.
