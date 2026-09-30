# GitHub PR Agent Coordination

`github-pr-agent-coordination` is an instruction-only Codex skill for designing or reviewing safe pull-request coordination between multiple AI agents.

It treats GitHub—not separate private chats—as the shared coordination channel. Issues, pull requests, reviews, checks, labels, and one idempotently updated bot status comment give every participant the same durable state and leave humans an auditable place to intervene.

## What problem it solves

Multiple agents may work on the same repository while being unable to read one another's conversations. A human then becomes a message relay, review state goes stale after new commits, and it becomes unclear whether a real agent integration or a person actually reviewed the code.

This skill helps Codex inspect the repository's existing controls and design a GitHub-native coordination flow that:

- records review state on the pull request;
- handles PR creation, reopening, draft/readiness changes, and new commits;
- moves stale approvals back to a review-needed state after a new commit;
- uses explicit labels for agent, human, and merge-readiness states;
- updates one clearly marked status comment without overwriting human comments;
- separates safe metadata automation from real code review integrations;
- reports missing setup instead of pretending that an integration exists.

## What it intentionally does not do

The skill is guidance, not an agent service or GitHub App. Installing it does **not**:

- create tokens, repository secrets, or provider credentials;
- install or configure an external agent integration;
- impersonate an agent, fabricate a review, or turn a comment into an approval;
- run untrusted pull-request code with privileged credentials;
- approve pull requests, merge them, or enable auto-merge;
- replace branch protection, required checks, or human judgment.

## Repository layout

```text
.
├── LICENSE
├── README.md
├── plugin.json
└── skills/
    └── github-pr-agent-coordination/
        └── SKILL.md
```

The portable plugin manifest and `skills/<name>/SKILL.md` layout follow the [official OpenAI plugin packaging guidance](https://developers.openai.com/plugins/build/plugins). The skill has no scripts, hooks, network service, or bundled credentials.

## Install in Codex

Clone the repository, then copy the named skill directory into your personal Codex skills directory:

```bash
git clone https://github.com/dinx393/github-pr-agent-coordination.git
mkdir -p ~/.codex/skills
cp -R github-pr-agent-coordination/skills/github-pr-agent-coordination ~/.codex/skills/
```

If the destination already exists, review the changes before replacing it. Start a new Codex session after installation so the skill is rediscovered.

For plugin-based distribution, the repository root is also a portable, single-skill plugin. Add it to a compatible personal or repository marketplace rather than copying the skill directory directly. See [Package your plugin](https://developers.openai.com/plugins/build/plugins) for current marketplace instructions.

## Invoke it

Ask Codex to use the skill explicitly:

```text
Use $github-pr-agent-coordination to design safe PR coordination for this repository.
Inspect the existing workflows, labels, branch protection, and installed agent
integrations first. Do not enable auto-merge or claim that an agent can review
unless its integration is actually configured.
```

You can also ask Codex to review an existing coordination workflow against the same safety model.

## Safe `pull_request_target` model

`pull_request_target` can be appropriate for metadata-only coordination because it runs in the context of the base repository and can update pull-request metadata. That privilege is also why the boundary must be strict.

A safe design should:

1. Trigger only on metadata-relevant PR events such as `opened`, `reopened`, `ready_for_review`, `converted_to_draft`, and `synchronize`.
2. Use the smallest permissions needed—typically `contents: read`, `pull-requests: write`, and `issues: write` only when labels or issue-style comments require it.
3. Run workflow logic from the default branch. Never check out, import, source, or execute code from the PR head in the privileged job.
4. Keep model calls, repository secrets, deployment credentials, approval, merge operations, and arbitrary PR code out of the workflow.
5. Update only a bot-owned status comment identified by a stable marker. Never edit or delete human comments.
6. Fail visibly when required labels or integrations are missing, rather than silently simulating progress.

Real code review should happen through a separately installed and explicitly authorized GitHub App, native integration, or tightly scoped credential. The metadata workflow may request or record that review; it must not impersonate it.

## Manual repository setup

Create these labels manually, or adapt the names once and keep the mapping explicit in the implementation:

| Label | Meaning |
| --- | --- |
| `coordination:review-needed` | The current PR head needs a fresh review. |
| `coordination:waiting-agent` | A configured external agent has been requested and has not responded. |
| `coordination:human-needed` | Automation cannot decide safely; a person must act. |
| `coordination:ready-to-merge` | Coordination checks are complete; repository protections still decide whether merging is allowed. |

Repository owners must also:

1. Review GitHub Actions settings and allow only the write permissions required for PR labels and the bot-owned status comment.
2. Configure branch protection or rulesets independently, including required reviews and checks.
3. Install each external agent's supported GitHub App or integration and grant only the permissions it needs.
4. Add the minimum required secret only when that integration's official setup requires one. This skill never creates, guesses, or embeds secrets.
5. Map each configured agent to an explicit trigger and verifiable response (for example, an App-authored review or required check).
6. Test the flow on a non-production pull request: creation, reopening, draft/readiness changes, new commits, repeated events, missing labels, and preservation of human comments.

Do not enable auto-merge as part of this setup. A `coordination:ready-to-merge` label is status metadata, not permission to merge.

## License

MIT. See [LICENSE](LICENSE).
