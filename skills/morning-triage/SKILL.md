---
name: morning-triage
description: >
  Builds a short morning plan from Moxo: the steps waiting on the user, the flows
  that are past due, and the one thing to do first. Use when the user says
  'what's on my plate in Moxo', 'what needs me', 'what's overdue', 'triage my
  Moxo', 'plan my morning', or starts the day by asking about their flows. Not for
  a report on one client or one template (use flow-status-report), not for
  starting a flow (use start-flow-for-client), and not for a whole-organization
  standup (the server's /flow_activity_summary prompt covers that). It reads
  first and never sends a reminder or completes a step without a yes.
---

# Morning triage

The Moxo server has a `/show_my_attention` prompt that sorts the user's work into
groups. This skill gathers the same things, then does the part that prompt leaves
open: it ranks the list, names the one step to take first, and offers to act on it.

## Gather

Make these three calls together. None depends on another.

1. `list_attention_items`: the steps where the user is the current assignee. Each
   item has `flowId`, `flowName`, `stepExecutionId`, `stepName`, `dueAt` and
   `isOverdue`.
2. `list_overdue_flows`: flows past their due date, most overdue first, with
   `daysOverdue`.
3. `get_dashboard_summary`: counts of active flows, overdue flows and flows
   completed today, for one line of context.

`list_attention_items` only works with a personal access token. An organization
key has no person behind it, so the list comes back empty. If the list is empty,
say "Nothing is waiting on you" and move on to the overdue flows. Don't guess at
why it is empty.

For overdue flows that aren't on the user's list, find out what each one is stuck
on: call `query_steps` with that `flowId` and `isOverdue: true`. The result names
the late step and its `assignees`. Do this for the five most overdue flows at most.

## Sort

Two groups, at most five items each. Say how many more there are.

- **Needs you.** Items from `list_attention_items`. Overdue first (most days late
  first), then due today, then by due date, then items with no due date.
- **Overdue, waiting on someone else.** Overdue flows with no item in "Needs you".
  Name the late step and who it is waiting on.

Name each flow so the user can find it in Moxo. Where the client can open MCP
resources, link it as `[Flow name](moxo://flows/{flowId})`.

## Pick the first thing

End with one recommendation and the reason for it, in this order of priority:

1. The most overdue step in "Needs you".
2. A step in "Needs you" that is due today.
3. A flow that has waited on someone else for three days or more: suggest a
   reminder to the person it is waiting on.
4. If nothing is late or due: say so, and name the next due date.

## Offer, then act only on a yes

After the plan, offer the next move. Don't take it.

- **Send a reminder** with `send_reminder` (`flowId`, `stepExecutionId`). Use the
  step's `id` from `query_steps` or the `stepExecutionId` from
  `list_flow_steps`. A reminder sends a real email or text, so name the person and
  the step and wait for a yes to that exact reminder.
- **Work on a step**: open it with `get_flow_step` and show what it asks for.
  `complete_step` changes a real flow. Show the answers you will submit and wait
  for a yes first.

A yes covers one action. Ask again before the next one.

## Example

```
Moxo, Monday 9:05. 42 active flows, 6 overdue, 3 completed today.

Needs you (3)
- Acme Corp onboarding: Approve engagement letter. 2 days overdue.
- Patel tax return 2025: Review documents. Due today.
- Northwind renewal: Sign-off. Due Thursday.

Overdue, waiting on someone else (2)
- Globex KYC: Upload ID, waiting on Dana Lee (client). 5 days overdue.
- Initech audit prep: Partner review, waiting on Sam Ortiz. 1 day overdue.

Start with the Acme engagement letter: it is the most overdue step on your list.
Want me to send Dana Lee a reminder about the Globex ID upload?
```
