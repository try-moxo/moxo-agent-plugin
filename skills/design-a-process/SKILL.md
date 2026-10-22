---
name: design-a-process
description: >
  Helps the user think through a business process before anything is built in
  Moxo: who is involved, the steps, the decisions, the deadlines, what happens
  when something is late or rejected, and which systems it touches. Then maps it
  onto what Moxo can do with search_moxo_docs and find_examples, shows a short
  design in plain words, and hands off to build-a-template after a yes. Use when
  the user says 'help me set up vendor onboarding in Moxo', 'I want to automate
  our invoice approvals', 'design a client onboarding workflow', or describes a
  process and asks how Moxo could run it. Not for small edits to an existing
  template (use build-a-template), not for starting a flow from a template that
  already exists (use start-flow-for-client), and not for checking on flows. It
  only reads and never builds without a yes.
---

# Design a process

Most processes are easier to build once they are written down. This skill talks
the process through in the user's own words first, then works out how Moxo would
run it, and only then hands off to `build-a-template`. Nothing is created in Moxo
here.

## 1. Understand the process

Talk about the work, not about Moxo. Don't use Moxo words such as "template",
"action" or "role" yet. Ask a few questions at a time, and skip what the user has
already told you.

- **Who is involved.** Inside the company and outside it (clients, vendors,
  partners). Is each one a named person or "whoever is on the team"?
- **How it starts.** Someone kicks it off, a date comes round, an email arrives,
  or something happens in another system.
- **The steps, in order.** What each person does: fill something in, upload a
  document, review, approve, sign, pay.
- **Decisions.** Where the path splits ("over $10,000 needs the finance lead"),
  and what happens on each side.
- **Deadlines.** How long each step should take, and who hears about it when it
  runs late.
- **Rejections and rework.** What happens when something is turned down or comes
  back incomplete: does it go back a step, or end?
- **Other systems.** Which tools it reads from or writes to (a CRM, an accounting
  system, shared storage), and what should move between them.
- **Done.** What "finished" looks like, and who needs to know.

Play the process back in a few lines and ask whether you have it right before
moving on.

## 2. See what Moxo can do

Make these calls once the process is clear. None writes anything.

1. `find_examples` with a one-line description of the process. Each example comes
   back as an outline with an example id. If one is close, say so: building from
   it is often faster than starting empty. Keep the id for `build-a-template`.
2. `get_organization_overview`: what this person may build, the action types on the
   organization's plan, and the apps, secrets and AI agents it already has (by
   name). Design only with what is there.
3. `search_moxo_docs` for each part you aren't sure Moxo can do ("can a step wait
   for a signed document?", "can a flow start from an email?"). Answers come from
   Moxo's own guides. When nothing is found, don't guess: say it isn't something
   you could confirm, and design around it.

If a part of the process needs something that isn't on the plan or isn't
connected, say so plainly and offer another way to do that part. Never design
around a feature the organization doesn't have.

## 3. Show the design

Keep it to one screen and in plain words.

```
Vendor onboarding: proposed design

People
- Vendor: fills in company details, uploads tax form and insurance
- Procurement (Sam Ortiz): reviews the vendor
- Finance lead: approves vendors over $10,000

Steps
1. Vendor fills in company and bank details (due in 5 days)
2. Vendor uploads tax form and insurance certificate
3. Procurement reviews: approve, or send back to step 1 with a note
4. Over $10,000: finance lead approves
5. Vendor is added to the accounting system
6. Vendor gets a welcome email

Rules
- If a step is late, remind the person, then escalate to their manager
- A rejection at step 3 or 4 ends the flow and tells the vendor why

Needs a person in Moxo
- Connect the accounting system (an admin does this once)

Starts: Procurement starts it for each new vendor
```

- Use the people's job titles or names, not system words.
- List under "Needs a person in Moxo" anything only someone in Moxo can do, such
  as connecting an app or adding a secret. Never ask the user for a password, key
  or token: those go into Moxo directly.
- If you plan to start from an example, say which one and what you would change.

## 4. Get a yes, then hand off

Ask one question: "Shall I build this in Moxo as a draft?"

- Only a clear yes counts: "yes", "build it", "go ahead". A change or a "but…"
  means update the design, show it again in full, and ask again.
- On a yes, continue with `build-a-template`. Carry over the design, the example
  id if there is one, and what you learned from `get_organization_overview`.
- Building creates a draft only. Nothing reaches anyone until the user tries it
  with a test run and then publishes it.
