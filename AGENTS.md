# AGENTS.md

> This is a **living document**. It describes how AI agents (and humans
> acting like one) should operate inside the **Hackpack** repository. It is
> updated whenever the workflow, tooling, or repository structure changes.
> If you notice this document is out of date after making a change, update
> it as part of the same piece of work.

---

## What this repository is

Hackpack is an up-to-date collection of precompiled offensive-security
binaries, PowerShell scripts, and Bettercap caplets, built and released
automatically via GitHub Actions. It is **not** a place to write new tooling
from scratch — most of the value lives in:

- `windows/` — Git submodules (and a couple of vendored scripts) pointing to
  upstream offensive tooling repositories (Certify, Rubeus, Seatbelt,
  PowerSploit, Inveigh, etc.), compiled by CI into ready-to-use binaries.
- `bettercap/caplets/` — Bettercap caplets used for network attacks
  (DHCPv6 spoofing, DNS spoofing, etc.).
- `.github/workflows/hackpack.yml` — the single workflow that compiles C#
  projects, collects PowerShell scripts, bundles caplets, and publishes a
  rolling `latest` GitHub Release.
- `.github/README.md` — the public-facing README, rendered on GitHub and
  reused (stripped of the logo) as the release notes body.

Understanding this shape matters: touching `windows/` usually means bumping
a submodule pointer or adjusting the build matrix in
`hackpack.yml`, not editing vendored code in place.

---

## Read this first

Before making **any** change to this repository, read
**[CONTRIBUTING.md](./CONTRIBUTING.md)** in full. It contains the
non-negotiable rules for branching, commits, and merges. This file
(`AGENTS.md`) tells you *what the repo is* and *how to think about it*;
`CONTRIBUTING.md` tells you *the exact mechanics* of contributing safely.
The rules in `CONTRIBUTING.md` apply to every agent and every human,
without exception.

---

## Ground rules for agents specifically

- **Never work directly on `main`.** Create a branch first, per
  `CONTRIBUTING.md`.
- **Never merge or push to `main`.** Merges are only authorised explicitly by
  the project owner ([@cosasdepuma](https://github.com/cosasdepuma), see
  `.github/CODEOWNERS`).
- **Prefer submodule bumps over vendored edits.** If a tool in `windows/`
  needs a fix, the fix belongs upstream. Only touch files that are not
  submodules (e.g. `windows/Get-ServiceACL/`) directly.
- **Keep the CI matrix and the README table in sync.** Any tool added,
  removed, or renamed in `windows/` or `bettercap/caplets/` must be reflected
  in both `.github/workflows/hackpack.yml` and `.github/README.md`. A tool
  that isn't in all three places (submodule/file, workflow matrix, README
  table) is effectively broken.
- **Don't invent secrets or credentials.** The workflow relies on
  `secrets.GITHUB_TOKEN` only. Never hardcode tokens, keys, or credentials in
  workflow files, scripts, or caplets.
- **Validate before asking for a merge.** At minimum, lint the YAML you
  touched and sanity-check that `.github/README.md` table entries still
  match the actual files/submodules. See `CONTRIBUTING.md` for the full
  validation checklist.
- **Offensive tooling context.** Everything here is meant for authorised
  security testing. Don't add functionality that weakens that boundary
  (e.g. exfiltration to third-party endpoints, telemetry, or anything not
  requested by the maintainer).

---

## Keeping this document alive

This file must evolve with the repository. Update `AGENTS.md` when:

- The repository structure changes (new top-level directories, new
  submodule categories, new artifact types).
- The CI workflow changes in a way that affects how contributions are
  validated (new jobs, new required checks, new secrets).
- The contribution process itself changes — but for that, edit
  `CONTRIBUTING.md` and simply make sure this file still points to it
  correctly.

If you're an agent and you just made a structural change, updating this
file is part of finishing the task, not an optional follow-up.
