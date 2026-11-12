# Case Study: Luna — AI Voice Receptionist

**What it is.** An AI voice agent that answers a small business's phone
line, speaks English and Spanish, checks a live calendar, and books
appointments. It runs during business hours with a hard cap on call
length, and every call ends with a written summary delivered to the
owner.

## The problem

A one-person business can't answer the phone while doing the work.
Missed calls are missed customers — but a careless AI receptionist is
worse than none: one that invents prices, books over existing
appointments, or talks a vendor into a meeting can do real damage in a
single afternoon.

## Architecture

- Voice AI platform (speech, language model, telephony) connected to a
  workflow-automation layer that handles calendar access, post-call
  summaries, and notifications.
- The calendar is the single source of truth. The agent checks real
  availability before offering any time — it cannot offer a slot it
  hasn't verified.
- Returning callers are matched by phone number, then by name, and their
  existing appointment is read back and confirmed before any change is
  discussed.

## Safety design

- **Approval gate on changes.** Booking a new appointment is
  low-risk and autonomous. Cancelling or moving one is not — the slot is
  placed on a soft hold and the change waits for the owner's one-tap
  approval. The agent literally no longer has delete/update tools; the
  capability was removed, not just discouraged in a prompt.
- **No invention.** The agent never states prices, timelines, discounts,
  or guarantees. Price questions get one answer: an offer to book a
  meeting with the owner.
- **No solution design on calls.** The agent gathers requirements; it
  doesn't architect a system live on the phone.
- **Identity discipline.** It presents as the business's team and only
  names the owner if a caller explicitly asks who owns the company.
- **Vendor screening.** Sales pitches are never transferred and never
  booked. The agent collects a name, company, number, and one-line
  offer, ends politely, and flags the summary so the owner alone decides
  on a callback.
- **Human handoff.** Callers who declare an emergency, ask twice for a
  human, or are audibly distressed are transferred to a human line —
  a blind transfer by design, tested end-to-end before it was trusted.
- **Rescheduling requires an email address**, so changes are anchored to
  something verifiable rather than a voice claim.

## Lessons that changed the design

- **Verify the way the owner sees it.** Early testing "verified" a
  booking three ways — API timestamps, execution logs, and a rendered
  summary all agreed it was correct — while the calendar the owner
  actually reads displayed it hours late, because that calendar's own
  timezone setting had never been set. Rule since then: a booking isn't
  verified until it's read through the owner's own view.
- **Test the phone system by phoning it.** The transfer feature was
  only declared working after a live proof call reached the human line.
  Several earlier "should work" configurations hadn't.
- **Prompts are policy, but tools are law.** Anything that truly must
  not happen was removed at the tool level. Instructions drift; missing
  capabilities can't.
