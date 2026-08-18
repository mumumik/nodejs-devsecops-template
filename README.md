# Node.js DevSecOps Template

A template repo for Node.js services with a security pipeline (SAST + SCA + DAST)
via Veracode already set up. The goal: new teams just click **"Use this
template"**, push code, and the security pipeline runs right away — no need
to set up SAST/SCA/DAST from scratch.

## Pipeline structure

| Workflow | Trigger | Scan | Behavior |
|---|---|---|---|
| `pr-quick-scan.yml` | Every Pull Request | SAST (Pipeline Scan) + SCA | Fast (~2-5 min), feedback on the PR, does not change the application's compliance status on the Veracode Platform |
| `main-security-scan.yml` | Push to `main`, every Monday, or manual | SAST (Policy Scan) + SCA | Full scan, uploads to the Veracode Platform, updates the application's compliance status |
| `dast-scan.yml` | Manual, or every Wednesday | DAST (DAST Essentials) | Attacks a live staging URL — don't point it at production |

Why separate the pipeline scan (PR) from the policy scan (main)? The pipeline
scan doesn't need an application profile and doesn't affect compliance
status, so it's safe to run repeatedly on every PR without "polluting" the
official scan history. The policy scan is what becomes the application's
official compliance record.

## What teams using this template need to do

If org-level secrets have already been set up by the admin (see
`docs/SECRETS.md`), **there's nothing left to configure**. If not, this is
the only manual step per repo:

1. Settings → Secrets and variables → Actions → add:
   - `VERACODE_API_ID`, `VERACODE_API_KEY` — Veracode API credentials
   - `SRCCLR_API_TOKEN` — token for agent-based SCA
   - `VERACODE_DAST_WEBHOOK_SECRET` — only if this team uses DAST
2. Settings → Secrets and variables → Actions → Variables → add:
   - `VERACODE_APP_NAME` — the application profile name on the Veracode
     Platform (create the profile first if it doesn't exist yet, or set
     `createprofile: true` once in `main-security-scan.yml` to auto-create it)

Ideally the steps above are done once at the **organization** level, not per
repo — see `docs/SECRETS.md` for how to set that up so other teams get true
zero-config.

## Running locally

```bash
npm install
npm test
npm start
```

## Important note about secrets in the workflow

All secrets in this workflow are passed via the `env:` block on each action's
step (not substituted directly into the `run:` shell), and consistently use
`secrets.*` for confidential values and `vars.*` for non-confidential values
like the application name. This avoids exposing secrets in logs if someone
mistakenly puts a secret under `vars.*`, or messy shell parsing caused by
direct interpolation in `run:`.

## Adapting for other services (not Express)

- `pr-quick-scan.yml` and `main-security-scan.yml` only need a `zip` of the
  source code — change the "Package artifact" step to match your project's
  structure.
- `veracode-pipeline-scan-action` supports JavaScript/TypeScript, Java,
  Kotlin, Scala, Groovy, and Android without needing to compile first for JS/TS.
- If using a monorepo, add `working-directory` to the checkout/build steps,
  and adjust `filepath` in the packaging step.

## Advanced: further centralization with reusable workflows

If the number of team repos grows large and you want to maintain these
workflow files in one place (instead of copy-pasting per repo), the three
workflows above can be moved into a *reusable workflow* (`workflow_call`) in
a central repo, so each team repo only needs a few-line caller file that
invokes
`org/central-devsecops-workflows/.github/workflows/nodejs-security.yml@main`.
This template intentionally isn't set up that way yet, so it's easy to
understand and can be tested by one team first before centralizing.
