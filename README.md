# Provider Onboarding

A lean, manual-invoke skill that fills a web product's environment and configures its providers after the repo already exists. The repo's current code and committed `.env.example` files are the spec; this skill just gets every value into the right store and wires each dashboard and CLI.

Run it deliberately when standing up providers or a deployment stage. It first checks each provider's intended environment model and naming constraints, preferring native development, sandbox, and production isolation inside one canonical project when that is how the service works. It completes each provider across the tiers whose dependencies are known, sequencing canonical domains and origins before credentials, callbacks, and webhooks that depend on them. It then drives a tight one-step-at-a-time loop: the agent explains a step, you act in the dashboard, you pass values back, and the agent writes them to the right files and confirms by key name only. DNS records include intentional proxy modes and self-documenting comments. Local values fill `.env.local` beside each `.env.example` in the repo. Values bound for provider dashboards or deployed stores are kept as ready-to-paste copies in a folder you choose outside every git repo, one per destination, so each pastes in one action. The loop does not stop until every key is filled or deferred.

## Layout

- `SKILL.md` holds the whole workflow: source of truth, stages, where filled values live, the loop, what the agent does itself, deferrals, finish, and cleanup.

It is a single self-contained file by design, with no references.
