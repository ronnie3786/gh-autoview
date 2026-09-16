# gh-autoview

A [GitHub CLI](https://cli.github.com) extension that marks boring PR files
(tests, lockfiles, snapshots, generated docs, …) as **Viewed** so the PR review
UI collapses them and you only read what matters.

Which files are "boring" comes from a per-repo config file written in
`.gitignore` syntax — matching is done with `git check-ignore`, so the
semantics are exactly the ones you already know. With no config file, the
default policy is: mark everything that is not hand-written Swift, plus
generated Swift (`*.g.swift`) and unit tests (`*Test.swift`, `*Tests.swift`).

Fork of [`OperatorJack/gh-autoview`](https://github.com/OperatorJack/gh-autoview).
Changes here: **dry run is the default** and needs `--apply` to write,
`.gitattributes` as a pattern source, per-repo configs under
`~/.config/gh-autoview/<owner>-<repo>`, and a post-write verification pass.

## Install (local fork)

    cd ~/Documents/Development/gh-autoview
    gh extension install .

`gh` symlinks the working copy instead of copying it, so edits take effect on
the next run — no reinstall.

## Requirements

- **Bash ≥ 4.4** (macOS's stock `/bin/bash` 3.2 is too old — `brew install bash`)
- [`gh`](https://cli.github.com) (authenticated with `repo` scope)
- [`jq`](https://jqlang.github.io/jq/)
- `git`
- `column` (util-linux/bsdmainutils; only needed for `--list`)

Works on Linux, macOS, and WSL. On native Windows, run it under Git Bash or WSL.

## Use

    cd ~/git/some-repo            # on a branch with an open PR
    gh autoview 123               # dry run (default)
    gh autoview 123 --apply       # mark matching files viewed
    gh autoview 123 -l            # list all files of PR 123 with state/match
    gh autoview 123 -u --apply    # undo: un-view matching files
    gh autoview -h                # all options

Options:

    PR                 PR number, URL, or branch. Default: PR for current branch.
    -a, --apply        Actually change viewed state. Without it: a dry run.
    -n, --dry-run      Explicit dry run; this is already the default.
    -l, --list         Show every PR file with its viewed state and match.
    -u, --unmark       Un-view matching files instead (undo). Needs --apply.
    -c, --config FILE  Pattern file to use.
    -R, --repo O/R     Repo to use (default: derived from current directory).

## Uninstall

    gh extension remove autoview

## Configure

Create `.autoview` (or `.github/autoview`) at the repo root, in gitignore
syntax (see [`.autoview.example`](.autoview.example)):

    *Tests.cs
    *.test.tsx
    MAP.md
    pnpm-lock.yaml
    !src/important.test.tsx   # negation supported

Config lookup order: `--config FILE`, `.autoview`, `.github/autoview`,
`~/.config/gh-autoview/<owner>-<repo>`, `~/.config/gh-autoview/default`, then the
built-in default policy.

A config file **replaces** the default policy, so a repo that wants its own list
can opt out entirely. The policy itself is five gitignore lines — last match
wins, and `!**/` is load-bearing (without it git refuses to re-include files
under a directory matched by `**/*`):

    **/*             every file
    !**/             ...except directories
    !**/*.swift      ...and except hand-written Swift
    **/*.g.swift     ...but mark generated Swift
    **/*Test.swift   ...and unit tests
    **/*Tests.swift

Because the default is "everything that is not Swift", pointing this at a
non-Swift repo would mark nearly every file. Give such a repo a config file.

The two `~/.config/gh-autoview/` paths let you keep repo-specific patterns out
of the work repo, so `git status` stays clean.

## How it works

1. Resolves the PR with `gh pr view` and fetches every changed file with its
   `viewerViewedState` via the GraphQL API (paginated).
2. Feeds the paths through `git check-ignore` using your config as the
   exclude file.
3. Sends `markFileAsViewed` / `unmarkFileAsViewed` mutations in batches of 25,
then re-reads the viewed state and fails loudly if anything didn't take.

## License

[MIT](LICENSE)
