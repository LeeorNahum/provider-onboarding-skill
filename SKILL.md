---
name: "provider-onboarding"
description: "Run an interactive external-provider setup loop for a web product, wiring auth, DNS, hosting, backend, billing, storage, and email across local, staging, and production stages and filling every environment value into its correct store. Use when onboarding providers, filling env files, configuring dashboards, or standing up a new deployment stage."
disable-model-invocation: true
metadata:
  author: "Leeor Nahum"
  version: "2.6.1"
---

# Provider Onboarding

Run after a repo is already built. The job is purely operational: configure every provider dashboard and CLI, across auth, DNS, hosting, backend, billing, storage, and email, and get every environment value into its correct store so the product actually runs. Architecture and opinions were decided when the repo was built; do not reopen them here.

You are a setup guide, not a feature builder. Do not redesign the app, add UI, or write product code during onboarding.

## Source Of Truth

The repo as it stands is the spec. Before touching anything, read every committed `.env.example` and the code that reads env to learn the exact set of keys and which runtime owns each. Fill those keys. Do not invent, rename, or re-debate them.

Each runtime that reads env owns its own contract: a committed `.env.example` template and a filled `.env.local` beside it in the repo. Use the same key names across stages; only values change by store.

## Provider Shape And Naming

Before provisioning a provider, check its current official documentation and dashboard model for the intended way to isolate development, staging, and production. Prefer the provider's native environments, modes, sandboxes, deployments, or credential scopes when they provide the required isolation inside one canonical project. Create separate projects or accounts only when the provider requires them or when its native model cannot keep stages independently configurable and safe.

Map the product stages onto the provider's actual model instead of assuming that every provider needs one project per stage. Record the mapping before creating or filling resources.

Use the product's canonical name for the provider project, application, workspace, or account when the provider accepts it. Preserve normal capitalization and spaces when supported. Adapt only as required by the provider, progressively applying constraints such as removing spaces, restricting characters, lowercasing, adding a purpose suffix, or creating separate stage-specific names. Check availability before falling back to a less canonical name.

## Provider Resource Metadata

Fill optional human-readable descriptions, labels, and tags whenever a provider resource supports them. State the product, the resource's operational purpose, and its stage or shared scope so a person or agent can identify it without reconstructing setup history.

- Use concise descriptions that explain what the resource enables and which stage it serves.
- Use provider-native tags or labels for product, environment, purpose, and management source when those fields exist.
- Keep terminology consistent with the canonical product, surface, runtime, and stage names already used by the repo.
- Apply the DNS comment format below to DNS records; it is the exact special case for that resource type.

## Two Stages, Three Branches

- Dev/test credentials are used for both `dev` (local) and the `preview` branch (staging deploy); they usually share the same secret values, but things like origin URLs or certain per-stage endpoints may differ as needed.
- Production credentials serve `main` only.

Keep dev/test and production resources or credentials isolated through the provider's intended mechanism. Never build `LOCAL_*`, `DEV_*`, `PROD_*` key triples.

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

Configure each provider as completely as its known dependencies allow, across dev/test and production, while its dashboard and setup context are already open.

- Work one provider or one small sub-step at a time. Within that provider, configure its dev/test tier first, then its production tier, using the provider's native environments, modes, deployments, or credential scopes.
- Complete production provisioning and configuration when the required values are already known, even if end-to-end production verification will happen later. Keep live activation actions behind their existing approval gates.
- Order integrations by dependency truth. Establish canonical projects, deployments, domains, and origins before creating credentials, allowlists, callbacks, webhooks, or redirect URIs that refer to them. When an upstream value is not available yet, defer the dependent setting instead of entering a temporary value that will need retroactive repair.
- Before moving to the next provider, finish every actionable setting across its applicable tiers, including keys, native integrations, OAuth credentials, origins, roles, callbacks, webhooks, and dashboard state. Record each remaining deferral with its blocker and the exact later step that unblocks it.
- Fill local env and push the backend dev deployment as soon as the dev/test provider values are ready, so local verification can proceed while production configuration is preserved for later verification.
- Push the repo to its remote before wiring a host that deploys from that remote, since git-connected branch deploys need the remote to exist.
- For each hosted stage, provision its credentials, fill its env, then add its domain and DNS records as soon as that stage has an origin to point at.
- Treat a stage as done only when its dashboard state is set too, such as allowed origins, callbacks, webhooks, and branch deploys, alongside its keys.
- When a host offers human-facing deployment protection or an SSO login wall in front of deployments, disable or bypass it for any surface that machine clients call directly, such as public APIs, MCP servers, and webhook receivers. That wall answers non-browser callers with an HTML login page, which silently breaks their OAuth discovery, dynamic client registration, and requests, and the actual endpoint call hangs until it times out. Surfaces that carry their own token or OAuth auth do not need it; keep the wall only on human-only surfaces, and check this per stage, since preview and production scope it independently.
- When a later step changes an earlier value, update every store that holds it in the same pass.

Start from this order, and adapt it when a provider forces a different sequence.

## DNS

Treat DNS as part of the runtime contract. Records, proxy mode, and comments are all intentional.

Proxy mode:

- Keep third-party service records as DNS only (unproxied): hosting platform CNAMEs, auth provider frontend and portal CNAMEs, and email authentication records. Proxying breaks certificate issuance and email signature verification for these services.
- Proxying is fine for A and AAAA records pointing at your own origin servers.

Email authentication record names are literal, including underscores and dots. Enter them exactly as the provider shows them.

Every record carries a comment in one format:

```text
Provider - Product Purpose (Scope)
```

- Provider is the service the record points at or is verified by.
- Product Purpose is the product plus what this specific record is for: the surface it serves, the connection it enables, or the verification it proves.
- Scope is the deployment stage in parentheses. Include it on any record that serves a single stage, production included. Leave it off only when one shared record is reused unchanged across every stage.

So a single record reads as one phrase that says who owns it, what it does, and which stage it belongs to. Apply the comment to every record created during domain, verification, or provider setup, so the zone stays self-documenting.

Ask before changing any production DNS record.

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
