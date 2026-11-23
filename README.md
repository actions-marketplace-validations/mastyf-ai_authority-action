<p align="center"><img src="logo.png" width="96" alt="Mastyf Authority"></p>

# Mastyf Authority Weekly Report

**Which GitHub changes should have needed a second person's approval?** Once a week this Action reads your
repositories' activity, judges each risky change the way [Mastyf Authority](https://github.com/mastyf-ai/mastyf-authority)
would, and opens an issue in a private repository of yours: force-pushes to `main`, repositories made public or
deleted, new owners, admins and deploy keys, branch protection removed.

- **Shadow mode.** It changes nothing and blocks nothing.
- **Your data stays in GitHub.** It runs on GitHub's runners in your account. Nothing is sent to Mastyf: no data, no
  telemetry, no tokens.
- **Never reads code.** It uses GitHub's activity log and repository settings only.
- **Free and open source** (AGPL-3.0). About a minute of Actions time a week.

[See it in your browser first](https://mastyf-ai.github.io/mastyf-authority/github-app/#try)

## Set up (two minutes)

1. Create a **private** repository for the reports, for example `mastyf-reports`. Reports name people and
   repositories; in a public repository the issues and the Actions logs are public too, so the Action refuses to
   post there.
2. Add [`examples/mastyf-authority.yml`](examples/mastyf-authority.yml) to it as `.github/workflows/mastyf-authority.yml`:

   ```yaml
   name: Mastyf Authority
   on:
     schedule: [{ cron: "0 6 * * 1" }]   # Mondays, 06:00 UTC
     workflow_dispatch:
   permissions: { contents: read, issues: write }
   jobs:
     report:
       runs-on: ubuntu-latest
       steps:
         - uses: mastyf-ai/authority-action@v1
           with:
             org-token: ${{ secrets.MASTYF_ORG_TOKEN }}
   ```

3. Run it once: **Actions → Mastyf Authority → Run workflow**. The first report appears a minute later; after that it
   runs every Monday.

## Cover the whole organisation (optional)

A workflow's own token sees only its repository. To cover every repository and organisation-level changes (new
owners, repositories made public or deleted, protections removed, new deploy keys and admins), add a read-only token:

1. Create a fine-grained token at <https://github.com/settings/personal-access-tokens/new>: resource owner = your
   organisation, **All repositories**, repository permission **Administration: Read-only**, organisation permission
   **Members: Read-only**. Nothing else.
2. Save it in the reports repository as the secret `MASTYF_ORG_TOKEN`
   (**Settings → Secrets and variables → Actions**).

The first report explains this if the token is missing.

## What it checks

| Change | Report says |
|---|---|
| Force-push to `main`, `master`, `release/*` or the default branch | Would have been refused |
| Repository made public, deleted or transferred | Would have waited for a second person |
| New organisation owner, repository admin or deploy key | Would have waited for a second person |
| Branch protection or a ruleset removed | Would have waited for a second person |
| Other force-pushes and deleted branches | Allowed, unless unusual for that person (judged against their last 90 days) |

Branch activity comes from GitHub's activity log, with who and when. Settings are compared with last week's, which
is stored compressed in a hidden comment in the previous report: that issue is the only storage. GitHub records who
changed a setting only in its Enterprise audit log, so for settings the report gives the week, not the person.

## Inputs

| Input | Default | |
|---|---|---|
| `org-token` | (none) | Optional read-only organisation token, see above |
| `github-token` | the workflow's token | Reads this repository and posts the report |
| `days` | `7` | Days covered by each report |
| `model` | `preview` | `preview` trains a small risk model on the runner from synthetic examples; `none` uses rules and history only |
| `allow-public` | `false` | Allow posting in a public repository |
| `dry-run` | `false` | Print the report in the log instead of posting it |

## Versions and pinning

`@v1` follows the latest 1.x release. For a fixed version use `@v1.0.0`, or pin a commit SHA. Each release installs an
exact commit of the open-source engine, named in [`action.yml`](action.yml), so a given version always runs the same
code.

## Privacy and security

Everything runs inside your GitHub account. [Privacy policy](https://mastyf-ai.github.io/mastyf-authority/github-app/privacy.html).
Report a vulnerability privately: <https://github.com/mastyf-ai/mastyf-authority/security/advisories/new>.

Want risky changes to actually wait for a second person, across GitHub, cloud consoles, data exports and AI agents?
That is [Mastyf Authority](https://www.mastyf.ai/authority) itself, running in your own infrastructure.
