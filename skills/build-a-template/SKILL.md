---
name: build-a-template
description: >
  Creates or changes a Moxo template (a workflow): checks what the user may build
  and what is on their plan, builds the draft in small batches, checks it after
  each batch, shows the outline and a link, offers a test run, and publishes only
  after an explicit yes. Use when the user says 'build this in Moxo', 'add an
  approval step to the invoice approval template', 'change the deadline on the
  vendor review', 'try the client onboarding template', or 'publish it', or after
  design-a-process ends with a yes. Not for working out a new process from
  scratch (use design-a-process first), not for starting real work from a
  published template (use start-flow-for-client), and not for checking on flows
  (use morning-triage or flow-status-report). Never publishes without a yes, and
  never asks for a password, key or token.
---

# Build a template

A template is the design of a process: who takes part (roles), what each of them
does (actions), the decisions, the deadlines and the apps it uses. This skill
builds one as a draft, checks it, lets the user try it, and publishes it on their
word.

A personal access token builds as its person, with that person's permissions.
An organization API key builds like an admin, and its templates belong to the
organization's service account. A test run needs a personal access token,
because a person walks through the steps.

## 1. Get oriented

Call `get_organization_overview` first, every time.

- **What this person may do.** If they can't create templates, or can't change
  this one, or can't publish, say so and stop at what they can do.
- **What the plan includes.** Use only action types the organization has. Never
  plan one that isn't available. If the design needs one, say so and offer
  another way to do that part.
- **What already exists.** Connected apps, secrets and AI agents come back by
  name. Use those names in the template.

For each action type you will use, call `get_action_type` with its `type` (and
`app` for an action on a connected app). It returns the settings, a working
example and common mistakes. Follow the settings over any other description. Call
`get_guide` when you need how something works, such as branching or deadlines.

## 2. Confirm before the first write

Show a short outline of what you will build and ask for a yes. If the user just
said yes to a design from `design-a-process`, that yes covers creating the draft.

A draft reaches nobody. Still, each write changes something in Moxo, so don't
create or change a template the user didn't ask for.

## 3. Build in small batches

- **New template.** Call `create_template` with a `name`, a short `description`
  and the first batch of changes: the roles and the first few actions. To start
  from a `find_examples` result, pass its id as `fromExample`.
- **More changes.** Call `change_template` with the next batch and `baseVersion`
  set to the draft version from the last result.
- Keep each batch to a few related changes, so a problem is easy to place.
- A batch is all or nothing. If it is refused, nothing changed: read each issue,
  fix the batch and send it again.

Each result has a `status` (`ready`, `has_issues` or `published`), the new draft
version, the ids for the short names you gave, what changed, `fixedForYou`,
`issues`, `needsAPerson`, links and an outline.

## 4. Check after each batch

Call `check_template`. It saves nothing. New problems come back in `issues`;
problems that were already in the template come back in `existingIssues`.

- Fix each issue in `issues`. When an issue comes with a suggested change, use
  it.
- `existingIssues` are the user's call. Mention them, but don't fix them unless
  the user asks.
- Tell the user in a line what `fixedForYou` adjusted, so nothing changes
  silently.
- Move on to the next batch once the check shows no new issues.

## 5. Secrets and things only a person can do

- Never ask for, accept or pass on a password, API key or token. Refer to a
  secret or a connected app by the name `get_organization_overview` gave. If the
  user pastes a secret into the chat, don't use or repeat it: ask them to add it
  in Moxo instead, and suggest they change it since it has been shared.
- `needsAPerson` lists what only a person can do in Moxo: connect an app, add a
  secret's value, place signature fields on a document. Pass each one to the
  user with its link. Don't try to do them another way.
- **Documents.** To attach a file from the user's computer, call
  `get_upload_link` and run the command it returns. The file goes straight to
  Moxo and never passes through the chat. Then attach it with `change_template`
  using the file id it returned. Ask before uploading.

## 6. Show what you built

After building, always show:

- the outline, from the last result or from `get_template_outline`
- the link to open the draft in Moxo
- anything in `needsAPerson`, each with its link
- an offer to try it: "Want a test run? You'll get the emails, marked [TEST]."

`start_test_run` runs the draft as a practice flow. Every role you don't name is
assigned to the user, so they see each step as it arrives. Test runs stay out of
live work and reports. Ask before starting one, and say who will get emails if
you name other people.

## 7. Refine by chatting

The user will want changes after seeing it or trying it. For each one: make the
change with `change_template`, run `check_template`, and show what changed. Show
the full outline again when the shape changed.

## 8. Publish only on a yes

Publishing makes the draft the version every new flow uses, and it is recorded
under the user's name.

- Run `check_template` first. Publish only when the status is `ready`.
- Ask: "Publish Vendor onboarding? New flows will use this version." Only a clear
  yes counts: "yes", "publish it", "go ahead".
- Call `publish_template`, then share the link.
- To start real work from it, continue with `start-flow-for-client`.

## Changing an existing template

1. Find it with `list_templates`. If several match, ask which.
2. Call `get_template_outline` first. Use its ids in your changes, and its draft
   version as `baseVersion`.
3. Confirm the change in plain words, then make it, check it and show it as
   above. Changes go to the draft; the published version keeps running until the
   user publishes.
4. If `change_template` is refused because the template changed since you read it,
   someone else edited it. Call `get_template_outline` again, rebuild your change
   on the new version, and tell the user if their change now clashes with
   someone else's.
5. To throw away unpublished changes, `discard_draft` returns the draft to the
   published version. It can't be undone, so ask first. It keeps other people's
   changes unless the user agrees to discard those too.

## Outside the template

Some settings sit around a template rather than in it. Each is a write, so
confirm first.

- `manage_template_triggers`: how flows start on their own: schedules, app
  triggers, a start by email or by webhook, and which webhook fields fill which
  template fields (try the mapping with a sample before saving it). On a
  published template these start real flows, so say so when you confirm.
- `manage_template_settings`: owners, folder, mode, and chat support with its
  FAQs.
- `manage_agent`: the AI agents the template uses: list, create, update,
  publish, and add knowledge.
- `manage_data_table`: find, read or create the data tables the template uses.

## When it fails

- **No access:** a personal access token acts as the user. If they can't build
  or publish this template in Moxo, the agent can't either. An organization API
  key builds like an admin but can't start a test run; ask for a personal access
  token for that.
- **Action type not available:** it isn't on the organization's plan. Say so and
  offer another way to do that part.
- **Timeout or unclear result:** the change may have gone through. Call
  `get_template_outline` and look before sending it again.
