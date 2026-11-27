---
name: todos
description: 'Use when a message contains `TODO ADD: <requirement>`, tasks are too vague or read as headings, or the plan and the tree drifted apart. Also use when the user asks to sophisticate or update the todos, resync the task list, or say what to do next. Add mode records the requirement. Enhance mode sharpens a coarse list. Update mode reconciles a drifted list.'
---

# Keeping the task list true to the plan and the tree

One skill owns the native task list in three modes. Add mode records one `TODO ADD:` requirement in the task list and in its durable requirement source. Enhance mode rewrites a coarse list into executable work. Update mode reconciles the list with what landed and names the single next move.

Writes go through the todo tool, and add mode also writes one durable requirement source. Every write is local and reversible through version control or undo. No mode mutates a remote, and no mode implements the work it tracks.

## Pick the mode from the message

A message that contains `TODO ADD: <requirement>` selects add mode. A vague list, a list that reads as headings, or a request to sophisticate the todos selects enhance mode. A request to update the todos, resync the task list, or say what to do next selects update mode. A plan and a tree that drifted apart also select update mode.

When one message asks for more than one mode, run add, then update, then enhance. Finish each mode before the next starts. A new requirement goes on the list before the list is reconciled, and the reconciled list is the one to sharpen.

## Add mode: record one requirement in two places

You take a single `TODO ADD:` line and record it twice, once as a native todo and once in its durable requirement source, so both stay in agreement. Writes stay limited to one durable source and one native todo.

Add mode runs only when a message contains `TODO ADD: <requirement>`. It does not deepen coarse lists, resync drifted lists, retitle, reorder, or complete work. Those belong to enhance mode and update mode.

### Pick the one durable source

Use the current durable plan or specification when one clearly owns the work. If none exists, create or update `.outline/requirements/todo-add.md`. Never scatter the requirement across multiple documents. The durable source and the native todo must contain the same requirement in the same turn.

### Add the requirement without losing it

Follow this order, because the recovery rule depends on it.

1. Parse the text after the first `TODO ADD:` as one requirement. Reject an empty requirement.
2. Compare it semantically with the current durable source and the native todo list.
3. If it is already present in both, return `Duplicate: no change` and stop.
4. If it conflicts with an existing requirement, write nothing. Ask one question that presents the two incompatible forms and wait.
5. Assign the todo phase by the work's owning module or domain, not by wording or chronology.
6. Append the native todo first, with a backlink or stable reference to the requirement source. Call the todo tool with op append, phase set to the phase name, and items holding the single requirement.
7. Update the durable requirement source with the same normalized requirement and the native todo identifier.
8. Verify both sides resolve to the same text and ownership.

Refusals that hold for the whole flow: no empty requirement, no durable-source write when native todo creation failed, no silent removal of a retained todo on partial failure, and no write while a conflict is unresolved.

### Handle the one partial failure honestly

If native todo creation fails, do not touch the durable source. If the durable-source write fails after native todo creation, retain the native todo, mark it `requirement-source-write-failed`, and warn with the exact failed path and the recovery action. Do not claim success. Never silently remove the retained todo.

Return `Added`, `Duplicate`, `Conflict`, `Partial failure`, or `Refused`, plus the requirement-source path and native todo identifier when they exist. `Refused` covers a message without the `TODO ADD:` prefix and an empty requirement; it writes nothing and names the rule. Adding a requirement records it and never executes it.

## Enhance mode: turn a vague list into executable work

You rewrite a coarse task list so every item names one behavior, sits where its dependencies are met, and carries a checkable finish line. Nothing leaves this session as a file, version control, or remote change. When a live todo tracker holds the list, write the rewritten list back through the todo tool with op init and a list whose entries each carry a phase and items. Otherwise present the rewritten list in the reply and report that the tracker was left unchanged.

Enhance mode does not reconcile a drifted list against the tree (that is update mode) and does not add requirements (that is add mode). The full current task list must be present in context; without it, stop and say a task list is required.

### Diagnose every item once

Classify each item into exactly one bucket:

- atomic: one behavior, executable as written.
- compound: hides two or more separable pieces of work.
- vague: names an area, not a change.
- unordered: correct, but placed where its dependencies are unmet.
- unverifiable: executable, but nobody can tell when it is done.

### Split, order, then pin the finish line

1. Split every compound item into atomic tasks. A task is atomic when it names one behavior and can be executed without a further design decision. Length is not the test: a one-line task that still forces a choice between two approaches is compound. Stop when no task hides more than one decision.
2. Draw the dependency edges. B depends on A only when B cannot function without the output of A, not merely because A feels earlier. Mark genuinely independent tasks as parallel. Introduce a phase only where a real barrier exists, never to make the list look organized.
3. Give every task one observable acceptance test: a command, an output, or a state someone can check. Stop when zero items remain vague or unverifiable and every task carries exactly one criterion.

### Know when to stop instead of claiming success

If the list is empty, the finish condition holds vacuously; report that. If any task remains unclassified, unverifiable, or compound after one pass, do not claim success. Report the remaining items by class. On any failure, discard the rewritten list and present the diagnostic; the original list is unchanged.

The output is a rewritten task list where every item is atomic, ordered by real dependency, and annotated with one observable acceptance criterion each, or a terminal report listing the remaining unclassified or unverifiable items when the finish condition cannot be reached.

## Update mode: resync the list with what actually landed

You reconcile the task list with the codebase and name the single next move. Writes go through the todo tool, with rollback through undo, and no implementation happens here.

Update mode does not sophisticate coarse headings (that is enhance mode) and does not add requirements (that is add mode).

### Gather the three sources first

You need the current task list and the codebase state (file reads plus test or command output). Conversation context about design decisions is the optional third source for spotting overtaken items. Never mark an item complete without proof: test output, a command result, or a `path:line` reference. Someone saying an item is done is the claim under test, not proof. If the codebase state cannot be fully determined, return a partial result with explicit gaps rather than pretending the list is clean.

### Classify, prove, and write back

1. Read the current task list.
2. Inspect the codebase: read changed files, run targeted tests or commands, and check `path:line` references to determine what actually landed for every item.
3. Read conversation context for design changes that may have overtaken items.
4. Classify each existing item exactly once:
   - landed: done, with proof.
   - still-open: unchanged, still required.
   - overtaken: a design change made it unnecessary.
   - blocked: cannot proceed until something external clears.
   - newly-discovered: not on the list, found during the work.
5. Keep every landed claim attached to its proof (the test, the command output, or the `path:line` that demonstrates it). An unproven completion stays still-open; record the proof gap in the report.
6. Write the reconciled list back through the todo tool. If the tool replaces rather than patches the list, capture item identity, text, completion, and in-progress state before replacement and preserve those states in the reconciled list. Restore state using the tool current identifiers or matching contract, not stale display text or assumed ordering. Retagging does not reopen completed work. When the tool replaces the list, write it with op init and a list whose entries each carry a phase and items.
7. Drop each overtaken item with a one-line reason recorded in the report, never silently. A dropped item with no recorded reason is indistinguishable from a forgotten item.
8. Name exactly one next action: the first still-open item whose blockers are all clear, stated as a concrete action rather than a heading. When two items tie, pick the one that unblocks more of the remaining list. When every remaining item is blocked, name the one blocker that has to clear first, one answer rather than a list.
9. Emit the delta only: what changed classification, and why.

If the todo tool fails, do not fabricate a list state. Report the error; the list remains unchanged.

The delta report lists Completed, Still open, Overtaken, Blocked, and New; every landed item cites its proof, exactly one next action or one blocker is named, and zero stale items remain.
