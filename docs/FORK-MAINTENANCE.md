---
title: T3 Code fork maintenance and customization guide
type: runbook
brand: none
domain: software
status: active
updated: 2026-10-26
keywords: [t3code, fork, upstream, git, architecture, customization]
confidential: false
---

# T3 Code fork: architecture and upstream workflow

This checkout is a personal fork of [`pingdotgg/t3code`](https://github.com/pingdotgg/t3code), cloned at `products/t3code-fork`. `origin` is the personal GitHub fork; `upstream` is the maintainers' repository. The codebase is MIT-licensed; keep its copyright and license notices in redistributed copies. This is a source-code fork, not a promise that every upstream change will merge cleanly or that upstream will accept contributions.

## Remotes and branch model

- `main` tracks `origin/main` and should remain a clean, upstream-derived integration branch.
- `upstream/main` is the maintainers' latest default branch.
- `feature/<short-topic>` branches contain our product changes. Keep work in small, focused commits and merge/rebase it onto `main` deliberately.
- Release/install builds are separate from source branches: building this checkout does not replace the installed packaged app unless we explicitly package and install it.

Inspect before changing anything:

```sh
git status -sb
git remote -v
git branch -vv
```

Fetch and review upstream changes:

```sh
git fetch upstream
git log --oneline --decorate --graph --left-right main...upstream/main
```

Update the clean integration branch, then push the updated fork:

```sh
git switch main
git merge --ff-only upstream/main
git push origin main
```

If `--ff-only` fails, stop and inspect `git log --graph --oneline --decorate --all`; do not force-push or blindly merge. If `main` intentionally has local commits, use a reviewed merge (`git merge upstream/main`) or rebase those commits onto the new upstream, then resolve/test conflicts and push normally. Never rewrite shared history without a deliberate decision.

Start a customization from the current integration branch:

```sh
git switch main
git pull --ff-only origin main
git switch -c feature/my-feature
```

When upstream moves, first update `main` as above. Bring the new `main` into the feature branch with `git merge main` (safer when the branch is shared) or `git rebase main` (only for private, unpublished work). Resolve conflicts by understanding both sides, run focused checks, and commit the resolution. Push feature work to `origin`, never to `upstream`.

## Customization architecture strategy

T3 Code is a multi-surface product, not just a UI: a TypeScript monorepo with server/provider integrations and shared contracts, plus web, Electron desktop, and React Native mobile clients. Before implementing a change, read the repository's `AGENTS.md`, `docs/internals/overview.md`, the relevant package README/AGENTS file, and the contribution/testing guidance. Trace a feature across its boundaries rather than patching only the screen where it first appears.

Prefer, in order:

1. Configuration, existing extension points, themes, and reusable components.
2. Small additive packages/modules with explicit integration points and tests.
3. Narrow modifications to existing code, keeping commits feature-scoped and easy to replay.
4. Broad upstream-file rewrites or generated/vendor changes only when unavoidable.

For UI/UX changes, decide explicitly whether behavior applies to web, desktop (Electron shell around web), and mobile (separate React Native client). For shared behavior, inspect `packages/client-runtime`; wire changes crossing client/server boundaries through `packages/contracts`; implement user-triggerable actions as service/domain logic where applicable rather than only in a UI handler. Consider provider adapters, local and remote connection modes, accessibility, reverse actions/states, performance, and tests/docs. Use the repository's own instructions for checks; avoid repo-wide tests unless asked.

Keep an architecture decision log for choices that deliberately diverge from upstream: motivation, touched surfaces/packages, alternatives, migration/rebase notes, and tests. Store it in this fork's `docs/decisions/` as dated Markdown. Do not copy secrets, user databases, or local credentials into commits.

## Expected upstream maintenance

A GitHub fork does **not** stop receiving the maintainers' work. Git keeps the shared commit ancestry; fetch discovers their new commits and merging/rebasing integrates them. It does not automatically combine conflicting edits: when both sides change the same code, a human must resolve the conflict and verify behavior. With a fast-moving project, update regularly, keep local diffs small, avoid long-lived divergence, and carry customizations as discrete feature commits. There is no guarantee a heavily modified fork will remain low-effort to maintain.

Use this review loop for each update:

1. Fetch upstream; inspect the commit range and release notes.
2. Update `main` by fast-forward where possible; otherwise stop for explicit history review.
3. Merge updated `main` into each active feature branch (or rebase private work).
4. Resolve conflicts by preserving intent from both sides; do not resolve wholesale with `ours`/`theirs` without reviewing files.
5. Run relevant focused tests/typechecks/lints, build/package checks for changed surfaces, then manually smoke-test affected behavior.
6. Push to the personal fork and record non-obvious conflicts/decisions.

## Current local installation

The packaged nightly desktop application is installed separately through Debian's package manager. This source checkout does not itself run as an installed Linux desktop application. To install a later upstream nightly, obtain its Linux `.deb` from the official [T3 Code releases](https://github.com/pingdotgg/t3code/releases), verify its SHA-256 against the published release metadata, and install it with `sudo apt install ./T3-Code-*.deb`. Nightly builds can be unstable and may replace/downgrade preview builds; verify the package version before/after. Do not run a source dev server against real user data: follow the repository's dev-isolation instructions in `AGENTS.md`.
