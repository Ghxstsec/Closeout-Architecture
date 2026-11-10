# ADR 0003: The retest worker is isolated from the rest of the system

## Context

The retest worker is the only component that sends requests to systems Closeout doesn't control, and the only one that parses their responses. Those responses are attacker-influenced by definition: the targets are, after all, vulnerable systems. If a parsing bug or a dependency vulnerability ever lets a response compromise the worker, the question is how far an attacker gets from there.

## Decision

The worker gets the least reach of any component:

- It runs on its own network segment, with internet egress but **no route to the API** or the web app.
- It talks to the database with its own role and **column-level grants**: it can read what a retest needs and write results, but it can't read users or sessions, edit findings' content or touch the audit log.
- It never handles issue tracker credentials. Status changes reach the tracker sync through a database trigger and an outbox, not through the worker.
- Its jobs are pinned to a single tenant, and it re-checks scope and window itself before every request, instead of trusting whatever queued the job.

## Consequences

- A compromised worker can't pivot to the API, impersonate users or rewrite history.
- The worker has to do its own authorisation checks rather than relying on the API's. That duplicates some logic, deliberately: it's a second wall, not a single point of failure.
