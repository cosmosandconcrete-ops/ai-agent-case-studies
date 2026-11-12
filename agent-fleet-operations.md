# Case Study: Operating a Small-Business Agent Fleet

**What it is.** A fleet of AI agents and automations that run the
front office of a small services business — plus the operations layer
that keeps them honest. Individually each piece is modest; the case
study is the *shared operating pattern* that lets one person trust all
of them at once.

## The fleet

- **SMS booking assistant.** Books appointments by text using the same
  calendar rules as the voice receptionist. A confirmed booking is
  final — the assistant never retracts one. Change and cancellation
  requests by text are captured as details for the human team to
  confirm, not executed by the agent.
- **Lead follow-up system.** Timed follow-up sequences for new leads,
  built on a workflow-automation platform. Run discipline is strict:
  sequences advance one round at a time and only on the owner's word —
  a rule written after an early rehearsal showed how easily an eager
  operator (human or AI) can double-fire a sequence.
- **Website chatbot.** Answers visitor questions bilingually, discloses
  up front that it is an AI, and follows the same no-invention rule as
  every other agent: no prices, no guarantees, no facts it can't ground
  — it offers a meeting instead. It never uses the owner's name unless a
  visitor explicitly asks who owns the company.
- **Watchtower monitoring.** Automated website monitoring and alerting
  for client sites. (Described honestly as monitoring — never marketed
  as "cybersecurity," because there's no security operations center
  behind it.)
- **Internal technician agent.** A break-glass operations agent that can
  run diagnostics and prepared runbooks on the business's own
  infrastructure when the primary operator is unavailable. It does no
  client contact, no spending, and no deleting.

## Shared operating patterns

- **Approval gates everywhere.** Agents draft; humans send. Support
  replies are prepared as real drafts in the business inbox and nothing
  leaves without the owner's tap. Anything published, purchased, or
  contacted follows the same per-item approval rule.
- **Watchdogs that verify outcomes.** The fleet's biggest design driver
  is a stated fear: an agent declaring everything green while something
  is silently broken for the customer. So health checks verify *end
  states*, not job statuses — a scheduled run that "succeeded" but
  didn't deliver its message counts as a failure, and high-frequency
  mechanical checks run as silent scripts that only wake a human (or an
  agent) when there's actual news.
- **Script-vs-AI discipline.** Every automation is classified up front:
  mechanical work (sweeps, syncs, threshold alerts) runs as scripts at
  near-zero cost; agent runs are reserved for work needing judgment or
  language, because an agent run carries hundreds of thousands of
  tokens of standing context before it does anything. Token usage is
  audited on a schedule, and automations are checked against the whole
  roster before being placed so jobs don't pile onto the same minute.
- **Client-system access rules.** In any client's live system: read-only
  first; writes only where a build step requires them, on test records,
  never in bulk; nothing deleted or overwritten; every write logged with
  before/after state; anything unexpected means stop, change nothing
  further, and report. Workflows are proven on dummy data and at a
  volume of one before they fan out.
- **A living operations playbook.** Every fix or fine-tune to any system
  gets a dated lesson appended the same day — what it did, what changed,
  the rule now — and lessons that generalize are promoted to house
  standards shared by the whole fleet.

## Representative lessons

- **Per-process counters lie.** A rate limiter backed by in-memory
  storage silently doubled its real limit because the app ran two
  worker processes, each counting separately. Found by hammering the
  real endpoint past the stated limit — the limit is only "verified"
  when a live test trips it at exactly the stated number. Fix: shared
  storage for any counter that must hold across processes, now a house
  standard.
- **"Delivered" means the recipient got it.** A morning briefing job
  ran every check correctly and was marked successful — but never sent
  its message. Scheduled jobs that owe a human a message now carry an
  explicit mandatory-delivery step, and verification means finding the
  actual message, not trusting the run status.
- **Believe the user's eyes.** Repeatedly, the owner's live test — a
  text message, a screenshot, a calendar view — caught what API-level
  checks called healthy. The fleet's rule: when the checker and the
  human disagree, the human is right and there's a layer to go find.
