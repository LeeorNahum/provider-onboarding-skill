# AGENTS.md

Rules for editing the **provider-onboarding** skill. The setup workflow lives entirely in `SKILL.md`. `README.md` is the human skim layer.

## Shape

This skill is intentionally a single self-contained `SKILL.md` with no references. It is the lean, operational executor that runs after a repo exists. Keep it that way: if a detail is an opinion about how to build a web product, it belongs in the web-repository-opinions skill, not here.

## File roles

| File | Role |
| --- | --- |
| `SKILL.md` | The whole onboarding workflow |
| `README.md` | Short human summary |

## Editing

- Bump `metadata.version` with semver in the same change whenever behavior changes.
- Keep it slim. This skill earns its value by getting straight to filling env and configuring providers, not by re-teaching architecture.
- **No project leakage.** Use placeholder product, repo, and domain names. Do not embed a real product's deployments or credentials.
- **No em dashes.** Use commas, periods, parentheses, or "to".
- **Capitalized bullets.** Start every bullet with a capital letter.
- **Secret discipline.** Never write a procedure that echoes a secret back, or that stores filled values inside a git tree. Confirm by key name and store only.
- **Positive rules.** State the action to take, not the mistake to avoid.
- **Use the harness question tool.** Keep the guidance to prefer a dedicated question tool or inline answer UI for the per-step status check, with context in the body and a short question.
- **Quote every frontmatter string value.** Keys stay unquoted.

## Before finishing

- No real product, domain, or repo names introduced.
- No secret value ever echoed in an example.
- `metadata.version` bumped if and only if behavior changed.
- README matches the actual file layout and does not overclaim invocation style.
