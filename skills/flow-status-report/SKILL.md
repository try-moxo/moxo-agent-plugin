---
name: flow-status-report
description: >
  Writes a status report on Moxo flows for one client or one template over a date
  range: what started, what finished, what is stuck or overdue, and what happened,
  using list_flows, query_steps and query_audit_trail. Use when the user asks 'how
  is Acme doing', 'status of all onboarding flows this month', 'what happened on
  the KYC flows last week', or wants a summary to send to a client or a manager.
  Not for the user's own to-do list (use morning-triage), not for a
  whole-organization standup with no client or template (the server's
  /flow_activity_summary prompt covers that), and not for changing flows. It only
  reads.
---

# Flow status report

The Moxo server has a `/flow_activity_summary` prompt for a quick standup across
the whole organization over the last day. This skill narrows the view to one client
or one template, over the window the user asks for, and goes down to steps and
history.

## 1. Pin down the scope

You need a client or a template (or both), and a date window.

- **Client.** Call `list_external_users` with `search`. Use the person's
  `accountId`. If several people match, ask which. If the person has no
  `accountId`, flows can't be filtered by client: ask for a template instead, or use
  `search` to find flows by name.
- **Template.** Call `list_templates` and use its `id`. If several match, ask.
- **Window.** Turn the user's words into an ISO-8601 start and end ("last week",
  "this month", "since 1 October"). If they gave none, use the last 7 days and say
  so in the report.

## 2. Find the flows

`list_flows` filters only by start date, so a date filter alone misses flows that
started earlier and were still moving in the window. Make these calls with
`accountId` and/or `templateId`:

1. Started in the window: `startedAfter` set to the window start.
2. Still open: `status: "IN_PROGRESS"`, then `status: "NOT_STARTED"`, with no date
   filter.
3. Failed: `hasFailed: true`.

Use `limit: 100`, and page with `offset` while `pagination.hasMore` is true. Merge
by flow `id`. Drop flows that started after the window end.

## 3. Read the steps

- **Client scope:** one `query_steps` call with the `accountId` and
  `isOverdue: true` finds every late step across the client's flows. A second call
  with `hasReopens: true` shows rework.
- **Template scope:** `query_steps` has no template filter, so call it per flow
  with `flowId` and `isOverdue: true`.

Each step row has `stepName`, `status`, `dueAt`, `assignees` and `reopenCount`.

## 4. Read the history

`query_audit_trail` has no client or template filter, so call it per flow with
`flowId` and `since` set to the window start. Drop events after the window end.
Useful event types: `flow.started`, `flow.completed`, `flow.cancelled`,
`step.completed`, `step.reopened`, `step.overdue` and `step.failed`. A
`flow.completed` event is how you know a flow finished inside the window.

Each call returns at most 100 events, newest first. If a flow hits 100, say its
history is cut off. With more than 15 flows, read the history only for flows that
finished, failed, or have late steps, and say so in the report.

## 5. Write the report

Lead with what changed most. Keep it to one screen.

```
Acme Corp: flow status, 1 to 31 October 2026 (as of 3 November, 09:00)

3 started · 2 finished · 4 open · 1 overdue

Needs attention
- Acme payroll setup: "Upload bank details" is 6 days overdue, waiting on
  Dana Lee.

Finished
- Acme onboarding finished on 14 October.
- Acme Q3 review finished on 28 October.

Open and on track
- Acme renewal 2027: at "Partner sign-off", due 8 November.
- …
```

- Use plain words, not codes: "in progress", not `IN_PROGRESS`; "step finished",
  not `step.completed`.
- Give each late step its assignee and how late it is.
- Name each flow so the user can find it in Moxo. Where the client can open MCP
  resources, link it as `[Flow name](moxo://flows/{id})`.
- If the report is going to the client, ask before including internal users' names
  or internal steps.

This skill doesn't send anything. If the user wants to share the report, they copy
it.
