# Develop-Into-Feature-Dev


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

Wenn bei dir **incoming = `develop`** ist und du für z.b. **`.swarm/memory.db`** die `develop`-Version übernehmen willst:

```bash
git checkout --theirs -- .swarm/memory.db
git add .swarm/memory.db
```

Falls du stattdessen **deine** Version behalten willst:

```bash
git checkout --ours -- .swarm/memory.db
git add .swarm/memory.db
```



Falls du oder KI in einer Datei etwas ändern und du sie dann als Quelle für den Merge Konflikt nutzen willst:
```shell
git add -- "apps/test/package.json"
```



Falls auf dem develop Branch eine Datei gelöscht wurde, also zum Beispiel in Incoming, und man das bestätigen will, kann man das hier machen.
```shell
git rm -- src\modules\pvs\charly\contracts\CharlyPatient.ts
```


Bei Konflikten mit der package-lock.json:


Method 1 (recommended)
```shell
nvm use 26.2.0

pnpm install --lockfile-only

git add pnpm-lock.yaml

git merge --continue
```
Warum so und nicht löschen?
pnpm install --lockfile-only ist hier der sauberste Weg, weil es:

die Lockdatei aus dem jetzt korrekten package.json neu ableitet
kein unnötiges node_modules-Refresh erzwingt


Method 2 (not recommended)
```shell
Remove-Item -Path "pnpm-lock.yaml" -Force

pnpm i

git add pnpm-lock.yaml
```







---


Add changes
```bash
git add -A

git commit
```











<br><br>



`git checkout --theirs -- <datei>` funktioniert nur, wenn auf der Merge-Seite `theirs` **eine echte Dateiversion existiert**.

Wurde die Datei im Branch, den du gerade mergst, **gelöscht**, dann gibt es dort **keine `their`-Version** mehr. Deshalb kommt der Fehler:

```text
does not have their version
```

Für dein Cheat Sheet:
- `theirs` vorhanden: `git checkout --theirs -- <datei>`
- `theirs` = Datei wurde gelöscht: `git rm -- <datei>`

Merksatz:
Wenn `theirs` die Datei gelöscht hat, kannst du sie nicht auschecken, sondern musst die Löschung übernehmen.











<br><br>

<br><br>


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


