# ADR 0002: A human confirms every closure

## Context

Automated retests are the core of Closeout, and it would be easy to let a passing retest close a finding on its own. But a matcher only checks what it was written to check. A fix can change the response enough to stop matching while leaving the vulnerability reachable another way: a different parameter, an encoding trick, a second endpoint.

The firm's name goes on the attestation, not Closeout's.

## Decision

Automation can take a finding as far as `appears_fixed`. Only a tester can move it to `verified_fixed`, by explicitly confirming the retest result. Rejecting the result sends the finding back to `still_vulnerable`.

The confirmation is recorded in the audit log, together with who made it.

## Consequences

- Closure always has a named person behind it, which is what an auditor expects.
- Automation saves the repetitive part (replaying the request, comparing results) and leaves judgment to the tester.
- Scheduled regression checks are the exception: a finding a human has already verified stays verified when a check passes, and is reopened automatically when one fails. Reopening is the safe direction, so it doesn't need a person.
