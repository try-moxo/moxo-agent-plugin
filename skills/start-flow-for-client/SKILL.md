---
name: start-flow-for-client
description: >
  Starts a Moxo flow for a client or person: finds the person, picks the right
  template, fills every role and start field, shows a summary, and calls
  start_flow only after the user says yes to it. Use when the user says 'start
  onboarding for Acme', 'kick off a flow for Jane Doe', 'send the KYC flow to this
  client', or 'run the tax return template for the Patels'. Not for checking on
  flows that already exist (use morning-triage or flow-status-report), and not for
  building or editing templates. Never starts a flow without an explicit yes.
---

# Start a flow for a client

The Moxo server has a `/start_flow_for` prompt that finds a person, suggests a
template and confirms. This skill follows the same path and adds what a real start
needs: it reads the template so every role and required start field is filled, it
uses only published templates, and it shows the full summary before anything
starts.

Starting a flow is real. It can send emails or texts to the people on the first
steps, and it can't be undone from here (a flow can only be cancelled after).

## 1. Find the person

- Call `list_external_users` with `search` set to the name, email or phone the user
  gave. If you can't tell whether a name is a person, a template or a flow, call
  `search` instead; each hit has a `type`.
- One clear match: use it, and show the name and email so the user can catch a
  wrong match.
- Several matches: show up to three with email and title, and ask which one.
- No match: ask whether to add them. Adding is a write, so call
  `create_external_user` only after a yes. It doesn't send a portal invite.
- Internal people for other roles come from `list_team_members`.

Never pick between people by guessing.

## 2. Pick the template

- Call `list_templates`. Match the user's words against each template's `name` and
  `description`.
- Only a template with `status` `ACTIVE` can start. If the best match is a draft or
  archived, say so: it has to be published in Moxo first.
- If more than one template fits, list them and ask. If the user named a template
  that doesn't exist, show the closest ones.

## 3. Read the template

Call `get_template` with the `templateId`. It lists the roles, the start fields and
the steps.

- **Roles.** Put the client in the role that faces the client (often called
  "Client"; read the role names, don't assume). For every other role, ask who fills
  it unless the user already said. Use role names exactly as `get_template` shows
  them.
- **Start fields.** Ask for every required field the user hasn't given. Never
  invent a value. Use ISO dates (`2026-11-03`).
- **Flow name** is optional. Moxo picks one if you leave it out; suggest one that
  names the client, such as "Acme Corp onboarding".

## 4. Confirm

Show one summary and ask one question:

```
Start this flow?

Template:  Client onboarding (v3)
Name:      Acme Corp onboarding
Client:    Dana Lee (dana@acme.com)
Reviewer:  Sam Ortiz
Start date: 2026-11-03

Dana Lee gets an email for the first step as soon as it starts.
```

Only a clear yes counts: "yes", "start it", "go ahead". A question, a change, or
"looks good but…" is not a yes. If anything changes, show the whole summary again
and ask again.

## 5. Start

Call `start_flow` with:

- `templateId`
- `name`, if you have one
- `fieldValues`, keyed by each field's `key` from `get_template`
- `roleAssignments`, keyed by role name. Each value is
  `{ "kind": "contact", "id": "usr_…" }` for an external person or
  `{ "kind": "user", "id": "usr_…" }` for a team member. Pass each `id` exactly as
  `list_external_users`, `list_team_members` or `search` returned it. A role that
  holds several people takes an array.

Then report the new flow's name and what happens next.

## When it fails

- **Unknown role name:** read `get_template` again and use the role names exactly.
- **Template not published:** it has to be published in Moxo before it can start.
- **No access:** the token acts as the user, so if they can't start this template
  in Moxo, the agent can't either.
- **Timeout or unclear result:** the flow may have started. Before trying again,
  call `list_flows` with the `templateId` and `startedAfter` set a few minutes back,
  and look for it. Never start twice to be safe.
