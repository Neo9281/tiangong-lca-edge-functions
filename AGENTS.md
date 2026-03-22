# AGENTS.md

## 1. Purpose

This file defines how AI coding agents should work in this repository.

Use `AGENTS.md` for:

- repo-level workflow rules
- validation requirements
- documentation sync rules
- directory and test placement conventions
- deployment guardrails

Do not use `AGENTS.md` as the main place to document detailed function behavior. Human-facing runtime semantics, request/response examples, and deploy command lists belong in `README.md`.

## 2. Source of Truth Boundaries

- `AGENTS.md`
  - AI workflow rules and non-negotiable repo conventions
- `README.md`
  - local setup
  - environment preparation
  - function behavior and request examples
  - deployment and remote config commands
- `supabase/functions/*` and `supabase/functions/_shared/*`
  - runtime source of truth
- `test/` and `test/_shared/`
  - test source of truth

If `AGENTS.md` and `README.md` start repeating the same operational detail, keep the concise rule in `AGENTS.md` and move the detailed explanation to `README.md`.

## 3. Repo Snapshot

- Runtime: Supabase Edge Runtime (`Deno 2.1.x`) plus a small Node toolchain
- Functions root: `supabase/functions`
- Shared runtime modules: `supabase/functions/_shared`
- Test root: `test`
- Shared helper tests: `test/_shared`
- Request collection: `test.example.http`
- LCA integration helper: `scripts/lca_submit_poll_fetch.sh`
- Dependency mapping is pinned in `supabase/functions/deno.json`; keep imports exact unless there is a deliberate upgrade

## 4. Directory Conventions

- Runtime code belongs under `supabase/functions/*` and `supabase/functions/_shared/*`.
- Tests do not belong in runtime directories unless there is a very strong repo-specific reason.
- Shared helper tests should live in `test/_shared/`.
- Manual request samples and fixture payloads should live under `test/` or `test/fixtures/`, not the repo root.
- Human-facing operational guides belong in `README.md`, not in ad hoc files unless the topic is large enough to justify a dedicated document.

## 5. Working Rules

- Prefer changing shared modules over copying logic into multiple functions.
- Keep edits scoped to the task; avoid drive-by refactors.
- Do not introduce new dependencies unless necessary; if you do, update both `README.md` and `AGENTS.md` when workflow expectations change.
- Preserve exact import pinning style in `supabase/functions/deno.json`.
- Never expose secrets, tokens, or full connection strings in code comments, docs, logs, commits, or responses.
- If you touch `_shared` behavior, think through all direct dependents before finishing.

## 6. Required Validation

After any repository change, run:

```bash
npm run lint
```

Additional rules:

- If you changed files under `supabase/functions`, `deno check` is mandatory.
- For a single function change, check the changed file directly:

```bash
deno check --config supabase/functions/deno.json <changed-file>
```

- For a shared module change, also check every directly dependent function.
- For test-only or fixture-only changes outside runtime directories, `npm run lint` is the minimum; add `deno check` or `deno test` when imports or executable examples changed.

## 7. Validation Matrix

- Search / hybrid search changes:
  - `deno check` the changed function plus directly used shared modules
  - prefer a smoke test from `test.example.http`
- LCA changes:
  - `deno check` the touched file(s)
  - if `_shared` LCA modules changed, check every directly dependent LCA function
  - prefer `scripts/lca_submit_poll_fetch.sh` for end-to-end verification when behavior changed materially
- TIDAS package changes:
  - `deno check` the touched function(s) and `tidas_package.ts` dependents when shared logic changed
- OpenAI shared wrapper changes:
  - check the touched wrapper and direct dependents
  - verify the intended OpenAI call path still type-checks

## 8. Documentation Sync Rules

Update `README.md` when changes affect:

- local setup
- env vars or secrets workflow
- function behavior or request examples
- deployment steps
- test/demo entry points humans are expected to run

Update `AGENTS.md` when changes affect:

- repo workflow expectations
- validation rules
- directory conventions
- dependency management policy
- deployment guardrails

Avoid putting long per-function feature descriptions into `AGENTS.md` unless the detail is a repo-wide invariant that materially changes how an agent should edit the code.

## 9. Deployment Guardrails

- Local serve entrypoint:

```bash
npm start
```

- Before deployment, finish the required validation for the touched area.
- Secrets sync and remote deploy commands live in `README.md` under `Remote Config`; treat that section as the deploy command source of truth.
- If you changed human-facing behavior or deploy scope, update `README.md` before or together with the deployment change.
- If you changed a shared runtime module, do not assume one function deploy is enough; identify every dependent function that needs redeploying.
- Be careful with `supabase secrets set`; it mutates remote project configuration.

## 10. Response Expectations

When reporting completed work, state:

1. which files changed
2. which commands were run
3. which checks passed or were not run
4. any follow-up deployment or integration risk
