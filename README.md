# gh-autoview

A [GitHub CLI](https://cli.github.com) extension that marks boring PR files
(tests, lockfiles, snapshots, generated docs, …) as **Viewed** so the PR review
UI collapses them and you only read what matters.

Which files are "boring" comes from a per-repo config file written in
`.gitignore` syntax — matching is done with `git check-ignore`, so the
semantics are exactly the ones you already know.

## Requirements

- **Bash ≥ 4.4** (macOS's stock `/bin/bash` 3.2 is too old — `brew install bash`)
- [`gh`](https://cli.github.com) (authenticated with `repo` scope)
- [`jq`](https://jqlang.github.io/jq/)
- `git`
- `column` (util-linux/bsdmainutils; only needed for `--list`)

Works on Linux, macOS, and WSL. On native Windows, run it under Git Bash or WSL.

## Install

    gh extension install OperatorJack/gh-autoview

## Use

    cd ~/git/some-repo            # on a branch with an open PR
    gh autoview -n                # dry run
    gh autoview                   # mark matching files viewed
    gh autoview 123 -l            # list all files of PR 123 with state/match
    gh autoview 123 -u            # undo: un-view matching files
    gh autoview -h                # all options

Options:

    PR                 PR number, URL, or branch. Default: PR for current branch.
    -n, --dry-run      Show what would be marked, change nothing.
    -l, --list         Show every PR file with its current viewed state and match.
    -u, --unmark       Un-view matching files instead (undo).
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

Config lookup order: `--config FILE`, `.autoview`, `.github/autoview`, then
`~/.config/gh-autoview/default` as a fallback for repos without one.

## How it works

1. Resolves the PR with `gh pr view` and fetches every changed file with its
   `viewerViewedState` via the GraphQL API (paginated).
2. Feeds the paths through `git check-ignore` using your config as the
   exclude file.
3. Sends `markFileAsViewed` / `unmarkFileAsViewed` mutations in batches of 25.

## License

[MIT](LICENSE)
