# Contributing to Hackpack

> This document is addressed to **every person who touches this codebase** —
> human or AI agent. Read it fully before making any change. There are no
> exceptions.

---

## The golden rules

These rules are non-negotiable. Violating any of them will require the work to
be redone from scratch.

### 1. You never work on `main`

`main` is the stable, deployable trunk. It is not a scratch pad. Every push
to it triggers the release workflow (`.github/workflows/hackpack.yml`), which
compiles tools and publishes a public GitHub Release.

- **Never commit directly to `main`.**
- **Never push to `main` directly.**
- **Every single change**, no matter how small, goes on a dedicated branch
  first.

If you find yourself on `main` with uncommitted changes, stop. Stash them,
create a branch, and apply them there:

```bash
git stash
git checkout -b fix/my-accidental-change
git stash pop
```

### 2. You never merge without explicit consent

Merging into `main` is a deliberate, human-approved action. No branch is ever
merged automatically, speculatively, or "just to keep things tidy".

- **Wait for an explicit "merge this" instruction** before running any merge.
- When in doubt, ask. Do not assume.
- The only person who authorises a merge is the project owner
  ([@cosasdepuma](https://github.com/cosasdepuma), see `.github/CODEOWNERS`).

When a merge is authorised, always use `--no-ff` so the branch topology is
preserved in the graph. The merge commit must also use the commit convention:

```bash
git checkout main
git merge --no-ff feat/add-tool -m "chore(tools): merge add-tool branch"
```

### 3. Every commit and every branch must follow the naming convention

Consistency in naming makes the history readable at a glance. Deviating from
the convention makes the history noisy and harder to audit.

See [Branch naming](#branch-naming) and [Commit messages](#commit-messages)
for the full rules.

### 4. Validate before asking for a merge

Before asking for a merge, validate the parts of the repository you touched:

- **Submodules (`windows/*`):** if you bumped a submodule pointer, confirm it
  points to a real commit on the tracked branch declared in `.gitmodules`.
  ```bash
  git submodule status
  ```
- **CI workflow (`.github/workflows/hackpack.yml`):** if you edited it, lint
  the YAML and, ideally, validate it with
  [`actionlint`](https://github.com/rhysd/actionlint) or by triggering a
  `workflow_dispatch` run on your branch.
  ```bash
  actionlint .github/workflows/hackpack.yml
  ```
- **README table (`.github/README.md`):** any tool added, removed, or
  renamed must be reflected here. Every entry must link to either the
  upstream repository (for submodules) or the local file/caplet path.
- **New scripts or caplets:** confirm the file is not accidentally binary,
  has no embedded secrets or credentials, and, where applicable, runs
  without syntax errors (`pwsh -File script.ps1 -WhatIf` style checks or
  `bettercap -eval "..."` dry runs, as appropriate).

Fix any problem before requesting a merge.

---

## Branch naming

Branches follow the `type/short-description` pattern in **kebab-case**:

```text
feat/add-tool
fix/godpotato-build-matrix
chore/bump-rubeus-submodule
docs/update-agents-guide
refactor/workflow-jobs
```

| Prefix | When to use |
|---|---|
| `feat/` | New tool, caplet, or capability added to the pack |
| `fix/` | Bug fix (build failures, broken links, wrong paths) |
| `chore/` | Maintenance: submodule bumps, workflow tweaks, tooling |
| `docs/` | Documentation only (`README.md`, `AGENTS.md`, `CONTRIBUTING.md`) |
| `refactor/` | Restructure without behaviour change |
| `style/` | Formatting only, no logic or content change |

**Rules:**

- Use only lowercase letters, numbers, and hyphens. No slashes beyond the
  prefix.
- Keep descriptions short and specific (`fix/inveigh-publish-target`, not
  `fix/stuff`).
- One concern per branch. If you need to do two unrelated things (e.g. bump
  a submodule *and* fix the workflow), open two branches.

---

## Commit messages

Follow [Conventional Commits](https://www.conventionalcommits.org/) strictly.
Every commit, including a merge commit, must have the format:

```text
type(scope): short imperative description

Optional body explaining *why*, not *what*.
```

The scope is mandatory. Never omit it. Use a scope that names what the
commit touches: a tool name (`rubeus`, `godpotato`), an area
(`workflow`, `readme`, `submodules`, `caplets`), or `agents` for guidance
docs.

The description must be in the imperative mood: use "add", "fix", or
"remove", not "added", "fixes", or "removing".

**AI agent rule:** Never append `Co-Authored-By` trailers to commit messages.

### Types

| Type | When to use |
|---|---|
| `feat` | New tool, caplet, or capability |
| `fix` | Bug fix |
| `chore` | Maintenance: submodule bumps, CI config, tooling |
| `docs` | Documentation only |
| `refactor` | Restructure without behaviour change |
| `style` | Formatting or whitespace |

### Valid examples

```text
feat(caplets): add arp-spoofing caplet
fix(workflow): correct GodPotato build platform
chore(submodules): bump Rubeus to latest upstream commit
docs(agents): document submodule workflow
refactor(workflow): merge csharp and powershell jobs
```

### Invalid examples

```text
fix stuff                              ← missing type and scope
feat(tools): Added a caplet            ← past tense; use "add"
update                                 ← meaningless, no type, no scope
WIP                                    ← never commit WIP to a shared branch
feat(workflow): fix build and add tool ← separate unrelated concerns
```

---

## Working with submodules

Most of `windows/` is made of Git submodules pointing to upstream offensive
tooling repositories, declared in `.gitmodules`. When working with them:

- **Don't edit vendored code in place.** Fixes to a tool's source belong
  upstream, in the tool's own repository. Open an issue or PR there.
- **Bumping a submodule** is a legitimate `chore/` change:
  ```bash
  git submodule update --remote windows/Rubeus
  git add windows/Rubeus
  git commit -m "chore(rubeus): bump submodule to latest upstream commit"
  ```
- **Adding a new tool as a submodule** requires updating three places in the
  same change: `.gitmodules`, the build/collection matrix in
  `.github/workflows/hackpack.yml`, and the table in `.github/README.md`.
- Local, non-submodule scripts (e.g. `windows/Get-ServiceACL/`) can be edited
  directly, but still follow the same branch/commit rules.

---

## Workflow summary

```text
1. Start from an up-to-date main
   git checkout main && git pull

2. Create a dedicated branch
   git checkout -b feat/add-tool

3. Work and commit with meaningful Conventional Commit messages
   git commit -m "feat(scope): do something specific"

4. Validate what you touched
   git submodule status
   actionlint .github/workflows/hackpack.yml   # if the workflow changed

5. Push the branch and ask for an explicit merge approval
   git push origin feat/add-tool
```

Do not merge or push to `main` without explicit authorisation.
