---
name: organize-directories
description: 'Use when a directory layout needs deciding, new code keeps landing in the wrong place, or files must move without breaking imports, build files, or doc links. Not for module boundaries and interface seams: use architect.'
---

# Decide a directory layout

A layout is a contract about where change lands: a reader who knows the feature knows the directory. This skill decides that contract, applies it as moves, and declines moves that buy nothing. The grouping rule comes first, the move procedure second, the refusals last.

## Group by what changes together

Group directories by feature or domain before kind. All handlers in one directory and all models in another make every feature change cross the tree; `catalog`, `billing`, and `sessions` keep a feature's change in one directory and one review. Shared code that several features change together gets its own domain-named directory, not a kind-named bin. Apply the deciding test to any proposal: name the next three likely changes and check that each lands in one directory; a change that splits across directories is a grouping defect.

## Keep one concern per directory

A directory answers one question. Where a feature keeps its tests, its fixtures, and its generated files, each gets its own subdirectory, because each changes under a different rule. A directory holding one file with no planned siblings merges into its parent. A directory answering two unrelated questions splits.

## Bound depth and fan-out

Keep depth at four levels and sibling counts near twelve. Past either bound, the grouping above the growth point is missing a distinction; add the distinction the new files actually follow instead of numbering directories. A level with one child directory is grouping debt and folds upward.

## Name from the domain

Directory names carry domain words: `catalog`, `invoices`, `auth`. Kind words (`utils`, `helpers`, `common`, `misc`, `core`) name no change, so a directory so named collects unrelated files. A name a newcomer cannot map to the product's vocabulary fails, and renaming it belongs to the same move.

## Where the fixed kinds live

Tests sit beside the code they test where the toolchain allows it (Go `_test.go` files) or in the mirrored tree the toolchain requires (Java's `src/test`); the toolchain decides this, the layout does not. Generated files live under one generator-owned directory covered by the ignore file, and only the generator writes there. Scratch and local outputs stay outside the tree or in an ignored directory, never committed. Docs live at the root or under `docs/`, one home each.

## Move without breaking

Moves are mechanical and provable. List every reference into the moved paths first: `grep -rn` over sources, build files, and docs, recorded as one old-to-new pair list. Use `git mv` for every move so rename detection follows, and keep the whole move in one commit with no content edits, so review reads the moves and a revert undoes them. Apply the pair list to imports, build files, and doc links in the follow-up change, then let the build and a link check prove nothing dangles: `git log --follow <moved-file>` still reaches the history. A move without a passing build is not done.

## When not to reorganize

Decline the reorganization when the pain is one directory: move that directory and stop. Decline it mid-feature, since the feature's review then reads two reorganizations. Decline it when the touched code is scheduled for deletion; moving dead code is ceremony. Decline it when no change trips the grouping test: a layout nobody stumbles over is doing its job, and churn there costs review attention for nothing.

## What you hand back

The layout decision with its grouping rule, the mechanical move commit, the reference updates proven by the build, and the refusals with their reasons.
