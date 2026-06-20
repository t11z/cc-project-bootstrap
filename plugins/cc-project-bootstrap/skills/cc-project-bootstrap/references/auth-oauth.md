# Auth-Modell — OAuth-only

Regel: nie `ANTHROPIC_API_KEY`, immer `CLAUDE_CODE_OAUTH_TOKEN`. Token wird lokal mit `claude setup-token` erzeugt und als Repo-Secret `CLAUDE_CODE_OAUTH_TOKEN` hinterlegt.

## Wer nimmt den Token direkt

`anthropics/claude-code-action` akzeptiert `claude_code_oauth_token` als first-class Alternative zu `anthropic_api_key`. Der Issue-Triage-Bot läuft damit ohne Umbau. Das ist die gehärtete Action (Actor-Permission-Checks, Secret-Scrubbing) — die richtige für untrusted Input wie Issues und Fork-PRs.

## Warum der Security-Review einen Umbau braucht

`anthropics/claude-code-security-review` ist als Composite-Action darauf ausgelegt, gegen die API zu authentifizieren. Zwei Stellen im Quelltext stehen dem OAuth-only-Betrieb im Weg:

**1 — Pre-Flight-Guard (`claudecode/github_action_audit.py`, `validate_claude_available()`):**
Prüft `claude --version` *und* `os.environ.get("ANTHROPIC_API_KEY")`. Fehlt der Key, bricht der Lauf ab, bevor die CLI startet — obwohl die CLI selbst sauber über `CLAUDE_CODE_OAUTH_TOKEN` authentifizieren würde. Der Guard prüft nur *Präsenz*, er *benutzt* den Key nicht für den CLI-Call.

**2 — FP-Filtering (`claudecode/claude_api_client.py`):**
Instanziiert das `anthropic`-SDK mit `Anthropic(api_key=ANTHROPIC_API_KEY)`. Das SDK sendet den Key als `x-api-key`-Header. Ein OAuth-Token (`sk-ant-oat…`) ist ein Bearer-Scheme-Token und als `x-api-key` ungültig. Der Gate `if use_claude_filtering and api_key:` fällt aber graceful auf die deterministischen Hard-Rules zurück, wenn der Key fehlt — kein Crash.

## Der Umbau

Den Composite nicht als `uses:` aufrufen, sondern seine Scan-Steps im eigenen Workflow nachbauen:

1. Security-Review-Repo auf gepinntem SHA in einen Unterordner klonen/auschecken.
2. `pip install -r .../claudecode/requirements.txt`; `npm install -g @anthropic-ai/claude-code`.
3. **Guard patchen** — den Presence-Check so erweitern, dass er `CLAUDE_CODE_OAUTH_TOKEN` als gültige Auth akzeptiert. Der Patch matcht den exakten Block über die eindeutige Fehlermeldungs-Zeile und setzt einen Selbsttest: matcht der Block nicht mehr (Upstream-Drift), schlägt der Step fehl, statt still falsch zu laufen.
4. Lauf mit `CLAUDE_CODE_OAUTH_TOKEN` (echte Auth via CLI), `ENABLE_CLAUDE_FILTERING=false`, ohne `ANTHROPIC_API_KEY`. Deterministisches Filtering bleibt aktiv.

Konsequenz: voller semantischer Audit unter OAuth, ohne KI-FP-Filter. Das ist der bewusste Trade-off.

## Lokal vs. CI

Der lokale Slash-Command `/security-review` (`.claude/commands/security-review.md`, Anthropics Original) ist **auth-agnostisch**: er läuft in der bereits authentifizierten Claude-Code-Session und braucht keinen Key. Nur die CI-Action braucht den Umbau. Derselbe Audit also zweigleisig, beide ohne `ANTHROPIC_API_KEY`.

## GitHub-Token für den Triage-Bot

Default-`GITHUB_TOKEN` kann keine Folge-Workflows triggern und committet als `github-actions[bot]`. Für echte Bot-Identität und funktionierende CI auf erzeugten PRs: GitHub-App-Token via `actions/create-github-app-token` (Secrets `APP_ID`, `APP_PRIVATE_KEY`). Achtung: mit App-Token greift der eingebaute Actor-Permission-Check der claude-code-action nicht — der Implement-Pfad braucht daher einen **eigenen** Permission-Check-Step (Comment-Author muss Write+ haben).

## Token-Lebensdauer

`setup-token`-Tokens sind CI-tauglich, aber endlich. Bei Auth-Fehlern in CI: Token neu erzeugen, Secret aktualisieren. (Kein automatisches Refresh-Issue — bewusst weggelassen.)
