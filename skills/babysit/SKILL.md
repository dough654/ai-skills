---
name: babysit
description: Supervised build mode. Before making ANY file change, the agent presents a clear diff preview and asks for explicit user approval. Mimics Claude Code's default confirmation mode. Use this skill when the user wants to review every change before it's applied, wants supervised mode, asks for babysit mode, or says things like "ask me before changing anything", "confirm changes with me", "supervised mode", "review each change", or "don't auto-apply". User-invocable with /babysit.
---

# Babysit Mode

You are now in **supervised build mode**. You have full capability to research, plan, and implement changes -- but you MUST get explicit user approval before every file modification.

Think of this as the middle ground between Plan mode (read-only) and Build mode (auto-apply). You do the work, but the user confirms every write.

## Core Rule

**NEVER modify a file without explicit user approval.** This is non-negotiable.

This applies to ALL file-modifying operations:
- Editing existing files (Edit tool)
- Writing new files (Write tool)
- Any other operation that changes file contents on disk

This does NOT require approval (proceed freely):
- Reading files
- Searching / grepping / globbing
- Running bash commands that don't modify files (git status, ls, cat, test runs, etc.)
- Using the TodoWrite tool
- Launching subagents for research (Task tool with explore/general)

## The Approval Loop

For every file change, follow this exact sequence:

### 1. Present the Change

Before calling any file-modifying tool, show the user what you intend to do. Format it clearly:

**For edits to existing files**, show a diff-style preview:

```
File: path/to/file.ext

- old line being replaced
+ new line replacing it
```

Include enough surrounding context (3-5 lines above and below) so the user can understand where the change sits. If the change is large, summarize the intent first, then show the full diff.

**For new files**, show the full proposed content:

```
New file: path/to/new-file.ext

<full file content>
```

**For multiple related changes in the same file**, batch them into a single preview rather than asking one at a time. Show all the edits together so the user can evaluate them as a cohesive change.

### 2. Ask for Approval

After presenting the change, use the `question` tool to ask:

```
question({
  questions: [{
    question: "Apply this change to <filepath>?",
    header: "Approve change",
    options: [
      { label: "Accept", description: "Apply the change as shown" },
      { label: "Reject", description: "Skip this change entirely" }
    ]
  }]
})
```

The `custom` option is enabled by default, which adds a "Type your own answer" option. This lets the user provide modification requests without you needing to add an explicit option for it.

### 3. Handle the Response

- **Accept**: Apply the change exactly as shown. Proceed to the next change.
- **Reject**: Do NOT apply the change. Acknowledge the rejection and move on. Do not ask again unless the user brings it up.
- **Custom response**: The user is providing feedback or requesting modifications. Revise the change based on their input and present the updated diff for approval. Repeat the loop.

## Batching Guidelines

- **Same file, related changes**: Batch into one preview and one approval.
- **Same file, unrelated changes**: You may batch or separate -- use judgment based on complexity.
- **Different files**: Separate approvals per file, unless the changes are trivially connected (e.g., updating an import in file A and the corresponding export in file B).
- **Large refactors touching many files**: Present a summary of all planned changes first, then go file-by-file for approval. The user may choose to approve the whole batch at that point.

## Bash Commands

For bash commands that **modify files** (e.g., `sed`, `mv`, `rm`, `cp`, writing redirects), treat them the same as file edits -- show what will happen and ask for approval first.

For bash commands that are **read-only or build/test commands** (e.g., `git status`, `npm test`, `cargo build`, `ls`), run them freely without asking.

If you're unsure whether a command modifies files, ask.

## Workflow Integration

This mode works naturally with the rest of your workflow:

- **Planning**: Plan freely. Discuss approaches. Create todo lists. None of this requires approval.
- **Research**: Read files, search code, explore the codebase. No approval needed.
- **Implementation**: This is where the approval loop kicks in. Every file write goes through the loop.
- **Testing**: Run tests freely after changes are applied. If tests fail and you need to fix something, the fix goes through the approval loop again.
- **Git operations**: `git add`, `git commit`, and `git push` should be treated as significant actions. Ask before committing or pushing -- show the commit message and what's being committed.

## Session Behavior

- If the user says "just do it" or "auto-approve the rest" or similar, remind them that they can switch to Build mode (Tab key) for unrestricted changes. Do not disable the approval loop within this skill -- that defeats its purpose.
- If the user approves several changes in a row without modification, do NOT start skipping approvals. Every change still goes through the loop.
- Stay efficient. Don't over-explain the approval process each time. After the first few changes, the user will understand the rhythm. Keep the diff previews clean and the approval prompts brief.

## Starting the Session

When this skill is loaded:
1. Acknowledge that you're now in supervised/babysit mode
2. Briefly explain: you'll show diffs and ask before every file change
3. Continue with whatever task the user needs -- this mode is about HOW you make changes, not WHAT you work on
