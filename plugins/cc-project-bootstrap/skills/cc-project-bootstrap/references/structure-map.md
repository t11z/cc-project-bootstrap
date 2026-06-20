# Claude Code — Vollständige Strukturübersicht

## Globale Ebene (~/. claude/) — gilt für alle Projekte

```
~/.claude/
├── CLAUDE.md                    # Persönliche Präferenzen, globaler Stil
├── settings.json                # Globale Permissions, Hooks, Modell
├── commands/                    # Persönliche Slash-Commands
│   └── git-summary.md           → /user:git-summary
├── skills/                      # Persönliche Skills
│   └── my-workflow/
│       └── SKILL.md
├── agents/                      # Persönliche Agents
│   └── my-agent.md
└── projects/                    # Session-History (system-managed)
    └── -Users-thomas-myproject/
        └── <session-uuid>.jsonl

~/.claude.json                   # System-managed State (nicht manuell bearbeiten)
                                 # Enthält: auth, plugins, theme, usage-stats
```

## Projektebene (.claude/) — gilt nur für dieses Projekt

```
project-root/
│
├── CLAUDE.md                    # Projekt-Kontext (< 200 Zeilen!)
├── CLAUDE.local.md              # Persönliche Overrides (gitignored)
├── .mcp.json                    # MCP-Server-Konfiguration (ins Git)
│
└── .claude/
    ├── settings.json            # Permissions, Hooks, Modell (ins Git)
    ├── settings.local.json      # Persönliche Permission-Overrides (gitignored)
    │
    ├── rules/                   # Glob-basierte Regeln
    │   ├── code-style.md        # Gilt für: *.ts, *.tsx, ...
    │   ├── testing.md           # Gilt für: *.test.*, *.spec.*
    │   ├── api-conventions.md   # Gilt für: src/api/**
    │   └── security.md          # Gilt für: **
    │
    ├── commands/                # Slash-Commands → /project:<name>
    │   ├── review.md            → /project:review
    │   ├── fix-issue.md         → /project:fix-issue
    │   └── deploy-check.md      → /project:deploy-check
    │
    ├── skills/                  # Auto-triggered Workflows
    │   ├── deploy/
    │   │   ├── SKILL.md
    │   │   └── references/
    │   │       ├── aws-steps.md
    │   │       └── gcp-steps.md
    │   └── db-migration/
    │       └── SKILL.md
    │
    ├── agents/                  # Subagent-Personas
    │   ├── code-reviewer.md
    │   ├── security-auditor.md
    │   └── test-writer.md
    │
    └── hooks/                   # Hook-Scripts
        ├── validate-branch.sh
        ├── lint-changed.sh
        └── secrets-guard.sh
```

## Hierarchisches CLAUDE.md (nested)

```
project-root/
├── CLAUDE.md                    # Projekt-Root: allgemeine Regeln
└── src/
    ├── api/
    │   └── CLAUDE.md            # API-spezifisch: überschreibt Root
    └── frontend/
        └── CLAUDE.md            # Frontend-spezifisch: überschreibt Root
```

Claude priorisiert immer die spezifischste (tiefste) CLAUDE.md für den aktuellen Kontext.

## Prioritäts-Hierarchie (von hoch nach niedrig)

1. `.claude/settings.local.json` (persönliche Overrides)
2. `.claude/settings.json` (Team-Settings)
3. `~/.claude/settings.json` (globale Settings)
4. `CLAUDE.local.md` (persönliche Projekt-Overrides)
5. `CLAUDE.md` (Projekt-Root)
6. `~/.claude/CLAUDE.md` (globale Präferenzen)

## Git-Strategie

**Ins Git (geteilt):**
- `CLAUDE.md`
- `.mcp.json`
- `.claude/settings.json`
- `.claude/rules/*`
- `.claude/commands/*`
- `.claude/skills/*`
- `.claude/agents/*`
- `.claude/hooks/*`

**Gitignored (persönlich):**
- `CLAUDE.local.md`
- `.claude/settings.local.json`
- `.env`, `.env.*`

**.gitignore-Eintrag:**
```
CLAUDE.local.md
.claude/settings.local.json
```
