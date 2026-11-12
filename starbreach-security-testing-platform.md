# Case Study: STARBREACH — Authorized Security-Testing Platform

**What it is.** A web-based penetration-testing platform built to give
small businesses a deeper assessment than a commodity vulnerability
scan — with the safety architecture treated as the product itself.
Clients grant written authorization; the platform tests only what is in
the locked scope, validates everything it reports, and plans (but does
not autonomously execute) exploitation.

## Why "validation-first"

Commodity scanners flag anything that answers a request, which floods
reports with false positives and trains clients to ignore them. This
platform's core design decision: **a finding is only reported as
confirmed when its content proves it** — an exposed file must actually
parse as the file it claims to be, a cross-account data leak must be
proven by the other account's content, and anything that can't be
confirmed is closed as a false positive *with the reason recorded*.
Every confirmed finding ships with plain-language remediation guidance,
so the report reads like an analyst wrote it, not like a scanner
emailed a spreadsheet.

## Methodology

- Engagement structure follows **PTES** (Penetration Testing Execution
  Standard): authorization and scoping, intelligence gathering,
  vulnerability analysis, approval-gated exploitation planning,
  reporting.
- Web checks are aligned to the **OWASP Web Security Testing Guide**;
  severities map to **CVSS v3.1** bands; template matches carry their
  CVE/CWE identifiers.
- Tooling combines established open-source scanners (port scanning,
  template-based vulnerability checking, passive subdomain enumeration)
  with a custom validation engine.

## Safety architecture

- **No authorization, no engagement.** Client work requires a signed
  authorization document on file before an engagement can exist in the
  system. Testing one's own systems runs under a typed, timestamped
  attestation instead — and is labeled non-client, permanently.
- **Locked scope, enforced in code.** Every single probe re-checks its
  target against the engagement's locked in/out scope list before it
  fires. The out-of-scope list wins on any conflict. Discovered
  subdomains are reported, never probed, until the client adds them.
- **Kill switch.** One action halts a running stage within a step;
  engines poll for it continuously, including inside long scanner runs.
- **Rate discipline.** Fixed pauses between probes, polite scanner
  timing, capped request rates, and hard per-target time boxes.
  Scanning traffic identifies itself with an honest user agent.
- **Exploitation is a plan, not a reflex.** The deepest stage generates
  proposed moves — each with a written risk note — and a human approves
  or denies every one individually. In the current build, approved plans
  are recorded and *no exploitation traffic is sent*; arming execution
  is a separate, deliberate step taken per engagement, inside an
  approved window, after verified client backups.
- **Authenticated testing stays read-only.** When a client provides
  dedicated test accounts (test data only), the platform logs in and
  checks access control from inside — cross-account reads, privilege
  escalation, pages reachable without any login. The only form ever
  submitted is the login form itself; everything after is page reads.
  Test passwords are encrypted at rest and never displayed.
- **Secrets hygiene.** Confirmed secret files are reported by variable
  *name* with values redacted; credential material is never written to
  the database or the report.
- **Full audit log.** Engagement creation, scope changes, run starts,
  approvals, and kill-switch presses are all timestamped and
  attributable.

## Current status (November 2026)

STARBREACH is in production as Cosmos & Concrete's security-assessment
offering. Its front door is a free Posture Check: a business owner
requests one for their own domain and gets a graded, plain-English
report on the public layer — email anti-spoofing records, certificate
and encryption health, browser safety headers, cookie flags. Deeper
assessments run under the authorization and scope discipline described
above. Our own production website is a standing test target: after
remediating what the checker flagged (SPF moved to a hard-fail policy,
DMARC raised from monitor-only to quarantine), it now earns a perfect
grade in the public checker.

## Lessons from building it

- **A time limit that only checks when output arrives is not a time
  limit.** The template scanner produced no output at all against a
  clean target, so a naive read loop hung indefinitely; the run only
  ended when the reader was rebuilt to check the clock (and the kill
  switch) on a timer instead of on data arrival.
- **Catch-all sites defeat naive matching.** A site whose every URL
  returns the same gate page — with a small per-request random token —
  made hash comparison report over a hundred phantom "hidden pages."
  The fix was content-*similarity* calibration against a baseline
  response, the same auto-calibration approach professional fuzzing
  tools use. Both bugs were caught by testing against a real, gated,
  production site — our own — before any client engagement.
- **The exclusions are a feature.** Denial-of-service techniques,
  fuzzing templates, and password brute force are deliberately absent.
  The legitimate question brute force answers ("does the lockout work?")
  is answered instead by a five-attempt, dummy-password check that a
  human approves first.
