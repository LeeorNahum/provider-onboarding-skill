---
name: "provider-onboarding"
description: "Run an interactive external-provider setup loop for a web product, wiring auth, DNS, hosting, backend, billing, storage, and email across local, staging, and production stages and filling every environment value into its correct store. Use when onboarding providers, filling env files, configuring dashboards, or standing up a new deployment stage."
disable-model-invocation: true
metadata:
  author: "Leeor Nahum"
  version: "2.2.0"
---

# Provider Onboarding

Run after a repo is already built. The job is purely operational: configure every provider dashboard and CLI, across auth, DNS, hosting, backend, billing, storage, and email, and get every environment value into its correct store so the product actually runs. Architecture and opinions were decided when the repo was built; do not reopen them here.

You are a setup guide, not a feature builder. Do not redesign the app, add UI, or write product code during onboarding.

## Source Of Truth

The repo as it stands is the spec. Before touching anything, read every committed `.env.example` and the code that reads env to learn the exact set of keys and which runtime owns each. Fill those keys. Do not invent, rename, or re-debate them.

Each runtime that reads env owns its own contract: a committed `.env.example` template and a filled `.env.local` beside it in the repo. Use the same key names across stages; only values change by store.

## Two Stages, Three Branches

- Dev/test credentials are used for both `dev` (local) and the `preview` branch (staging deploy); they usually share the same secret values, but things like origin URLs or certain per-stage endpoints may differ as needed.
- Production credentials serve `main` only.

Create separate provider credentials per stage when the provider supports it. Never build `LOCAL_*`, `DEV_*`, `PROD_*` key triples.

## Where Filled Values Live

The committed `.env.example` is the template. Its filled counterpart is `.env.local`, which lives in the repo right beside it and is gitignored. Fill `.env.local` as a one-to-one copy of the example with real values. This is the default home for local values.

Values also destined for provider dashboards or deployed stores need filled copies to paste. By default these copy-paste files live in a folder the user picks outside every git repo, so sensitive filled values never sit in a git tree. Ask the user to choose that folder at the start, and write them only there.

Name each copy-paste file `.env.<destination>.tmp` so the standard env gitignore catches it and it reads as a temporary env dotfile. Each is a one-to-one copy of the matching `.env.example`, filled for a single destination, so it pastes in one action with no per-key hunting.

```text
in the repo, beside each .env.example:
  .env.local                            filled local values, gitignored

in the chosen folder outside any repo:
  .env.<backend>-dev.tmp                paste into the dev backend deployment
  .env.<backend>-prod.tmp               paste into the prod backend deployment
  .env.<host>-staging-<surface>.tmp     paste into the host project, staging scope
  .env.<host>-production-<surface>.tmp  paste into the host project, production scope
```

This split is the default unless the user says otherwise.

## Order Of Operations

Stand each stage up in dependency order so a value gets filled once it is known.

- Make the product work locally first: fill local env, push the backend dev deployment, and confirm the critical paths before standing up any hosted stage.
- Push the repo to its remote before wiring a host that deploys from that remote, since git-connected branch deploys need the remote to exist.
- For each hosted stage, provision its credentials, fill its env, then add its domain and DNS records as soon as that stage has an origin to point at.
- Treat a stage as done only when its dashboard state is set too, such as allowed origins, callbacks, webhooks, and branch deploys, alongside its keys.
- When a later step changes an earlier value, update every store that holds it in the same pass.

Start from this order, and adapt it when a provider forces a different sequence.

## The Loop

Work one provider or one small sub-step at a time. Never dump the whole checklist.

For each step, write in the message body: the goal and the exact clicks or command, then the precise paste handoff (`KEY=value` lines) or a note that there is nothing to paste this step. Then ask a short status check.

If the harness has a dedicated question tool or inline answer UI, use it for that status check so the user can answer in place and you can continue immediately in the same flow. Keep the full context in the message body, not inside the question; keep the question itself short, such as done, blocked, or in progress, and let the user pass values or errors through the free-form answer.

When the user returns values:

- Write them at once into the repo `.env.local`, the matching copy-paste files, and any provider CLI that applies.
- Confirm by key name and store only. Never echo a secret value back.
- Gate the next step on that confirmation or an explicit deferral.

Do not end the loop or stop early. Keep going until every key in every contract is filled or explicitly deferred by the user.

## What The Agent Does Itself

Do as much as the provider CLI allows, and hand the rest to the user as dashboard steps.

- Agent terminal: reactive backend session, env set, codegen, and deploy checks; host link and env push when the project is linked; repository and CI value setup where a CLI exists.
- User dashboard: auth, OAuth, allowed origins, object storage buckets and tokens, billing products and webhooks, API keys, domains, and branch deploys.

Before you run a command or fill a value, know where it lands, how it takes effect, and why, so you target the right store and stage in one pass. Pass explicit scope flags such as stage, branch, environment, team, or project so a command acts on the intended target without prompting.

If a CLI step needs interactive input, either ask the user for the one missing value and then run the non-interactive form, or hand them the exact command for their own terminal.

## Deferrals

Some values need a public origin that does not exist until the first staging deploy, such as a webhook target or allowed origins. Defer those explicitly, note which later step unblocks each, and keep a running list. Do not invent a placeholder to skip the wait.

A production provider on a custom domain issues records that must resolve before its keys work. Add those records, then defer each dependent step until the provider reports the domain verified.

## Finish

Confirm the stage actually works before calling it done: install dependencies, run backend codegen and push, run a typecheck across packages and surfaces, build each deployable surface, and smoke-test the critical paths. Report any failure by missing key name and store, never by value.

## Cleanup

When setup for a stage is complete, remind the user to remove the working copy-paste files from the chosen folder. Never commit a real or filled env file.

## Ask First

Ask before pointing staging at production resources, before any production DNS change, before enabling a live billing mode, and before any production deploy.
