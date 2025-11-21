# Hotfix Workflow (Feature/Dev-Logik)

<details><summary>Click to expand..</summary>


# Richtlinie zur Namenskonvention für Hotfix-Branches

## Branch-Typ

- Immer `hotfix` als erster Pfad-Bestandteil  
- Keine zusätzlichen Typen wie `fix`, `feat`, `refactor` davor oder dahinter

### Allgemeines Muster

**Main-Branch:**
```

hotfix/<TICKET-ID>/<optional-kurze-beschreibung>/main

```

**Dev-Branch:**
```

hotfix/<TICKET-ID>/<optional-kurze-beschreibung>/dde

```

**Kurzform ohne Beschreibung:**
```

hotfix/ABC-999/main
hotfix/ABC-999/dde

````

### Commit-Typ vs. Branch-Typ

- Branch-Typ bestimmt Workflow (Feature vs. Hotfix, Basis-Branch, Merge-Reihenfolge).  
- Commit-Typ (fix, feat, refactor, hotfix, …) beschreibt *Art der Änderung* gemäß Conventional Commits.  

**Fazit:**  
Branch-Typ `hotfix` reicht. Commit-Typ übernimmt die fachliche Einordnung.

---

## 1️⃣ Ticket in Jira anlegen

- Ticket erstellen (z. B. `PRIV-155`)  
- Status auf *In Progress* setzen  
- Ticket ggf. in aktuellen Sprint verschieben  

---

## 2️⃣ Hotfix-Branches erstellen

### Ausgangspunkt: master
```bash
git checkout master
git pull origin master
````

### Hotfix-Main-Branch (Single-Commit-Container)

```bash
git checkout -b hotfix/PRIV-155/main
```

**Zweck:**

* Enthält später *genau einen* sauberen Squash-Commit
* Wird in `master` und `develop` gemergt
* Keine Arbeit direkt hier – keine Experimente

### Hotfix-Dev-Branch erstellen

```bash
git checkout hotfix/PRIV-155/main
git checkout -b hotfix/PRIV-155/dde
```

**Zweck `dde`:**

* Arbeitsbranch
* Mehrere Commits, Tests, Refactorings möglich
* Lokale Iteration bis der Hotfix stabil ist

---

## 3️⃣ Hotfix entwickeln (auf `hotfix/PRIV-155/dde`)

### Auf Branch wechseln

```bash
git checkout hotfix/PRIV-155/dde
pnpm install
```

### Problem lösen

```bash
git add .
# git commit -m "fix(PRIV-155): bugfix step 1"
# git commit -m "fix(PRIV-155): bugfix step 2"
# git commit -m "hotfix(PRIV-155): critical bugfix payment processing"
mkcommit
```

### Tests ausführen

```bash
pnpm run test
```

### Version bump (falls kein automatisches Release-Build vorhanden)

* package.json manuell erhöhen

### Pushen

```bash
git push --set-upstream origin hotfix/PRIV-155/dde
```

---

## 4️⃣ Squash-Merge in den Hotfix-Main-Branch

### Wechseln & Squashen

```bash
git checkout hotfix/PRIV-155/main
git merge --squash hotfix/PRIV-155/dde
git commit -m "hotfix(PRIV-155): critical bugfix payment processing"
git push --set-upstream origin hotfix/PRIV-155/main
```

**Ergebnis:**
Ein einziger sauberer Commit im Branch `hotfix/ABC-999/main`.

---

## 5️⃣ Merge Hotfix in `master` (Produktion)

* PR erstellen:
  `hotfix/PRIV-155/main → master`
* **Wichtig:** Hotfix-Branch nicht löschen – wird noch für develop benötigt

---

## 6️⃣ Merge Hotfix in `develop`

* PR erstellen:
  `hotfix/PRIV-155/main → develop`
* Kann eigenständig gemergt werden

---

## Hinweis zu Hotfix-PRs auf develop

Mehrere Commits im Hotfix-PR auf develop deuten oft darauf hin, dass master & develop nicht vollständig synchron waren
(z. B. Release-PR develop → master, aber kein Rück-PR master → develop).

**Technisch unkritisch**, solange nur die tatsächlichen Hotfix-Änderungen im Diff enthalten sind.

Der Merge stellt gleichzeitig sicher, dass develop wieder historisch mit master gleichzieht.

---

# Grundprinzipien

## Naming

```
hotfix/<JIRA-TICKET>/<optional-beschreibung>/main
hotfix/<JIRA-TICKET>/<optional-beschreibung>/dde
```

Keine Prefixes wie `fix/`, `feat/`, `refactor/` bei Hotfix-Branches.

## Main vs. Dev

* **main:**

  * exakt **ein Squash-Commit**
  * Basis für Merge in master & develop

* **dde:**

  * mehrere Commits erlaubt
  * Arbeits- und Experimentierbranch

## Qualität

* Hotfix wird behandelt wie ein normales Feature
* Tests, CI/CD, Reviews, Merges sind Pflicht
* `master` bleibt stabil
* `develop` wird immer nachgezogen





</details>
