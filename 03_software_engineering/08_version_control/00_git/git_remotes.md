---
layer: 03_software_engineering
type: engineering
tool: git
status: growing
tags:
  - git
created: 2026-03-02
---

# Git Remotes

## Purpose

Remotes are named references to other copies of the repository (typically on GitHub, GitLab, or a private server). They enable collaboration by providing a shared point of truth to push to and pull from.

## Architecture

A remote is stored as a URL alias in `.git/config`. The conventional name for the primary remote is `origin`.

**Remote-tracking branches** (e.g. `origin/main`) are local read-only refs that record the last-known state of the remote. They are updated by `git fetch`.

```
Local repo:         Remote (origin):
main                main
origin/main  ←───   main (remote)
```

After a `git fetch`, `origin/main` advances but your local `main` does not — you decide when to merge or rebase.

## Implementation Notes

**Manage remotes:**
```bash
git remote -v                          # list remotes with URLs
git remote add <name> <url>            # add a remote
git remote remove <name>               # remove a remote
git remote rename <old> <new>          # rename
git remote set-url <name> <url>        # change URL
```

**Fetch** (download objects + update remote-tracking branches; does not touch local branches):
```bash
git fetch                              # fetch from origin
git fetch <remote>                     # fetch from specific remote
git fetch --all                        # fetch all remotes
```

**Push** (upload commits to remote):
```bash
git push                               # push current branch to its upstream
git push <remote> <branch>             # explicit
git push -u origin <branch>            # push and set upstream tracking
git push --force-with-lease            # force-push safely (fails if remote has new commits)
git push origin --delete <branch>      # delete a remote branch
```

**Pull** = `git fetch` + `git merge` (or rebase if configured):
```bash
git pull                               # pull current branch from its upstream
git pull --rebase                      # pull with rebase instead of merge
```

Prefer `git fetch` + explicit `git merge`/`git rebase` over bare `git pull` for more control and visibility.

## Trade-offs

- `git pull` (merge) creates merge commits on every pull; `git pull --rebase` keeps a linear history.
- `git push --force` overwrites remote history — dangerous on shared branches. Always prefer `--force-with-lease`.
- For HTTPS remotes, credentials can be cached with the system keychain or a credential helper. For SSH remotes, set up an SSH key pair for passwordless auth.

## References

- [git-remote documentation](https://git-scm.com/docs/git-remote)
- [git-push documentation](https://git-scm.com/docs/git-push)
- [git-fetch documentation](https://git-scm.com/docs/git-fetch)

## Links
- [[git_branching|Git Branching]]
- [[git_workflow|Git Workflow]]
- [[git_pull_requests|Pull Requests]]
