# About
check-my-vibe assists coders with understanding code changes that have been made by LLMs (currently Claude Code)
by initiating a dialogue between the LLM and the coder to facilitate mutual understanding of major code changes.

It's accompanied by a **github action** that can block PR merges until they have run `check-my-vibe`

This is a private dialogue between the software engineer and the LLM. We admit to ourselves that the process of
learning and understanding the inner workings of a PR can be messy and imperfect.

- **Private** — the Q&A happens locally in Claude Code. Nothing is posted to the PR or
  visible in CI logs; only a one-line status flip touches GitHub.
- **Optionally Enforced** — a GitHub Action arms an `understanding-check` status as *pending* on every
  push; the PR cannot merge until your local interview flips it to *success*.

See [PLAN.md](./PLAN.md) for the full design and rationale.

## How it works

A GitHub Action can't push an interactive chat onto your laptop, so the arrow is inverted:
the local interview is what unblocks the PR.

1. Develop in claude code.
2. Push PR and review it sufficiently.
3. Run `/check-my-vibe` to further your understanding
4. Claude runs .understanding/set-status.sh success for your PR (there is a bit of an honor system)
5. PR is unblocked and you can merge.

----

Because the status is written per commit SHA, pushing new commits resets the gate to
`pending` — so the check always reflects the *current* code.

## Install into a repo

### Quick install

The skill `/check-my-vibe` is installed locally in the repo along with its associated `.understanding/set-status.sh` script.

```sh
curl -fsSL https://raw.githubusercontent.com/Jeffrharr/CheckMyVibe/main/scripts/global-install.sh | bash -s -- /path/to/target-repo
```

This downloads and installs everything without cloning CheckMyVibe:

- `/check-my-vibe` skill → `~/.claude/skills/check-my-vibe/` (global, works in any repo)
- `.understanding/set-status.sh` — vendored into the target repo
- `.github/workflows/understanding-gate.yml` — vendored into the target repo
- `.understanding/config` — config template (skipped if one already exists)

To install just the skill without targeting a specific repo yet:

```sh
curl -fsSL https://raw.githubusercontent.com/Jeffrharr/CheckMyVibe/main/scripts/global-install.sh | bash
```

### Manual install (from a local clone)

If you have a local clone of this repo:

```sh
scripts/install-into.sh /path/to/target-repo            # skill lives in the target repo
```

### After either install

Then, in the target repo:

1. Commit the vendored files.
2. **Branch protection → Require status checks to pass →** add a required check named exactly
   `understanding-check`. This is what actually blocks merges. (The workflow still arms the
   check and the `/check-my-vibe` flow still works without this step — but nothing is
   *enforced* until the check is required.)

> **Note:** Required status checks need branch protection, which on GitHub's free tier is
> only available for **public** repos (private repos need Pro/Team). Without it, the gate is
> advisory — the check shows on PRs but doesn't block merge.

## Usage

1. Open a PR. The gate posts `understanding-check = pending` and the merge button is blocked.
2. When you're ready to merge, run `/check-my-vibe` in Claude Code. It pulls the diff
   and interviews you about the change.
3. Once you've shown you understand it and confirm, the skill flips the check to `success`
   and the PR can merge.

## Skills

The toolkit ships two Claude Code skills:

- **`check-my-vibe`** — the interactive pre-merge interview that probes your
  understanding of a PR and, on confirmation, clears the gate. This is the one wired into
  the gate by default.
- **`junior-review`** — a sharp but domain-blind junior reviewer that emits 3–5
  diff-specific questions exposing unstated assumptions. Output only; it doesn't conduct an
  interview or touch the gate. Useful as a quick self-review, or as a question source to
  feed the interview.

## Configuration

The gate reads optional settings from `.understanding/config` (created by the installer),
overridable by environment variables. Precedence: **built-in default < `.understanding/config` < env**.

| Setting | Default | Controls |
|---|---|---|
| `UNDERSTANDING_SKILL` | `check-my-vibe` | The slash command shown in the check's unblock message. Point it at your own interview skill if you've made one. |
| `UNDERSTANDING_CONTEXT` | `understanding-check` | The GitHub status check name. Must match your branch-protection required check — change both together. |
| `UNDERSTANDING_DOCS_URL` | this README's "Unblocking a PR" | The status "Details" link. |

Example `.understanding/config`:

```sh
UNDERSTANDING_SKILL=pr-deep-dive
UNDERSTANDING_DOCS_URL=https://github.com/acme/dev-docs#understanding-gate
```

## Unblocking a PR

If a PR is blocked by the `understanding-check` status, run **`/check-my-vibe`** in
Claude Code from the repo's working tree. It conducts the interview and, on your explicit
confirmation, clears the check on the PR's current head commit. (This section is what the
status's "Details" link points to.)

## Repo layout

```
action.yml                             # composite action — arms the gate on PR push (GitHub Marketplace)
scripts/set-status.sh                  # local status writer (vendored into consumers for /check-my-vibe)
scripts/install-into.sh                # vendor the gate into a target repo (from a local clone)
scripts/global-install.sh              # curl-installable install, no clone needed
templates/understanding-gate.yml       # the workflow copied into a consumer's .github/workflows
skills/check-my-vibe/SKILL.md          # the /check-my-vibe interview (clears the gate)
skills/junior-review/SKILL.md          # the /junior-review assumption-exposing questions
PLAN.md                                # design, components, milestones
```

## License

[MIT](./LICENSE) © 2026 Jeffrey Harrison
