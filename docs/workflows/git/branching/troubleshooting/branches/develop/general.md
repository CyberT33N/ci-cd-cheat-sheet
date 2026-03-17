# Forget to merge master back into develop but develop has change since then

### Grundsatz (damit die Reihenfolge klar ist)
- **Hotfix ist bereits in `master` (Production).** Also **muss** dieser Stand **nach `develop` zurück** (Backmerge), damit `develop` nicht “ohne Production-Fix” weiterläuft.
- Ein **MR `develop → master`** ist **ein Release** (du würdest Feature-Commits in Production ziehen) und ist **für Hotfix-Sync falsch**, außer ihr wollt bewusst genau das deployen.

Mehr neue Commits auf `develop` ändern daran nichts: sie machen den Diff nur größer und Konflikte wahrscheinlicher – die Richtung bleibt **`master → develop`** (oder äquivalent: Hotfix-Commit(s) nach `develop`).

---

### Architektonisch korrekte Schritt-für-Schritt-Anleitung (empfohlen: Backmerge mit Konfliktlösung lokal)
**Ziel:** Hotfix aus `master` nach `develop` bringen, ohne `develop`-Commits in Production zu ziehen, und Konflikte **einmal sauber** lösen.

#### 0) Vorbedingungen
- **Working Tree clean** (keine lokalen Änderungen), sonst stashe/committe sie vorher.
- Du brauchst einen **nicht-protected Source-Branch** für den MR (weil man in der Regel nicht direkt auf `master` “Konfliktlösungs-Commits” pushen darf).

#### 1) Lokale Branches aktualisieren
```bash
git stash ; git clean -f -d -x -i -e node_modules
git fetch origin

git checkout develop
git pull origin develop

git checkout master
git pull origin master
```

#### 2) Sync-Branch von `develop` erstellen (Integration-Branch)
```bash
git checkout develop
git checkout -b chore/sync-master-into-develop/<date-or-ticket>
```

#### 3) `master` in diesen Sync-Branch mergen (das ist der Backmerge)
```bash
git merge origin/master
```
- Wenn **keine** Konflikte: du hast einen Merge-Commit, fertig.
- Wenn **Konflikte**: Git stoppt und markiert Dateien als “unmerged”.

#### 4) Konflikte auflösen (entscheidender Teil)
```bash
git status
```
Dann pro konfliktärer Datei:
- Datei öffnen, Konfliktmarker entfernen (`<<<<<<`, `======`, `>>>>>>`)
- **Regel**: `develop`-Stand behalten *plus* Hotfix-Logik aus `master` integrieren (nicht “entweder/oder” blind übernehmen)
- Danach:
```bash
git add <conflicted-file>
```

Wenn alles gelöst ist:
```bash
git status
git commit
```
(Commit ist der Merge-Commit, in dem die Konfliktlösung dokumentiert ist.)

#### 5) Push und MR nach `develop`
```bash
git push -u origin HEAD
```
Dann im GitLab/GitHub:
- **Merge Request:** `chore/sync-master-into-develop/<…>` → `develop`
- **Wichtig:** Für diesen MR **kein Squash**, sondern normal mergen (damit der Sync als Merge nachvollziehbar bleibt).

#### 6) Aufräumen
- MR mergen, Source-Branch optional löschen.
- Ergebnis: `develop` enthält **alle bisherigen `develop`-Commits** *und zusätzlich* den Hotfix aus `master`.

---

### Was, wenn auf `develop` inzwischen viele Commits dazugekommen sind?
- **Gleicher Ablauf.** Du integrierst weiterhin `master` nach `develop`.
- Praktischer Hinweis: Je länger ihr Backmerges aufschiebt, desto häufiger/teurer werden Konflikte. Darum: **Backmerge direkt nach Hotfix-Merge** als festen Schritt.

---

### “Ist `master → develop` nicht falsch? Müsste es nicht andersrum sein?”
- **Nein.** `develop → master` bedeutet “wir releasen das, was in `develop` ist”.
- Beim Hotfix wollt ihr **genau nicht** beliebige neue `develop`-Änderungen in Production ziehen, sondern **nur** den Hotfix in Production *und dann* diesen Fix zurück in die Entwicklung.

---

### Sonderfall (Fallback): Hotfix-Commit(s) cherry-picken statt Backmerge
Nur wenn der Backmerge extrem noisy ist (z. B. weil `master` zusätzliche Release-Merges enthält) und ihr **wirklich nur** den Hotfix übertragen wollt:
- Branch von `develop` erstellen
- Hotfix-Commit(s) aus `master` cherry-picken
- MR → `develop`

Architektonisch bleibt aber die Pflicht bestehen: **Production-Fix muss nach `develop`** – der Backmerge ist der Standardweg, Cherry-pick ist die Ausnahme.
