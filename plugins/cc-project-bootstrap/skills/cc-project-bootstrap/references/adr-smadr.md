# smADR — kondensierte Spezifikation

Structured MADR (smadr.dev), Spec 1.0. Erweitert MADR um YAML-Frontmatter, Risk-Assessment pro Option und eine Pflicht-Audit-Sektion. Body-Sektionen englisch.

## Ablage & File-Naming

- Verzeichnis: `docs/decisions/`
- Datei: `{NNNN}-{slug}.md` (z.B. `0001-use-postgresql-for-primary-storage.md`) oder `adr_{NNNN}.md`
- `{NNNN}` zero-padded, fortlaufend.

## Frontmatter (Pflichtfelder)

```yaml
---
title: "Use PostgreSQL for Primary Storage"   # <=60 Zeichen, Title Case
description: "One-sentence summary"            # <=160 Zeichen
type: adr                                      # exakt "adr"
category: architecture                         # architecture|api|security|performance|infrastructure|migration|integration|data|testing
tags: [database, postgresql]                   # >=1, lowercase-hyphen
status: proposed                               # proposed|accepted|deprecated|superseded
created: 2025-01-15                            # ISO 8601
updated: 2025-01-15                            # >= created
author: "Architecture Team"
project: my-application
---
```

Optional: `technologies: [...]`, `audience: [developers, architects]`, `related: [adr_0001.md]`.

## Body-Sektionen (Pflicht, in dieser Reihenfolge)

1. `# ADR-{NNNN}: {Title}` (H1)
2. `## Status` — aktueller Status; bei Ersetzung `Supersedes ADR-XXXX`.
3. `## Context` → `### Background and Problem Statement`, optional `### Current Limitations`.
4. `## Decision Drivers` → `### Primary Decision Drivers`, `### Secondary Decision Drivers`.
5. `## Considered Options` → je Option ein `### Option N: …` mit:
   - **Description**
   - **Technical Characteristics**
   - **Advantages**
   - **Disadvantages**
   - **Risk Assessment**: Technical / Schedule / Ecosystem je `{Low|Medium|High}` + Begründung (SHOULD)
   - **Disqualifying Factor** (optional)
6. `## Decision` — gewählte Option + Implementierungsdetails.
7. `## Consequences` → `### Positive`, `### Negative`, `### Neutral`.
8. `## Decision Outcome` — Zusammenfassung + Mitigations.
9. `## Related Decisions`, `## Links`, `## More Information` (Date/Source/Related ADRs).
10. `## Audit` — Pflicht. Mindestens ein Eintrag `### {YYYY-MM-DD}` mit **Status** (Pending|Compliant|Non-Compliant|Partial), **Findings**-Tabelle (Finding | Files | Lines | Assessment), **Summary**, **Action Required**. Neue ADRs: Initialeintrag `Status: Pending`.

## Lifecycle

- `proposed` — Entwurf, änderbar.
- `accepted` — steht fest, Body **immutable**. Spätere Änderung → neuer ADR mit `supersedes`.
- `deprecated` — überholt ohne konkreten Nachfolger.
- `superseded` — durch konkreten ADR ersetzt; `superseded_by` muss gefüllt sein.

## Validierung

`zircote/structured-madr` (GitHub Action) prüft Frontmatter gegen JSON-Schema und Body-Struktur/Reihenfolge gegen `docs/decisions`. Gültig wenn: Frontmatter valides YAML, alle Pflichtfelder gültig, alle Pflicht-Sektionen present, Audit-Sektion mit >=1 Eintrag.

## Trigger

Wann ein ADR fällig ist: `conventions.md`. Bei Unsicherheit `proposed` eröffnen.
