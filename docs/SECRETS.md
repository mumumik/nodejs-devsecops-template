# Setup Secrets — Set Up Once, All Teams Use It

The goal of this document: so that teams creating a new repo from this
template **don't need to configure anything**. This is done once by the
org admin / platform team.

## 1. Get credentials from the Veracode Platform

| Value | Where from | Type |
|---|---|---|
| `VERACODE_API_ID` / `VERACODE_API_KEY` | Veracode Platform → Account → API Credentials (generate an API service account, not a personal account, so it doesn't break when someone leaves) | Secret |
| `SRCCLR_API_TOKEN` | Veracode Platform → Software Composition Analysis → Agent-Based Scan → generate token | Secret |
| `VERACODE_DAST_WEBHOOK_SECRET` | Veracode Platform → Dynamic Analysis → DAST Essentials → create a scan target → copy that target's webhook secret | Secret (optional, only if the team uses DAST) |
| `VERACODE_APP_NAME` | Application profile name per repo/service | Variable (not a secret) — usually set per repo, not org-level, since each service has a different name |

## 2. Set at the Organization level (not per repo)

GitHub organization settings → Secrets and variables → Actions:

1. Add a secret for `VERACODE_API_ID`, `VERACODE_API_KEY`, `SRCCLR_API_TOKEN`.
2. Repository access → choose **"Selected repositories"**, then add a
   policy so new repos created from this template are automatically
   included (or choose "All repositories" if every team uses Veracode
   anyway).
3. `VERACODE_DAST_WEBHOOK_SECRET` should stay per-repo since each service
   has a different scan target on the Veracode Platform.

With org-level secrets, a new repo created from this template can run its
pipeline immediately without the repo admin needing to touch Settings at
all.

## 3. If org-level isn't possible yet (needs approval, etc.), fall back to per-repo

Repo → Settings → Secrets and variables → Actions → New repository secret,
add the 3 secrets above one by one. This is what's briefly explained in the
main README.

## 4. How to verify without waiting for a real PR

Run `main-security-scan.yml` manually from the Actions tab → "Run workflow"
(workflow_dispatch is already set up). If it fails at the Veracode step with
an authentication error, that's a sign the credentials aren't set or are
wrong — not a bug in the workflow.

## 5. Credential rotation

Veracode API ID/Key should be rotated every few months per internal policy.
Since it's stored at the org level, rotation only needs to happen once and
automatically applies to every repo using this template — no need to update
it one by one for each team's repo.
