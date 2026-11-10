# ADR 0001: An inconclusive retest is never "fixed"

## Context

A retest replays a proof request and checks whether the vulnerability still shows up. The naive rule is: if the matcher doesn't match, the issue is gone.

That rule is wrong in exactly the cases that matter. If the client puts the endpoint behind a login, the WAF starts blocking Closeout's IP, the session cookie in the proof request has expired, or the server is down, the vulnerable response disappears. But the vulnerability hasn't.

A remediation tool that turns "we couldn't reach it" into "fixed" produces false attestations, and an attestation is something the client may show to an auditor.

## Decision

Responses with status 401, 403, 407, 408, 429 and any 5xx are **inconclusive**, whatever the matcher says. So are timeouts and connection failures.

An inconclusive result never changes the finding's status: it goes back to whatever it was before the retest, and the retest history records why.

## Consequences

- Some genuinely fixed findings won't auto-verify (for example, when the fix was to put the endpoint behind authentication). The tester has to update the proof request or verify it manually. That costs a little time, and in exchange no finding is marked fixed by accident.
- Firms can trust `appears_fixed` as a real signal, not a guess.
