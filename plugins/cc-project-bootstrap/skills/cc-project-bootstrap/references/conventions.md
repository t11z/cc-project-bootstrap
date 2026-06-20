# Konventionen — Single Source of Truth

Die festen Regeln für jedes gebootstrappte Projekt. ADR-Skill, ADR-Agent und Triage-Bot konsumieren dieselben Trigger-Kriterien von hier.

## CLAUDE.md-Philosophie

CLAUDE.md enthält **Verhaltensregeln, Konventionen und Wegweiser**. Sie dupliziert **niemals** Architekturentscheidungen oder andere flüchtige Informationen, die sich ändern können. CLAUDE.md soll nicht selbst zum Pflegefall werden.

Architektur lebt in `docs/decisions` als smADR. Wenn beim Schreiben einer CLAUDE.md der Drang aufkommt, eine Architekturentscheidung festzuhalten: das ist ein ADR.

Eine CLAUDE.md liegt zusätzlich auf der **ersten Unterverzeichnisebene** vom Repo-Root — aber nur für code-/konventionstragende Dirs. Sie legt die speziellen Konventionen, Regeln und Wegweiser für genau diesen Code-Teil fest, ohne Root-Regeln zu wiederholen.

## ADR — wann ja, wann nein

**Ein ADR ist zu schreiben, wenn eine Entscheidung:**
- schwer zurückzunehmen ist, oder
- mehrere Schichten oder eine öffentliche Schnittstelle oder einen Vertrag umspannt, oder
- eine Invariante, einen Provider oder einen Stack-Building-Block festhält, oder
- eine plausible Alternative verdrängt, die es wert ist, dokumentiert zu werden.

**Ein ADR ist NICHT zu schreiben für:**
- reine Implementierungsdetails,
- lokale Refactors,
- Nomenklatur,
- Helper-Strukturen,
- Test-Layout.

Diese sind im Code-Review gesettled und, wo stabil, in CLAUDE.md geregelt.

**Bei Unsicherheit:** ADR im Status `proposed` eröffnen.

## ADR-Lifecycle

ADRs sind **append-only**. Sobald `accepted`, werden sie nicht mehr editiert. Wird eine Änderung nötig: neuen ADR schreiben, alten via `supersedes`/`superseded_by` verknüpfen. Schema und Sektionen: `adr-smadr.md`.

## Sprachregime

- **Konversation mit dem Architekten:** in seiner Sprache.
- **Repository-Artefakte** (Codekommentare, Dokumentation, Issues, Pull Requests, Commit-Messages, ADRs, CLAUDE.md): durchgängig **Englisch**.

## Auth

Nie gegen `ANTHROPIC_API_KEY` authentifizieren. Immer `CLAUDE_CODE_OAUTH_TOKEN`. Begründung und Konsequenzen für den Security-Review: `auth-oauth.md`.

## Supply-Chain

Alle GitHub-Actions auf Commit-SHA gepinnt, mit `# vX.Y.Z`-Kommentar. Dependabot hält sie aktuell. Verfahren: `action-pins.md`.
