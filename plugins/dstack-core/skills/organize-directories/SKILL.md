---
name: organize-directories
description: 'Use when a directory tree needs rebuilding into an elegant, well organized, readable, and easy-to-reach layout; when a hard-to-read layout hides where files belong or makes them hard to find; when a layout needs deciding; or when files must move without breaking imports, build files, or doc links. Not for module boundaries and interface seams: use architect.'
---

# Rebuild the directory tree from its consumers

A directory layout is a contract with its consumers: which features change together, who references which paths, what a newcomer must find, and which paths outsiders depend on. State that contract first, derive the target tree from it alone, then replace the old tree one subtree at a time. The target tree is elegant, well organized, readable, and easy to reach; the three tests and the elegance check below judge it. A tree that already passes every test is sound structure and stays.

## State the consumer contract

Write the contract before touching the tree. Source each clause from evidence, never from the old layout:

- Change sets: which files change together. Read `git log --name-only` over recent months and group the paths that land in the same commits.
- References: who imports, links, or loads which paths, from `grep -rn` over sources, build files, configs, and docs.
- Readers: the features and entry points a newcomer must find, named in the product's own vocabulary.
- Outsiders: every path something outside the repository depends on.

Derive the target tree from this contract alone, without looking at how the old tree grew. If a contract clause has no evidence source, stop and report the missing evidence instead of inventing it.

## Group by what changes together

Group directories by feature or domain before kind. All handlers in one directory and all models in another make every feature change cross the tree; `catalog`, `billing`, and `sessions` keep a feature's change in one directory and one review. Give shared code that several features change together its own domain-named directory, not a kind-named bin.

## Keep one concern per directory

Make each directory answer one question. Split a directory that answers two unrelated questions. Merge a directory holding one file with no planned siblings into its parent. Give tests, fixtures, and generated files separate subdirectories when each changes under a different rule.

## Name from the domain

Use domain words such as `catalog`, `invoices`, and `auth`. Kind words such as `utils`, `helpers`, `common`, `misc`, and `core` name no change, so directories with those names collect unrelated files. Rename any directory whose name a newcomer cannot map to the product's vocabulary as part of the same move.

## Keep paths shallow and predictable

Keep depth at four levels and sibling counts near twelve. Past either bound, add the distinction the new files actually follow instead of numbering directories. Place entry points and the files opened most often nearer the root. Fold a level with one child directory upward, and never leave a directory that only forwards the reader to another directory.

## Give fixed kinds one home

Put tests beside the code they test where the toolchain allows it, such as Go `_test.go` files, or in the mirrored tree the toolchain requires, such as Java's `src/test`. The toolchain decides this, not the layout. Keep generated files under one generator-owned directory covered by the ignore file, and let only the generator write there. Keep scratch and local outputs outside the tree or in an ignored directory, never committed. Put docs at the root or under `docs/`, with one home for each document.

## Lay out sibling features alike

Use the same internal layout for sibling features so learning one feature teaches the others. Do not exempt one feature from the grouping, naming, or fixed-kind rules. When a domain distinction justifies a different layout, name that distinction and apply it consistently to every sibling that shares it.

## Test organization

Give every file one obvious home and keep what changes together in the same directory. Name the next three likely changes and check that each lands in one directory. A change that splits across directories, a file with two plausible homes, or a directory holding unrelated concerns fails this test.

## Test readability

Make the top level say what the system is and each directory name say what it holds. Show the top two levels to a reader who knows the product but not the code, name several real features, and ask where each feature lives. The tree passes when the reader answers from the directory names alone, without opening directories or translating internal jargon.

## Test reachability

Keep entry points and frequently opened files near the top, with predictable paths and no pass-through directories. Find the five files changed most often in recent months:

`git log --since='6 months ago' --name-only --pretty=format: -- . | awk 'NF' | sort | uniq -c | sort -nr | head -5`

For each file, start at the repository root, write down the path expected to reach it, and then compare that guess with its actual path. Count directory levels, sibling choices, and wrong guesses. Require at most four directory levels, sibling counts near twelve, and a correct first guess. A deeper path, a larger unexplained fan-out, a pass-through level, or repeated searching fails this test.

## Check elegance

Explain every placement with the smallest set of grouping, naming, and fixed-kind rules. A directory created as an exception for one feature or a sibling feature with an arbitrary internal layout fails the check. Different siblings may differ only when a named domain distinction applies consistently to every sibling with that distinction.

## Classify every old directory

Walk the old tree directory by directory against the target tree and record each verdict with a one-line reason:

- Essential: it has a home in the target tree. Name the target path.
- Residue: it has no home. Kind-named bins, pass-through levels, dead or duplicate directories, stale generated output, and single-child levels are residue. Residue is deleted, not moved.

The walk covers every directory, not a sample: a directory never read is a file lost by accident.

## Separate interior paths from boundary paths

An interior path is one whose every reference resolves inside the repository, proved by search. Interior paths move without asking.

Everything else is a boundary path. Present each boundary path and get an explicit yes or no before touching it: published import paths and package names; URLs and doc links outsiders hold; paths in configuration that runs in someone else's deployment; CLI and plugin paths; anything a version was promised against; and every reference a static search cannot resolve, including string-built paths, reflection, generated code, and runbooks. Unresolved means boundary. A no keeps the path where it is, permanently. Silence is not a yes, and a general instruction is never an answer for a specific path.

## Replace one subtree at a time

Work one target subtree at a time, atomically. Before the first move, the target paths must have no uncommitted changes; if they do, stop and name them, because reverting to the last green commit would discard them. Show the exact move and deletion list before anything is deleted.

Then, for each target:

1. Move with `git mv`, migrate every reference in the same commit (imports, build files, relative links, package clauses, doc links), and delete residue in the same change. Leave no compatibility residue: no forwarding stub, re-export file, alias package, or symlink at an old path. A boundary path that got a no simply stays.
2. Run the build and a link check. Require `git diff --cached -M --name-status` to show every listed pair as `R<score> <old> <new>` with any similarity score and no listed path in an `A` or `D` entry. A moved file whose own references were rewritten scores below 100, which is expected; only identical content earns `R100`.
3. If Git reports `D <old>` plus `A <new>` instead of `R`, the content edit is too large for the selected threshold. Defer content edits not needed by the move to the next commit so the move commit keeps building, or rerun the check with a lower `-M<n>` threshold and record that threshold.
4. Prove history: `git log --follow -- <new-path>` reaches the same earlier history that `git log --follow -- <old-path>` showed before the move. Commit before starting the next target.
5. After the commit, search every old path and every residue name; each must return nothing.

Half-moved is the forbidden state. A target that cannot finish reverts to the last green commit; it is never shipped half old, half new. Nothing in this skill pushes to a remote.

## Keep the limits

Leave a tree that already passes the organization, readability, and reachability tests alone: offense does not apply to sound structure. Delete dead code rather than moving it. Keep a boundary path that has no yes where it is. A named subtree stays a subtree; widen to a repository-wide replacement only when the request names the repository.

## Hand back the result

Return the consumer contract, the target tree with its test results, the per-directory classification with reasons, every boundary path with its answer, and the per-subtree commits with their build, link, rename, and history proofs plus the empty old-path and residue searches.
