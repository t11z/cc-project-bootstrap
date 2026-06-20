# Hooks Cookbook

## Event-Übersicht

| Event | Timing | Kann blockieren? | Typischer Use-Case |
|-------|--------|-----------------|-------------------|
| `PreToolUse` | Vor Tool-Aufruf | ✅ Ja | Validation, Guards |
| `PostToolUse` | Nach Tool-Aufruf | ❌ Nein | Linting, Logging, Formatting |
| `UserPromptSubmit` | Bei Nutzereingabe | ✅ Ja | Input-Sanitization, Logging |
| `Stop` | Claude ist fertig | ✅ Ja | Vollständigkeitsprüfung |
| `SubagentStop` | Subagent fertig | ✅ Ja | Subagent-Output-Validation |
| `SessionStart` | Session-Beginn | ❌ Nein | Setup, Kontext-Laden |
| `SessionEnd` | Session-Ende | ❌ Nein | Cleanup, Reporting |
| `Notification` | Claude-Notification | ❌ Nein | Alerting |

---

## Output-Protokoll (stdout von Hook-Script → Claude)

```json
// Blockieren (nur PreToolUse / UserPromptSubmit / Stop)
{ "block": true, "message": "Grund für Blockierung" }

// Non-blocking Feedback
{ "feedback": "Info-Text für Claude" }

// Output unterdrücken (bei verbose scripts)
{ "suppressOutput": true }

// Kombination
{ "feedback": "Lint-Warnings gefunden", "suppressOutput": false }
```

Exit-Code-Verhalten:
- `exit 0` → Hook erfolgreich
- `exit 2` + JSON auf stderr → Blockierung mit Message
- `exit 1` → Fehler (wird Claude mitgeteilt)

---

## Konfiguration in settings.json

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write|Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/pre-tool.sh",
            "timeout": 10
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/post-edit.sh"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/validate-completion.sh"
          }
        ]
      }
    ]
  }
}
```

**Matcher-Syntax:** Regex gegen Tool-Namen. `Edit|Write` matcht Edit und Write Tools.

---

## Hook-Bibliothek

### Branch-Guard (PreToolUse: Edit|Write)
Blockiert direkte Edits auf main/master.

```bash
#!/bin/bash
BRANCH=$(git branch --show-current 2>/dev/null)
if [[ "$BRANCH" == "main" || "$BRANCH" == "master" ]]; then
  echo '{"block": true, "message": "Direct edits on main are not allowed. Create a feature branch first: git checkout -b feature/your-feature"}' >&2
  exit 2
fi
```

---

### Secrets-Guard (PreToolUse: Write|Edit)
Erkennt potenzielle Secrets im zu schreibenden Content.

```bash
#!/bin/bash
INPUT=$(cat)
CONTENT=$(echo "$INPUT" | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('content',''))" 2>/dev/null || echo "")

PATTERNS="(sk-[a-zA-Z0-9]{20,}|api[_-]?key|password\s*=|secret\s*=|token\s*=|-----BEGIN (RSA |EC )?PRIVATE KEY)"

if echo "$CONTENT" | grep -qiE "$PATTERNS"; then
  echo '{"block": true, "message": "Potential secret or credential detected. Use environment variables instead."}' >&2
  exit 2
fi
```

---

### Post-Edit Linter (PostToolUse: Edit|Write)
Führt Linter automatisch nach jedem Edit aus.

```bash
#!/bin/bash
# Dateiname aus Tool-Input extrahieren
FILE=$(cat | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('file_path',''))" 2>/dev/null || echo "")

if [ -z "$FILE" ]; then exit 0; fi

case "$FILE" in
  *.ts|*.tsx)
    npx eslint "$FILE" --fix --quiet 2>/dev/null
    ;;
  *.py)
    ruff check "$FILE" --fix --quiet 2>/dev/null
    ;;
  *.go)
    gofmt -w "$FILE" 2>/dev/null
    ;;
esac

exit 0
```

---

### TypeScript Type-Check (PostToolUse: Edit|Write)
Prüft TypeScript-Kompilierung nach TS-File-Edits.

```bash
#!/bin/bash
FILE=$(cat | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('file_path',''))" 2>/dev/null || echo "")

if [[ "$FILE" != *.ts && "$FILE" != *.tsx ]]; then exit 0; fi

RESULT=$(npx tsc --noEmit 2>&1)
if [ $? -ne 0 ]; then
  echo "{\"feedback\": \"TypeScript errors found:\\n$RESULT\"}"
fi
```

---

### Test-Runner nach Edit (PostToolUse: Edit|Write)
Führt relevante Tests nach Code-Änderungen aus.

```bash
#!/bin/bash
FILE=$(cat | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('file_path',''))" 2>/dev/null || echo "")

# Nur für Source-Files, nicht für Tests selbst oder Config
if [[ "$FILE" == *.test.* || "$FILE" == *.spec.* || "$FILE" == *.config.* ]]; then
  exit 0
fi

if [[ "$FILE" == src/* ]]; then
  # Test-File-Pattern ableiten
  TEST_FILE="${FILE/src\//tests/}"
  TEST_FILE="${TEST_FILE%.ts}.test.ts"
  
  if [ -f "$TEST_FILE" ]; then
    npx jest "$TEST_FILE" --passWithNoTests --silent 2>&1 | tail -5
  fi
fi
```

---

### Completion-Validator (Stop)
Stellt sicher, dass Claude alle TODOs adressiert hat.

```bash
#!/bin/bash
# Prüft ob offene TODOs im aktuellen Diff sind
TODOS=$(git diff --cached 2>/dev/null | grep -c "TODO\|FIXME\|HACK" || echo "0")

if [ "$TODOS" -gt 0 ]; then
  echo "{\"feedback\": \"Warning: $TODOS TODO/FIXME comments in staged changes. Consider resolving before committing.\"}"
fi
```

---

### Session-Logger (SessionEnd)
Loggt Session-Aktivität für Audit-Trail.

```bash
#!/bin/bash
TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
LOG_FILE=".claude/session-log.jsonl"

echo "{\"timestamp\": \"$TIMESTAMP\", \"branch\": \"$BRANCH\", \"event\": \"SessionEnd\"}" >> "$LOG_FILE"
```

---

### Dangerous-Command-Guard (PreToolUse: Bash)
Blockiert destruktive Bash-Befehle unabhängig von Permissions.

```bash
#!/bin/bash
COMMAND=$(cat | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('command',''))" 2>/dev/null || echo "")

DANGEROUS_PATTERNS=(
  "rm -rf /"
  "rm -rf \*"
  "dd if=/dev/zero"
  "mkfs\."
  ":(){ :|:& };:"  # Fork bomb
  "chmod -R 777 /"
)

for PATTERN in "${DANGEROUS_PATTERNS[@]}"; do
  if echo "$COMMAND" | grep -qE "$PATTERN"; then
    echo "{\"block\": true, \"message\": \"Dangerous command pattern detected: $PATTERN\"}" >&2
    exit 2
  fi
done
```

---

## Anti-Patterns

**Hooks-Loop vermeiden:** Hooks dürfen keine Aktionen auslösen, die denselben Hook erneut triggern. Claude Code hat einen `stop_hook_active` Guard, aber trotzdem aufpassen.

**Timeout-Grenzen:** Standard-Timeout ist 60s. Für schnelle Guards (branch check, secrets scan) unter 5s bleiben. Langläufer (full test suite) in PostToolUse, nie in PreToolUse.

**Kein State zwischen Hooks:** Hooks sind zustandslos — kein Shared Memory zwischen Aufrufen. Für Audit-Trails → in Datei schreiben (SessionEnd).
