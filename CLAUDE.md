# CLAUDE.md

This file defines how Claude should behave in this repository.

## Project

`claude-experiments` is a personal sandbox for learning how to use Claude Code effectively — exploring features, writing experiments, and building intuition for working with AI agents.

## Behavior

- Proceed with local file operations (read, write, edit) without asking for confirmation.
- Be concise. Skip preamble and summaries — lead with the action or answer.
- When explaining Claude-related concepts, be direct and practical over theoretical.
- This repo is for experimentation: be willing to try things, iterate, and demonstrate how Claude features work.

## Code Style

- Language: TypeScript / JavaScript (Node.js)
- No unnecessary abstractions — keep experiments focused and minimal.
- Prefer self-explanatory code over comments.
- Do not add error handling for cases that can't happen in the current experiment scope.

## Installed Skills

The following agent skills are available in `.agents/skills/`:

| Skill | Purpose |
|---|---|
| `deploy-to-vercel` | Deploy projects to Vercel |
| `vercel-cli-with-tokens` | Vercel CLI usage with auth tokens |
| `vercel-composition-patterns` | React composition patterns (compound components, etc.) |
| `vercel-react-best-practices` | React/Next.js performance optimization |
| `vercel-react-native-skills` | React Native / Expo best practices |
| `web-design-guidelines` | UI/UX and accessibility review |

## Git

- Do not commit unless explicitly asked.
- Do not push unless explicitly asked.
- Branch: `main`

See [.claude/rules/deploy.md](.claude/rules/deploy.md) for commit and deploy preparation rules.

See [.claude/rules/frontend.md](.claude/rules/frontend.md) for frontend design pattern rules (Atomic Design).
