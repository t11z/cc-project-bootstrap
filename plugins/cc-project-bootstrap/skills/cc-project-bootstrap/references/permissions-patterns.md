# Permissions-Patterns nach Stack

## Grundstruktur

```json
{
  "permissions": {
    "allow": [ ... ],
    "deny": [ ... ]
  }
}
```

**Logik:** Deny hat Vorrang. Definiere zuerst was nie erlaubt ist, dann öffne gezielt.

---

## Universal-Deny (immer empfohlen)

```json
"deny": [
  "Read(.env)",
  "Read(.env.*)",
  "Read(.env.local)",
  "Read(**/secrets/**)",
  "Read(**/.ssh/**)",
  "Read(**/private-keys/**)",
  "Bash(rm -rf *)",
  "Bash(sudo *)",
  "Bash(curl * | bash)",
  "Bash(wget * | sh)"
]
```

---

## Node.js / TypeScript Projekt

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(npx *)",
      "Bash(node *)",
      "Bash(git *)",
      "Read(**)",
      "Edit(src/**)",
      "Edit(tests/**)",
      "Edit(*.json)",
      "Edit(*.md)"
    ],
    "deny": [
      "Read(.env)",
      "Read(.env.*)",
      "Edit(node_modules/**)",
      "Bash(npm publish)",
      "Bash(rm -rf *)"
    ]
  }
}
```

---

## Python Projekt

```json
{
  "permissions": {
    "allow": [
      "Bash(python *)",
      "Bash(pip install *)",
      "Bash(pytest *)",
      "Bash(ruff *)",
      "Bash(mypy *)",
      "Bash(git *)",
      "Read(**)",
      "Edit(src/**)",
      "Edit(tests/**)"
    ],
    "deny": [
      "Read(.env)",
      "Read(**/secrets/**)",
      "Bash(pip uninstall *)",
      "Bash(rm -rf *)"
    ]
  }
}
```

---

## Infrastructure / Terraform

```json
{
  "permissions": {
    "allow": [
      "Bash(terraform fmt)",
      "Bash(terraform validate)",
      "Bash(terraform plan)",
      "Bash(git *)",
      "Read(**)",
      "Edit(*.tf)",
      "Edit(*.tfvars.example)"
    ],
    "deny": [
      "Bash(terraform apply)",
      "Bash(terraform destroy)",
      "Read(*.tfvars)",
      "Read(**/credentials/**)",
      "Read(**/.aws/**)"
    ]
  }
}
```

> terraform apply und destroy explizit blockieren — nur manuell nach Review.

---

## Go Projekt

```json
{
  "permissions": {
    "allow": [
      "Bash(go build *)",
      "Bash(go test *)",
      "Bash(go fmt *)",
      "Bash(go vet *)",
      "Bash(golangci-lint *)",
      "Bash(git *)",
      "Read(**)",
      "Edit(*.go)",
      "Edit(go.mod)",
      "Edit(go.sum)"
    ],
    "deny": [
      "Read(.env)",
      "Read(**/secrets/**)",
      "Bash(rm -rf *)"
    ]
  }
}
```

---

## Monorepo (mit Workspaces)

```json
{
  "permissions": {
    "allow": [
      "Bash(pnpm *)",
      "Bash(turbo *)",
      "Bash(git *)",
      "Read(**)",
      "Edit(packages/**)",
      "Edit(apps/**)"
    ],
    "deny": [
      "Read(.env*)",
      "Edit(dist/**)",
      "Edit(build/**)",
      "Edit(.turbo/**)",
      "Bash(rm -rf node_modules)"
    ]
  }
}
```

---

## Read-Only Audit (z.B. Security-Review eines fremden Repos)

```json
{
  "permissions": {
    "allow": [
      "Read(**)",
      "Bash(grep *)",
      "Bash(find *)",
      "Bash(cat *)"
    ],
    "deny": [
      "Edit(**)",
      "Write(**)",
      "Bash(git commit *)",
      "Bash(git push *)",
      "Bash(npm *)",
      "Bash(pip *)"
    ]
  }
}
```

---

## Glob-Syntax Referenz

| Pattern | Bedeutung |
|---------|-----------|
| `*` | Ein Pfad-Segment (keine `/`) |
| `**` | Beliebige Tiefe inkl. Unterverzeichnisse |
| `*.ts` | Alle .ts-Dateien im Root |
| `**/*.ts` | Alle .ts-Dateien in allen Verzeichnissen |
| `src/**` | Alles unter src/ |
| `Bash(npm:*)` | Alle Bash-Calls die mit `npm` beginnen |
