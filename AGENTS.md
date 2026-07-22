# Tasks Workspace

The root chat is the user's single entry point for all tasks.

## Task files

```text
tasks/<name>/
├── spec.md
├── plan.md
├── progress.md
└── artifacts/
    └── review.md
```

Keep code in its real repository. `spec.md` must record the absolute local repository path, optional remote, base branch, task branch, requirements, and acceptance criteria.

## Flow

1. Use the Superpowers `brainstorming` skill to clarify the request and produce an approved `spec.md`. Then use the Superpowers `writing-plans` skill as-is to produce `plan.md`; the task-local path overrides its default plan path.
2. Do not implement until the user approves both files.
3. Start Codex's built-in `worker` for that task. The Worker creates or resumes its own native Codex Goal (the Goal exposed by `/goal`) and continues until the complete Spec and Plan are verified.
4. The Worker keeps `progress.md` current and may spawn child agents. When multiple agents write to the same repository, use the Superpowers `using-git-worktrees` skill so every writer has a separate worktree and branch. The Worker verifies and integrates all branches into the task branch.
5. Start a fresh custom `reviewer` on the integrated task branch.
6. `SPEC_INCOMPLETE` or `P0_BUG` returns to the same Worker for another Goal turn and fresh review. `PASS` lets the root chat report completion. Non-P0 findings remain in `artifacts/review.md` and do not block completion.

Independent tasks may run in parallel. The user normally talks to the root chat, but may open and steer any Worker or Reviewer thread directly. Task files are the durable source of truth.
