---
layer: 03_software_engineering
type: engineering
tool: git
status: growing
tags:
  - git
created: 2026-03-02
---

# Git Internals

## Purpose

Understand Git's object model — the low-level data structures that underpin every commit, branch, and file. This knowledge is rarely needed day-to-day but is essential for debugging complex Git situations and reasoning about what commands actually do.

## Architecture

Git stores everything in its object database (`.git/objects`). There are four object types, all content-addressed by their **SHA-1 hash**:

| Object | Description |
|--------|-------------|
| `blob` | Contents of a single file |
| `tree` | A directory listing (maps names → blobs or subtrees) |
| `commit` | A snapshot: points to a tree + parent commit(s) + metadata |
| `tag` | A named pointer to another object (usually a commit) |

A **commit** object contains:
- The SHA of the root `tree` (full snapshot of all tracked files)
- The SHA(s) of parent commit(s) (none for the initial commit; two for a merge commit)
- Author, committer, timestamps
- Commit message

Because every object is identified by the SHA-1 hash of its content, Git history is tamper-evident: changing anything changes the hash, which cascades up through all descendant objects.

## Implementation Notes

Inspect any object by its hash:
```bash
git cat-file -t <hash>   # print object type (blob, tree, commit, tag)
git cat-file -p <hash>   # pretty-print object contents
```

Example — inspecting a commit:
```bash
git cat-file -p 5ba786f
# tree 4e507fd...
# author  ...
# committer ...
#
# commit message
```

The tree listed in a commit can itself be inspected to see which blobs (files) it references.

**Short hashes:** Git accepts the first 7 characters of a SHA as a unique identifier, e.g. `5ba786f` instead of `5ba786fcc93e8092831c01e71444b9baa2228a4f`.

**Refs** are human-readable pointers to commits: branches, tags, and `HEAD` are all refs stored as files in `.git/refs/` or `.git/HEAD`.

## Trade-offs

- SHA-1 is cryptographically weak; Git is transitioning to SHA-256 (`git init --object-format=sha256`) but SHA-1 remains the default.
- The content-addressable store means identical file content is stored only once (deduplication), but large binary files can bloat `.git/objects`.
- Never modify `.git/objects` by hand — always use Git commands.

## References

- [Git Internals — Git Objects (Pro Git)](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)

## Links
- [[03_software_engineering/version_control/git/git_staging|Git Staging Area]]
- [[03_software_engineering/version_control/git/git_log|Git Log]]
