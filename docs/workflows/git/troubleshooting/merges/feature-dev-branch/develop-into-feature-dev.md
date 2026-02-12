# Develop-Into-Feature-Dev

<details><summary>Click to expand..</summary>

## 🧠 Ziel
Du willst den **aktuellen Stand von `develop`** in deinen **Feature-Dev-Branch** holen – z. B. um **temporäre Releases/Test-Deployments** zu erstellen.

➡️ Dafür reicht ein **normaler Merge** von `develop` in deinen Feature-Dev-Branch.

---

## ✅ Warum Merge (und nicht Rebase)

### Merge (empfohlen für diesen Zweck)
- Erstellt **einen Merge-Commit**.
- Konflikte müssen (falls vorhanden) **einmal pro Datei** gelöst werden.
- **Kein History-Rewrite** → geeignet, wenn der Branch gepusht/geteilt/deployed wird.

### Rebase (hier nicht der Standard)
- Spielt **alle** Feature-Dev-Commits auf eine neue Basis.
- Bei langen Branches sehr aufwändig und konfliktanfällig.
- Erfordert nach Push in der Regel einen **Force-Push** (History wird überschrieben).

---

## 🔒 Guards / Voraussetzungen
- Working Tree ist clean:

```bash
git status
```

- Du arbeitest auf dem **Feature-Dev-Branch** (nicht auf `develop`).
- Du holst dir den Stand von **Remote** (nicht nur lokale, ggf. veraltete Branches).

---

## 🧭 Step-by-Step: `develop` in Feature-Dev mergen

### 1) Remote-Refs aktualisieren

```bash
git fetch origin --prune
```

### 2) Auf den Feature-Dev-Branch wechseln und lokal fast-forwarden

```bash
git checkout <feature-dev-branch>

git merge --ff-only origin/<feature-dev-branch>
```

### 3) `develop` in den Feature-Dev-Branch mergen

```bash
git merge origin/develop
```

- **Ohne Konflikte**: Merge ist direkt abgeschlossen (Merge-Commit wird erzeugt).


- **Mit Konflikten**: Konflikte lösen und dann den Merge abschließen:

```bash
# Bei Konflikten mit der package-lock.json:
# Remove-Item -Path "pnpm-lock.yaml" -Force
# pnpm i
# git add pnpm-lock.yaml

git add -A

git commit
```

### 4) Optional: lokal verifizieren (Build/Tests)
Beispiele:

```bash
npm ci
npm test
```

### 5) Feature-Dev-Branch pushen

```bash
git push
```

---

## 🔍 Safety Checks
- Prüfe, dass du wirklich den `develop`-Stand drin hast:

```bash
git log --oneline --decorate -n 20
```

- Prüfe, ob noch Konflikte offen sind:

```bash
git diff --name-only --diff-filter=U
```

---

## ♻️ Rollback / Recovery (Enterprise-safe)
Wenn du den Merge bereits gepusht hast und ihn zurücknehmen musst, nutze **Revert** (kein History-Rewrite):

```bash
git revert -m 1 <merge-commit-sha>
```

---

## 🧩 Hinweis für späteren Squash-/MR-Flow
Ein Merge von `develop` in Feature-Dev ist grundsätzlich okay.

Wichtig ist nur: Wenn du später den Feature-Dev-Branch per **`git merge --squash`** in den **Feature-Branch** bringst, sollte der **Feature-Branch** vorher ebenfalls auf einem aktuellen `develop`-Stand sein – sonst könntest du `develop`-Änderungen ungewollt mit in deinen Squash-Commit aufnehmen.

</details>
