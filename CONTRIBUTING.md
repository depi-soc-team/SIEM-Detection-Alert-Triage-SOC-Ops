# Contributing

How the DEPI SOC team works in this repository. Read this before your first PR.

New to Git? Follow the step-by-step guide (Egyptian Arabic): [docs/GIT-GUIDE.md](docs/GIT-GUIDE.md). It covers setup, the daily workflow with GitHub Desktop, the command line or the website only, reviews and common problems.

## 1. Clone

```bash
git clone https://github.com/depi-soc-team/SIEM-Detection-Alert-Triage-SOC-Ops.git
cd SIEM-Detection-Alert-Triage-SOC-Ops
```

Before starting new work, update `main`:

```bash
git checkout main
git pull
```

## 2. Pick an issue

All tasks are GitHub issues on the **SOC Project Board**, labeled by phase, owner and type, and grouped into milestones M1–M4.

- Work only on issues assigned to you (or ones you support, coordinated with the owner).
- Move the card to **In Progress** when you start and **Review** when your PR is open.

## 3. Branch naming

```
<member>/<short-task>
```

- `member`: `basel`, `merna`, `ramez`, `saieed`, `ahmed`
- `short-task`: lowercase kebab-case, a few words

Examples: `merna/fortigate-syslog`, `ramez/dr-001-suspicious-powershell`, `ahmed/soc-dashboard`

```bash
git checkout -b merna/fortigate-syslog
```

## 4. Commits

- Small, focused commits with descriptive messages (e.g. `Add FortiGate syslog onboarding guide`).
- Check your diff for sensitive data before every commit (see [Security rules](#7-security-rules-mandatory)):

```bash
git diff --staged
```

## 5. Pull request flow

1. Push your branch: `git push -u origin <member>/<short-task>`
2. Open a PR into `main`.
3. Put `Closes #N` in the PR description (N = issue number) so the issue closes on merge.
4. Request **one reviewer** — usually the issue's support member, otherwise any teammate.
5. Address review comments, then the reviewer approves and merges.
6. Delete the branch after merge.

> **Team rule, not enforced by GitHub:** every PR needs one approving review before merge. Branch protection is not available on the free plan for private repositories, so GitHub will not block an unreviewed merge — we rely on each other to follow this rule. Do not merge your own PR without a review.

PR description template:

```markdown
## Summary
What this PR adds/changes.

## Evidence
Screenshots / validation (sanitized).

Closes #N
```

## 6. Screenshots

Name screenshots:

```
YYYYMMDD_member_task.png
```

Examples: `20261005_merna_fleet-agents-healthy.png`, `20261012_saieed_dr-003-alert.png`

- Store them next to the doc that uses them (e.g. in an `images/` folder in that section).
- Crop or blur real IPs, hostnames, usernames and tokens **before** committing.

## 7. Security rules (mandatory)

- Never write real IP addresses, hostnames, domains, usernames, passwords, API keys, enrollment tokens, or certificates into any file.
- Use placeholders: `<host-01>`, `<DC01>`, `<user>`, `10.0.0.x`, `example.local`, `<ELASTIC_PASSWORD>`.
- Secrets go in `.env` (git-ignored) and are referenced by variable name only.
- Do not commit raw logs or captures (`*.evtx`, `*.pcap`); commit sanitized excerpts instead.
- Defang IOCs in reports (`hxxp://`, `1.2.3[.]4`).

If you commit sensitive data by mistake, tell Basel immediately. Do not just delete it in a new commit; it stays in git history.

## 8. Content conventions

- Detection rules: `03-detection-rules/DR-XXX-short-name.md`, based on `rule-template.md`.
- Triage reports: `05-triage/TR-XXX-short-name.md`, based on `triage-template.md`.
- Lowercase kebab-case file names; dates as `YYYY-MM-DD`; times in UTC.
- Use Elastic Common Schema (ECS) field names; map every rule to MITRE ATT&CK.
- Kibana exports stay as `.ndjson`.
