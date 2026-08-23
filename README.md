# paug-fr-dns

DNS-as-code for `paug.fr`, hosted at [Gandi](https://www.gandi.net). Records
live in [`zones/paug.fr.yaml`](zones/paug.fr.yaml) and are kept in
sync with Gandi by [octoDNS](https://github.com/octodns/octodns) via the
[octodns-gandi](https://github.com/octodns/octodns-gandi) provider.

Workflow:

1. Edit `zones/paug.fr.yaml` on a branch and open a PR.
2. GitHub Actions (`.github/workflows/dns-plan.yml`) runs `octodns-sync` in
   dry-run mode and posts the diff as a PR comment.
3. Review the diff, get it approved, merge.
4. GitHub Actions (`.github/workflows/dns-apply.yml`) runs `octodns-sync
   --doit` on `main`, which pushes the change to Gandi.

No Terraform-style state file to manage: octoDNS diffs the YAML in this repo
directly against Gandi's live records on every run.

## One-time setup

1. **Create a Personal Access Token.** In the Gandi account dashboard, under
   [Personal Access Tokens](https://admin.gandi.net/organizations/account/pat),
   generate a token scoped at minimum to:
   - See and renew domain names
   - Manage domain name technical configurations
2. **Add a repo secret.** In this repo's Settings → Secrets and variables →
   Actions, add:
   - `GANDI_TOKEN`
3. **(Recommended) Protect the `production` environment.** The apply
   workflow runs under the `production` GitHub environment
   (Settings → Environments → New environment → `production`). Add required
   reviewers there if you want a manual approval gate on top of PR review
   before changes actually hit Gandi.
4. **Seed real records** (see below) before merging anything for real — the
   committed `zones/paug.fr.yaml` is currently a placeholder.

## Seeding real records

`zones/paug.fr.yaml` starts as a placeholder. To replace it with what's
actually live at Gandi:

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

export GANDI_TOKEN=...

octodns-dump --config-file=config/octodns.yaml --output-dir=zones --lenient paug.fr. gandi
```

This overwrites `zones/paug.fr.yaml` with the current live records.
Review the diff, remove the placeholder comment, and commit.

## Local dry run

```bash
source .venv/bin/activate
export GANDI_TOKEN=...

octodns-validate --config-file=config/octodns.yaml   # syntax/config check only
octodns-sync --config-file=config/octodns.yaml       # dry run, prints the plan
octodns-sync --config-file=config/octodns.yaml --doit  # actually apply
```

## Layout

```
config/octodns.yaml  # octoDNS provider config (source: ./zones, target: Gandi)
zones/paug.fr.yaml    # source of truth for paug.fr records
.github/workflows/    # plan-on-PR, apply-on-merge
```
