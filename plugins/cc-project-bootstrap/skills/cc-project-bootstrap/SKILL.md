---
name: cc-project-bootstrap
description: Setzt ein neues Claude-Code-Projekt nach Thomas' festen Konventionen auf — CLAUDE.md-Hierarchie, smADR in docs/decisions, Slash-Commands/Agents/Skills (ADR + Security-Review), GitHub-Workflows (OAuth-Security-Review, smADR-Validierung, gegateter Issue-Triage-Bot, CI), Issue/PR-Templates, Dependabot, optional Prisma-AIRS-Hooks. Verwende diesen Skill immer wenn ein neues Repo/Projekt für Claude Code eingerichtet, gebootstrappt oder strukturiert werden soll — "neues Projekt aufsetzen", "Repo bootstrappen", "Claude Code Projektstruktur", "CLAUDE.md und Workflows anlegen", "Projekt-Setup", "richte mir das Repo ein". Auch bei Teilaufträgen (nur Workflows, nur ADR-Setup, nur Triage-Bot) hier starten, um die Konventionen konsistent zu halten. Dieser Skill ersetzt cc-full-setup und adr-write; deren brauchbare Referenzen liegen unter references/.
---

# CC Project Bootstrap

Bootstrappt ein Repository auf Thomas' Standard. Meinungsstark, stack-agnostisch, OAuth-only. Jede Phase ist einzeln nutzbar — bei Teilaufträgen direkt in die relevante Phase springen.

**Pflichtlektüre vorab — in dieser Reihenfolge lesen, bevor irgendetwas gestampft wird:**
- `references/conventions.md` — die harten Regeln (CLAUDE.md-Philosophie, ADR-Trigger-Kriterien, Sprachregime, Append-only-Lifecycle). Single Source of Truth.
- `references/auth-oauth.md` — das Auth-Modell. Nie gegen `ANTHROPIC_API_KEY`, immer `CLAUDE_CODE_OAUTH_TOKEN`. Erklärt den Security-Review-Patch und den App-Token-Pfad.
- `references/action-pins.md` — die aktuell aufgelösten Action-SHAs und das Re-Resolve-Verfahren.
- `references/adr-smadr.md` — kondensierte smADR-Spezifikation (Frontmatter, Body, File-Naming). Basis für ADR-Skill und -Agent.

Vertiefung bei Bedarf: `hooks-cookbook.md`, `permissions-patterns.md`, `agent-patterns.md`, `rules-templates.md`, `structure-map.md`.

---

## Prozessübersicht

```
Phase 0 → Kontext klären (Repo-Pfad, Projekt, Team/Solo, AIRS ja/nein)
Phase 1 → Action-Versionen recherchieren + auf SHA pinnen
Phase 2 → Root-CLAUDE.md
Phase 3 → Sub-CLAUDE.md (nur code-/konventionstragende Dirs)
Phase 4 → docs/decisions: smADR-Template + ADR-Index
Phase 5 → .claude/: Commands, Agents, Projekt-ADR-Skill
Phase 6 → .github/: Workflows, Issue/PR-Templates, Dependabot
Phase 7 → Setup-Hilfe: SETUP.md + GitHub-App-Bootstrap-Skript
Phase 8 → Optional: Prisma-AIRS-Hooks (nur wenn gewünscht)
Phase 9 → Finalisierung + Checkliste
```

---

## Phase 0 — Kontext klären

Vor dem Stampfen klären, knapp:

1. **Repo-Pfad / Projektname** — wo wird gebootstrappt? Existiert schon Code?
2. **Was wird gebaut** — bestimmt, welche Sub-Dirs eine CLAUDE.md bekommen und wie die CI gefüllt wird. Den Stack hier *erkennen*, nicht vorgeben (siehe Phase 6).
3. **Team oder Solo** — Team → `.claude/settings.json`, `.mcp.json` ins Git. Solo → mehr global.
4. **Prisma AIRS gewünscht?** — Default nein. Wenn ja: nur Hooks (Phase 8).
5. **GitHub-Org/Repo-Slug** — für App-Token-Setup und Workflow-Permissions.

Nicht über den Stack hinausfragen. Stack-Templates werden nicht festgezurrt; die CI wird in Phase 6 an das erkannte Projekt angepasst.

---

## Phase 1 — Action-Versionen recherchieren + pinnen

Thomas' Vorgabe: aktuell von GitHub empfohlene Action- und Runner-Versionen verwenden, und alle Actions auf Commit-SHA pinnen (Supply-Chain).

Verfahren pro Action (`actions/checkout`, `actions/setup-*`, `actions/create-github-app-token`, `actions/github-script`, `anthropics/claude-code-action`, `zircote/structured-madr`, sowie der gepinnte Klon von `anthropics/claude-code-security-review`):

1. Aktuell empfohlene **Version** ermitteln (Releases/Tags des Action-Repos).
2. Tag auf **Commit-SHA** auflösen:
   ```
   git ls-remote https://github.com/<owner>/<repo>.git "refs/tags/<tag>^{}" | awk '{print $1}'
   ```
   (Leeres Ergebnis → ohne `^{}` erneut; das ist ein nicht-annotiertes Tag.)
3. In jeden Workflow schreiben als `uses: <owner>/<repo>@<sha> # <tag>` — SHA bindet, Kommentar macht lesbar und Dependabot-tauglich.

Runner: GitHub-Default `ubuntu-latest` verwenden (aktuell 24.04). Reproduzierbarkeits-Pin `ubuntu-24.04` ist möglich, nicht Pflicht.

`references/action-pins.md` enthält die zuletzt aufgelösten SHAs als Startwerte. Beim Bootstrap **neu auflösen**, da Versionen driften, und die Referenz mitaktualisieren.

---

## Phase 2 — Root-CLAUDE.md

`assets/CLAUDE.md.tmpl` nehmen, Platzhalter füllen, nach `<repo>/CLAUDE.md` schreiben.

Harte Regel aus `references/conventions.md`: CLAUDE.md enthält **Verhaltensregeln, Konventionen, Wegweiser** — keine Architekturentscheidungen, keine flüchtigen Fakten. Architektur lebt in `docs/decisions` als smADR. Wenn beim Füllen der Drang aufkommt, eine Architekturentscheidung in die CLAUDE.md zu schreiben: das ist ein ADR, kein CLAUDE.md-Eintrag.

Pflicht-Inhalte: Sprachregime (Konversation in der Sprache des Architekten, Repo-Artefakte durchgängig Englisch), Verweis auf `docs/decisions` + ADR-Trigger-Kriterien, Append-only-/Supersede-Lifecycle, Auth-Hinweis (`CLAUDE_CODE_OAUTH_TOKEN`, nie `ANTHROPIC_API_KEY`), Wegweiser auf `.claude/commands` und `.claude/agents`.

---

## Phase 3 — Sub-CLAUDE.md

Auf der **ersten Unterverzeichnisebene** vom Repo-Root je eine CLAUDE.md anlegen — **nur für code-/konventionstragende Dirs** (`src/`, `backend/`, `frontend/`, `infra/`, `packages/*` o.ä.). Nicht für `docs/`, `.github/`, `.claude/`, Asset-Ordner.

`assets/CLAUDE.subdir.md.tmpl` nehmen. Inhalt: lokale Konventionen, Regeln und Wegweiser für genau diesen Code-Teil. Keine Wiederholung der Root-Regeln, keine Architektur.

---

## Phase 4 — docs/decisions (smADR)

`assets/docs/decisions/` nach `<repo>/docs/decisions/` kopieren:
- `adr-template.md` — vollständiges smADR-Template (Frontmatter + alle Body-Sektionen inkl. Audit).
- `README.md` — ADR-Index, File-Naming, Lifecycle, Trigger-Kriterien.

Pfad ist `docs/decisions` (smADR-Validator-Default). Schema strikt nach `references/adr-smadr.md`: englische Body-Sektionen, YAML-Frontmatter, Pflicht-Audit-Sektion, Risk-Assessment pro Option. Append-only ab `accepted`; Änderung nur per neuem ADR mit `supersedes`.

---

## Phase 5 — .claude/

`assets/.claude/` nach `<repo>/.claude/` kopieren:
- `commands/adr-new.md` — Slash-Command, der einen neuen smADR anlegt (delegiert an den ADR-Skill/-Agent).
- `commands/security-review.md` — Anthropics Original, unverändert. Auth-agnostisch: läuft in der bereits OAuth-authentifizierten lokalen Session, braucht keinen Key.
- `agents/adr-author.md` — Subagent, schreibt smADRs nach Schema.
- `agents/security-reviewer.md` — Subagent für lokalen Security-Review.
- `skills/adr-write/SKILL.md.tmpl` → beim Stampfen als `skills/adr-write/SKILL.md` ablegen (projektlokaler ADR-Skill, smADR). Die `.tmpl`-Endung existiert nur im Bundle, damit der Skill-Zip beim claude.ai-Upload genau eine `SKILL.md` enthält.

ADR-Trigger-Kriterien stehen in `references/conventions.md` und werden sowohl hier (ADR-Skill/-Agent) als auch im Triage-Bot-Prompt (Phase 6) konsumiert — eine Quelle, zwei Konsumenten.

---

## Phase 6 — .github/

`assets/.github/` nach `<repo>/.github/` kopieren, dann SHAs aus Phase 1 einsetzen.

**Workflows:**
- `security-review.yml` — OAuth-Rewire von `anthropics/claude-code-security-review`. Klont die Action auf gepinntem SHA, patcht den `ANTHROPIC_API_KEY`-Presence-Guard (akzeptiert dann `CLAUDE_CODE_OAUTH_TOKEN`), fährt `github_action_audit.py` mit `ENABLE_CLAUDE_FILTERING=false`. Details und Patch-Rationale: `references/auth-oauth.md`. Nur auf `pull_request` von vertrauenswürdigen Quellen; minimale Permissions.
- `adr-validate.yml` — `zircote/structured-madr` (gepinnt) gegen `docs/decisions`.
- `issue-triage.yml` — `anthropics/claude-code-action` (gepinnt) mit `claude_code_oauth_token` + GitHub-App-Token. **Auto** bei `issues: opened` und `pull_request: opened`: triagieren, labeln, kommentieren, ADR-Breaking-Change prüfen (nach den Kriterien aus `conventions.md`) und bei Treffer kommentieren + schließen. **Gegated**: PR-Generierung nur auf `@claude implement` durch einen Maintainer mit Write-Permission (expliziter Permission-Check-Step, da der App-Token den eingebauten Actor-Check der Action umgeht).
- `ci.yml` — stack-agnostisches Skelett. Beim Bootstrap an das in Phase 0 erkannte Projekt anpassen (Lint/Test/Build). Keine festen Stack-Annahmen im Skill.

**Templates:** `ISSUE_TEMPLATE/{bug_report,feature_request,config}.yml`, `PULL_REQUEST_TEMPLATE.md`.

**Dependabot:** `dependabot.yml` fürs `github-actions`-Ökosystem (wöchentlich). Hält die SHA-Pins via PR aktuell — löst den „keine Auto-Patches"-Nachteil des Pinnings.

---

## Phase 7 — Setup-Hilfe

`assets/SETUP.md` und `assets/scripts/bootstrap-github-app.sh` nach `<repo>/` kopieren.

Macht dem Architekten das einmalige Secret-Setup leicht: OAuth-Token via `claude setup-token` erzeugen, GitHub-App anlegen (vorausgefüllte Permissions issues/pull-requests/contents:write), App ins Repo installieren, Secrets `CLAUDE_CODE_OAUTH_TOKEN` + `APP_ID` + `APP_PRIVATE_KEY` setzen (via `gh secret set`, wenn die gh-CLI vorhanden ist). Das Skript führt interaktiv durch die Schritte und prüft Voraussetzungen.

---

## Phase 8 — Prisma AIRS (optional, Hooks only)

Nur wenn in Phase 0 bejaht. Claude-Code-Hooks-Integration nach
`https://github.com/PaloAltoNetworks/prisma-airs-integrations/tree/main/Anthropic/claude-code-hooks`:
inline Prompt-Scan plus Pre/Post-Tool-Scan über die AIRS-Runtime. Voraussetzung: AIRS-API-Key/Strata-Profil als lokales Secret (nicht ins Repo). Bei Bedarf das aktuelle Hook-Skript der Integration ziehen und in `.claude/settings.json` als Hooks verdrahten. MCP- und Skill-Flavor sind bewusst draußen.

---

## Phase 9 — Finalisierung

Checkliste durchgehen und dem Architekten zurückspiegeln:

- [ ] Root-CLAUDE.md ohne Architekturinhalte, mit Sprachregime + Auth-Hinweis
- [ ] Sub-CLAUDE.md nur in code-/konventionstragenden Dirs
- [ ] docs/decisions mit smADR-Template + README, Pfad korrekt
- [ ] .claude/ Commands + Agents + ADR-Skill vorhanden
- [ ] Alle Actions auf SHA gepinnt, `# vX.Y.Z`-Kommentar dran, in Phase 1 neu aufgelöst
- [ ] security-review.yml: OAuth, Guard-Patch greift (Patch-Selbsttest grün), Filtering aus
- [ ] issue-triage.yml: Auto-Triage offen, Implement gegated, App-Token verdrahtet
- [ ] adr-validate.yml gegen docs/decisions
- [ ] Issue/PR-Templates + dependabot.yml
- [ ] SETUP.md + bootstrap-github-app.sh, Secrets-Liste klar
- [ ] AIRS nur falls gewünscht
- [ ] Hinweis an den Architekten: alten `cc-full-setup`- und `adr-write`-Skill entfernen (durch diesen ersetzt)

Auth-Realität benennen, ohne sie zu dokumentieren-als-Pflegefall: `CLAUDE_CODE_OAUTH_TOKEN` via `claude setup-token` ist CI-tauglich, aber endlich. Bei Auth-Fehlern in CI Token neu erzeugen und Secret aktualisieren.
