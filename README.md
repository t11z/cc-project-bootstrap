# cc-project-bootstrap

A [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces)
that ships the **CC Project Bootstrap** skill — opinionated, stack-agnostic,
OAuth-only scaffolding for new Claude Code projects.

## What's inside

| Plugin | Description |
| --- | --- |
| `cc-project-bootstrap` | Bootstraps a repo to Thomas' conventions: CLAUDE.md hierarchy, smADR decision records in `docs/decisions`, ADR + security-review commands/agents/skills, OAuth-only GitHub workflows (security review, smADR validation, gated issue-triage bot, CI), issue/PR templates, Dependabot, and optional Prisma AIRS hooks. |

## Install

In Claude Code:

```text
/plugin marketplace add t11z/cc-project-bootstrap
/plugin install cc-project-bootstrap@t11z
```

To pin a branch or tag:

```text
/plugin marketplace add t11z/cc-project-bootstrap@main
```

Once installed, the skill activates whenever you ask Claude Code to set up,
bootstrap, or structure a new project/repository.

## Layout

```
.
├── .claude-plugin/
│   └── marketplace.json          # Marketplace catalog
└── plugins/
    └── cc-project-bootstrap/
        ├── .claude-plugin/
        │   └── plugin.json       # Plugin manifest
        └── skills/
            └── cc-project-bootstrap/
                ├── SKILL.md      # Skill definition
                ├── references/   # Conventions, auth model, smADR spec, …
                └── assets/       # Templates: CLAUDE.md, .github/, .claude/, docs/, scripts/
```

## Local development

Test the marketplace from a checkout without publishing:

```text
/plugin marketplace add ./cc-project-bootstrap
/plugin install cc-project-bootstrap@t11z
```
