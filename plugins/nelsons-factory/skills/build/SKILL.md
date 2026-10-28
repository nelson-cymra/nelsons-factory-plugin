---
name: build
description: Use when the user wants something built by Nelson's Factory instead of locally, or wants to answer or reply to one of its jobs - "/nelsons-factory:build ...", "factory: ...", "get the factory to ...", "send this to the factory", or a feature or fix for one of their factory projects that should go through the factory's tools.
---

# Nelson's Factory: build

## Overview
The factory (the `factory` connection from this plugin) builds, checks, reviews and **merges to the
project's main branch by itself**, so the brief you send is the only thing it knows. This skill turns
a one-line idea into a grounded brief (questions one at a time, play back), then submits it. The
user never has to explain the process.

## The recipe
1. **Project.** Call `factory_projects` first, before anything else. It lists the projects (the
   GitHub repos connected to the factory) the user can send work to, their role, the areas their jobs
   may change, and whether each is ready. The project is always one of these, never inferred from
   the folder Claude Code is running in.
   - The user named one of them in their request: use it.
   - Only one: use it, and say which in your first message.
   - Several: ask which, as your first question, with the AskUserQuestion tool. One option per
     connected project, labelled with the repo name, its description saying whether it's ready (and
     why not, when it isn't). Wait for the answer before recon.
   - None: the factory isn't installed on any repo they can push to. Say so and point them to the
     dashboard, https://nodejs-1683750-6753929.cloudwaysnodeapps.com, to connect a repo.
   - The brief must fit the user's areas (everything, for an owner).
   - If the tools are deferred, load them first. If they aren't connected or need sign-in, tell the
     user: run `/mcp`, choose `plugin:nelsons-factory:factory`, and sign in with GitHub in the browser.
2. **Recon, before your first question, without asking.** Read the code the factory will build on,
   the project's default branch:
   - If the current folder is a checkout of the project, `git fetch` and read with
     `git show origin/<branch>:<path>`.
   - Else, if `gh` is installed and signed in, `gh api repos/<repo>/contents/<path>`.
   - Else say you couldn't read the code, and ask a little more instead.

   Find the page or route, its components, the data models and fields involved, existing helpers,
   and where the tests live. Read only; change nothing.
3. **Brainstorm.** First message after the project is settled: one line ("say *send it* any time and
   I'll submit what we have"), then what you found in a sentence or two, then **one** question
   grounded in it (name the real
   field, screen or rule). One question per message after that; open questions first, a short menu
   only when they're stuck. Usually 1-3 questions.
4. **The brief is your playback.** When the open points are settled, show it exactly as it will be
   sent, and end with: *Reply **send** to submit, or tell me what to change.*
5. **Submit on approval** ("send", "yes", "go", "looks good"), with no further question:
   `factory_submit(project, title, body)`. That opens the issue labelled for the factory, and the job
   starts by itself. Report the issue link and how to follow it (the board at
   https://nodejs-1683750-6753929.cloudwaysnodeapps.com, or `factory_status`). Then stop: no polling
   unless asked.

If the user says "send it" / "skip the questions" at any point, go straight to step 4 with what you
have, and fill the gaps with stated defaults under **Decisions**.

## Replying to the factory
When the user wants to answer a job or tell a stopped job what to do ("tell job 49 its checks can
run now", "answer the factory on the expiry report: use the batch's own date"), don't write a brief:
1. Find the job: `factory_status` (or `factory_job` with its id or issue number). If more than one
   job could be meant, ask which.
2. Read why it's waiting, with `factory_job`: a question (stage **needs-info**) or a stop for a
   person (stage **needs-human**, with the reason).
3. Send the user's words, without rewriting their meaning:
   - **needs-info:** `factory_answer(project, issue, text)`, and the job carries on.
   - **needs-human:** `factory_rework(project, issue, note)`, and the builder picks the job up
     again with the note.
   - **needs-human, nothing to change** ("merge it", "Actions are back on"; the reason says
     GitHub didn't run CI or refused the merge): `factory_merge(project, issue)`. The factory
     runs the checks and CI again on the commit it already passed and merges if they pass.

   If what they said doesn't fit what the job is waiting for, say so and ask before sending.
4. Report the factory's reply in one line. The reply is posted on the job's GitHub issue as them.

## The brief
**Title:** imperative, under 70 characters, names the screen ("Stock page: flag batches near expiry").

**Body**, these sections in this order:
- **Why**: the user's goal in their words.
- **Where things are today**: real paths, models and fields from recon, plus the commit you read.
- **What to build**: numbered behaviours, including empty and edge cases (no data, zero, timezone).
- **Acceptance criteria**: checkable statements, including the tests that must exist.
- **Out of scope**: what must not change.
- **Decisions**: each call the user made, and each default you chose.

## Conditions
- **Several independent changes:** one brief per change, each submitted in order. The factory runs
  a couple of an account's jobs at once (one at a time when they touch the same folders) and queues
  the rest.
- **A bigger idea that takes several steps:** plan it as a chain of small jobs (one screen or one
  behaviour each; a data-model change is its own first job).
  1. Before any brief, show the plan: numbered jobs, one line each, and which one each builds on.
     Ask your one question about the split.
  2. Then show every brief. Each later brief's **Where things are today** describes the main branch
     after the jobs before it ("after job 1: `Batch.expiryDate` exists").
  3. On **send**, submit them in order. Each dependent brief ends with a line `Depends on: #<issue>`,
     or `Depends on: #201, #203` for several, using the issue numbers the earlier `factory_submit`
     calls returned. The factory won't start a job until everything it depends on has merged, and
     says so on its issue.
- **Touches the data model, sign-in, permissions or dependencies:** the preview says so in one line.
  Unless the project's owners set critical paths, the factory merges these by itself too.
- **Needs files outside the user's areas:** say which ones and stop. That's an owner's call.
- **The idea isn't buildable yet** (no data to show, an unsettled business rule): say what's
  missing, offer a first brief that makes it possible, and submit nothing until it's agreed.

## What this replaces
When this skill runs, step 5 is the end of the conversation. The factory does its own intake, spec,
plan, build and review, so these endings don't apply:

| The usual ending | Here |
|---|---|
| Write and commit a spec document | The brief lives in the issue body |
| A separate review of the plan | The factory's intake and spec stages do that |
| Write a plan or build it locally | `factory_submit` hands it off |
| Ask before looking at the code | Recon always happens; it's read-only |

## Common mistakes
| Mistake | Fix |
|---|---|
| Calling `factory_start` after submitting | `factory_submit` already starts the job |
| Reading a stale local branch | Read `origin/<default branch>`, which is what the factory builds on |
| Guessing field names in the brief | Use the names recon found, or ask |
| A vague acceptance criterion ("works well") | Make it checkable: input, then the expected result |
