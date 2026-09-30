---
name: github-pr-agent-coordination
description: "Set up or review GitHub pull-request coordination among multiple AI agents when GitHub should be their shared workflow and review channel."
---

# Github Pr Agent Coordination

Use this skill when a team wants agents such as Claude, ChatGPT, or Codex to coordinate through issues and pull requests instead of a person relaying updates between separate chats.

## Outcome

Make GitHub the source of truth. A PR should visibly carry its coordination state, the review conversation, and any human escalation. Keep the main planning chat as a coordinator; do not assume agents can read one another's private conversations.

## Before changing a repository

Inspect the current workflows, PR templates, labels, branch protections, available agent integrations, and repository instructions. Preserve unrelated changes. Establish which agent can actually act through GitHub and which can only be mentioned or triggered manually.

Do not claim an agent will review code unless its GitHub App, token, connector, or supported native integration is already configured. Never invent secrets or provider access.

## Coordination design

Use GitHub PR comments, reviews, checks, and labels as the shared board. Cover at least PR creation, reopening, readiness changes, conversion to draft, and new commits. New commits must move stale review state back to a review-needed state. Use one clearly marked bot status comment and update it idempotently; do not overwrite human comments.

Use labels for states such as review needed, waiting for a named reviewer, human decision needed, and ready to merge. If labels must be created by a repository owner, document their exact names and fail visibly rather than silently creating or faking them.

## Workflow safety

For metadata-only coordination that needs write access, a `pull_request_target` workflow may be appropriate only when it checks out and runs trusted code from the default branch, never PR-head code, and uses the smallest permissions needed. Keep model calls, repository secrets, approval, merge, deployments, and arbitrary PR code out of that workflow. Prefer a separate, explicitly configured integration for real agent reviews.

Do not enable auto-merge or auto-approval unless the user explicitly asks and the repository's protections make it safe.

## External setup and handoff

Document the owner steps separately: creating labels, enabling required GitHub Actions permissions, installing any provider GitHub App, adding the minimum necessary secret, and validating the integration on a test PR. Distinguish clearly between what works immediately after merge and what waits for that setup.

Test state transitions, idempotent comments, label-missing behavior, and protection against human-comment changes. Run the relevant project suite and report exact evidence, remaining limitations, and whether publishing to GitHub still needs the user's approval.
