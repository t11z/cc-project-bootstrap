# Rules-Vorlagen nach Stack

Rules sind modulare Instruktionsdateien mit Frontmatter. Claude wendet sie automatisch an, wenn die Glob-Pattern auf die aktuelle Datei passen.

## Format

```markdown
---
description: Kurzbeschreibung (was diese Rule tut)
globs: "*.ts,*.tsx,src/api/**"
---

# Rule-Titel

[Instruktionen für Claude]
```

---

## TypeScript / React

```markdown
---
description: TypeScript und React Coding Conventions
globs: "*.ts,*.tsx"
---

# TypeScript Conventions

## Types
- Keine `any` — verwende `unknown` mit Type Guard oder konkreten Typen
- Interfaces für öffentliche APIs, Types für interne Unions/Aliases
- Strict Mode ist aktiv — alle Typen müssen vollständig sein

## Imports
- Absolute Imports, keine relativen `../../` außerhalb von Modul-Grenzen
- Barrel-Exports nur auf Top-Level (`index.ts`)

## React
- Functional Components only
- Props-Interface immer explizit (kein `React.FC<>`)
- Custom Hooks: Prefix `use`, pure functions, keine Side-Effects ohne useEffect
- Kein direktes DOM-Manipulation — nur über Refs

## Fehlerbehandlung
- Kein `try/catch` ohne explizites Error-Logging
- Async-Funktionen immer mit explizitem Return-Type
```

---

## Testing (Jest / Vitest)

```markdown
---
description: Test-Konventionen und -Patterns
globs: "*.test.ts,*.test.tsx,*.spec.ts,*.spec.tsx"
---

# Testing Conventions

## Struktur
- `describe` → Komponente/Funktion
- `it` → spezifisches Verhalten (nicht "should", sondern direkte Aussage)
- AAA-Pattern: Arrange / Act / Assert, durch Leerzeile getrennt

## Was testen
- Happy Path
- Edge Cases (null, undefined, leere Arrays, Grenzwerte)
- Error States
- Kein Testing von Implementierungsdetails — nur öffentliches Verhalten

## Mocks
- Externe Services immer mocken
- Keine globalen Mocks — lokal per `vi.mock()` oder `jest.mock()`
- Mock-Daten in `__fixtures__/` Verzeichnis

## Verboten
- `it.only()` und `describe.only()` nie committen
- `console.log` in Tests
- Timeouts unter 100ms (flaky tests)
```

---

## API-Conventions (REST)

```markdown
---
description: REST API Design Conventions
globs: "src/api/**,src/routes/**,src/controllers/**"
---

# API Conventions

## Routen
- Ressourcen im Plural: `/users`, `/orders`
- Nested für Ownership: `/users/:id/orders`
- Keine Verben in URLs — Verben kommen aus HTTP-Methode

## Response-Format
Immer einheitliches JSON:
```json
{
  "data": { },
  "error": null,
  "meta": { "page": 1, "total": 100 }
}
```

## Status-Codes
- 200: OK
- 201: Created (POST)
- 400: Bad Request (Validation Error)
- 401: Unauthorized
- 403: Forbidden
- 404: Not Found
- 409: Conflict
- 500: Internal Server Error (nie Business Logic Details leaken)

## Validation
- Alle Inputs mit Zod validieren — vor Business Logic
- Validation-Fehler → 400 mit field-level Details

## Controller-Pattern
Route → Controller (Request/Response) → Service (Business Logic) → Repository (DB)
Nie Business Logic in Routes oder Controllers.
```

---

## Python

```markdown
---
description: Python Coding Conventions
globs: "*.py"
---

# Python Conventions

## Style
- PEP 8 (via ruff enforced)
- Type Hints überall — Rückgabetypen explizit
- Docstrings für alle public functions (Google-Style)

## Imports
- Standard Library → Third Party → Local (via isort)
- Keine Wildcard-Imports (`from module import *`)

## Fehlerbehandlung
- Spezifische Exception-Typen (kein blankes `except:`)
- Custom Exceptions von `BaseException` erben
- Logging statt `print()` in Produktion

## Patterns
- Context Manager für Ressourcen (Files, DB-Connections)
- Generators für große Datensätze
- Dataclasses für Value Objects
```

---

## Go

```markdown
---
description: Go Coding Conventions
globs: "*.go"
---

# Go Conventions

## Error Handling
- Errors immer prüfen — nie ignorieren mit `_`
- Errors nach oben propagieren mit Kontext: `fmt.Errorf("context: %w", err)`
- Eigene Error-Types für Domain-Fehler

## Naming
- Kurze, prägnante Namen für lokale Variablen
- Interfaces: Verb-Suffix (`Reader`, `Writer`, `Storer`)
- Exported Names: PascalCase, unexported: camelCase

## Struktur
- Kleine Interfaces (1–3 Methoden)
- Konkrete Dependencies über Interfaces injecten
- Kein globaler State — alles via Dependency Injection

## Concurrency
- Channels für Kommunikation, nicht für Synchronisation
- `sync.Mutex` für geteilten State
- Goroutinen immer mit klarem Lifecycle (context.Context)
```

---

## Security (Universal)

```markdown
---
description: Security-Regeln für alle Dateien
globs: "**"
---

# Security Non-Negotiables

## Secrets
- Niemals Secrets, API-Keys, Tokens, Passwörter im Code
- Environment Variables via `.env` (gitignored) oder Secret Manager
- `.env.example` mit Platzhaltern für Dokumentation (nie echte Werte)

## Input Validation
- Alle externen Inputs validieren bevor Verarbeitung
- SQL: Parameterized Queries / ORM — kein String-Concatenation
- HTML-Output: Escaping / Template-Engine — kein raw HTML-Injection

## Dependencies
- Keine Dependencies mit bekannten CVEs
- Lockfiles committen (`package-lock.json`, `poetry.lock`, `go.sum`)
- Regelmäßige `npm audit` / `pip-audit` / `govulncheck`

## Auth
- Passwörter hashen (bcrypt, argon2 — nie MD5/SHA1)
- JWTs validieren (Signatur + Expiry + Claims)
- Session-IDs nach Login rotieren
```
