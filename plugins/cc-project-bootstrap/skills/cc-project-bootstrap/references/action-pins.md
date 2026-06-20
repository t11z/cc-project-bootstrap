# Action-Pins

Alle GitHub-Actions auf Commit-SHA pinnen, Form: `uses: <owner>/<repo>@<sha> # <tag>`. Der SHA bindet (Supply-Chain), der Kommentar bleibt lesbar und Dependabot-tauglich.

## Beim Bootstrap neu auflösen

Versionen driften. Pro Action die aktuell empfohlene Version ermitteln und auf SHA auflösen:

```bash
git ls-remote https://github.com/<owner>/<repo>.git "refs/tags/<tag>^{}" | awk '{print $1}'
# leeres Ergebnis -> nicht-annotiertes Tag, ohne ^{} erneut:
git ls-remote https://github.com/<owner>/<repo>.git "refs/tags/<tag>"    | awk '{print $1}'
```

Diese Datei danach mit den neuen Werten aktualisieren.

## Zuletzt aufgelöst (Startwerte)

```
actions/checkout@df4cb1c069e1874edd31b4311f1884172cec0e10                       # v6.0.3
actions/setup-node@48b55a011bda9f5d6aeb4c2d9c7362e8dae4041e                      # v6.4.0
actions/setup-python@a309ff8b426b58ec0e2a45f0f869d46889d02405                    # v6.2.0
actions/create-github-app-token@bcd2ba49218906704ab6c1aa796996da409d3eb1         # v3.2.0
actions/github-script@3a2844b7e9c422d3c10d287c895573f7108da1b3                   # v9.0.0
actions/upload-artifact@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a          # v7.0.1
anthropics/claude-code-action@ebcdfe6dc6bb7511eb63e59e07df256dbcf59a2e           # v1.0.145
zircote/structured-madr@1885ace5d825e2b2982505e28468dd4f5ba1d1c0                 # v1.2.0
anthropics/claude-code-security-review@0c6a49f1fa56a1d472575da86a94dbc1edb78eda  # main (kein Release; Commit-Pin)
```

## Runner

GitHub-Default `ubuntu-latest` (aktuell 24.04). Reproduzierbarkeits-Pin `ubuntu-24.04` optional.

## Dependabot

`.github/dependabot.yml` fürs `github-actions`-Ökosystem öffnet PRs, die SHA + Versions-Kommentar gemeinsam bumpen. Das gibt Sicherheit (Pin) und Wartbarkeit (Updates via reviewbarem PR) zugleich.
