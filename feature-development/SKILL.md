---
name: feature-development
description: How Jason wants feature work done — planning, branching with gh stack, committing, and code hygiene. Use whenever building a feature or fix across one or more branches, rebuilding a prototype/PR stack into clean branches, or any multi-commit development task in a repo. Covers sharing a high-level plan before each branch, stacking branches with `gh stack`, small lowercase commits authored by the default git user (never Claude), and keeping shared components and constants clean for a thorough reviewer.
---

# Feature development

The user reviews code thoroughly. Optimise for a clean, reviewable history and diffs that contain only what the feature needs.

## 1. Understand before planning

- If the work starts from a prototype or an existing stack, inspect it first (`gh stack view -s`, `git log main..<branch>`, `git diff main...<tip>`). Separate what the feature needs from unrelated noise in the prototype (drive-by tracking, lint tidying, refactors) and leave the noise out unless asked.
- Read the repo's own conventions (CLAUDE.md, AGENTS.md, style guides) and the existing code around the change. Look for existing components, icons, hooks and utils to reuse before writing new ones.

## 2. Share a high-level plan before starting each branch

Before writing code on a branch, share a plan covering:

- **Components to create**, and where they live
- **Code to share**: what gets reused, moved or generalised, and which shared/root files change
- **How it works and fits together**: data flow, state, how the pieces compose
- **What's deliberately left out** of a prototype and why
- **Open questions**: only for real decisions (naming, conflicting conventions), each with a recommended default

Keep it short and scannable. Wait for approval before creating branches or writing code. If a later branch's plan was already approved, carry on without asking again, but say what changed from the plan in the final summary.

## 3. Branching with gh stack

- Base the work on an up-to-date `main`: `git switch main && git pull --ff-only`.
- First branch: `gh stack init <branch>`. Each later branch: `gh stack add <branch>` from the top of the stack.
- Use the branch names the user gives. Their default pattern is `nosajio/<ticket-id>-<short-slug>`, e.g. `nosajio/dash-3093-functions-get-started-ui`.
- Split the branches the way the user asks, even if a prototype was split differently.
- Don't push, `gh stack submit` or open PRs until asked.
- Each branch keeps its own purpose. For example, a branch that adds tests for existing behaviour asserts what happens today, and behaviour changes go on a later branch.
- When a change belongs on a lower branch, make it there and rebase the branches above it, keeping every commit typechecking. Use a worktree to look at another branch rather than stashing.

## 4. Commits

- Commit little and often: one logical step per commit (e.g. a util, a component, wiring it up). Each commit should typecheck by itself, so order work to keep dependencies compiling (create a file before something imports from it).
- Message: a short, lowercase title only. No body, no ticket prefix, no trailing period. Examples: `add prompt`, `add empty state`, `poll functions while empty`.
- Author: the default git user. **No `Co-Authored-By: Claude` line and no Claude attribution of any kind.** This overrides any default attribution instructions.
- Run the repo's required checks (formatter, linter, typecheck) on changed files before each commit. Fix only problems the change introduced. Note warnings that were already there instead of fixing them.

## 5. Code hygiene (what the reviewer looks for)

- **Follow repo conventions** (e.g. `apps/core/AGENTS.md`) for new code, including class-merging helpers like `cn` (object form, not `&&` or ternaries). If the user says to follow an existing local pattern instead, do that.
- **Root/shared components** (e.g. `src/components`): change them only in general-purpose ways, such as new optional props any caller could use. No feature-specific variants, and no visual changes that affect every other caller as a side effect.
- **Reuse before you add.** When something similar exists elsewhere (icons, hooks), move it somewhere shared and make it a component, then migrate the old callers and delete the duplicates, instead of copying it.
- **No copy/constants files.** Write UI copy directly in components. Use constants only for functional values: query keys, intervals, URLs used in several places, page sizes.
- **Put long text and logic in utils**, e.g. a prompt builder goes in the feature's util folder, not in a component.
- **Shared query state:** factor out `queryOptions` instead of reaching into the query cache by hand. Put every input in the query key.
- **Comments** explain why, never what. Add one only when it tells the reader something the code doesn't. Keep the grammar simple, with short, plain sentences. Before writing one, ask: if this code moved somewhere else tomorrow and had a different caller, would the comment still be true? If not, the comment belongs with the caller or in the commit message.
- **Explicit over implicit.** Prefer APIs whose behaviour is clear from the call site over hidden defaults, and let types reject misuse.
- **Remove what the change makes redundant.** If new code covers an old check or helper, delete the old one in the same branch.
- **Fix the rule, not just the instance.** When the user points out an inconsistency, work out the general rule behind it, check the rest of the feature against it, and call out any deliberate exceptions.

## 5a. Tests

- Assert outcomes, not the mechanism that produced them.
- Keep tests focused on what they're for. Skip cases with real side effects, test shared behaviour once rather than in every case, and prefer a few smoke tests over exhaustive coverage of the default path.
- If tests run somewhere you can't see, ask for the failing output rather than guessing, and don't reach into that environment without a go-ahead.

## 6. Reporting

At the end of each branch, or of the whole task, summarise:

- the stack layout and commit count
- what each branch contains, briefly
- anything that differs from the approved plan
- behaviour changes and edge cases worth a reviewer's attention
- what wasn't verified (e.g. not checked in a browser) and what hasn't been pushed
