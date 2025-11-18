# Ticket > Merge Request (MR) Prozess

<details><summary>Click to expand..</summary>

## 1. Ticket in Jira setzen 📝
- Setze das Ticket auf **`In Progress`** in Jira.
- Verschiebe das Ticket in den aktuellen Sprint.

---

## 2. Feature- und Feature-Dev-Branch erstellen 🛠️
Erstelle einen neuen Feature-Branch und einen zugehörigen Feature-Dev-Branch basierend auf dem `develop`-Branch.

```shell
# Der Befehl mkbranch fragt auch automatisch, ob der aktuelle develop-Branch geladen werden soll
git checkout develop
git pull

# Feature Branch erstellen
# git branch feat/PRIV-001/add-new-button/main
# git checkout feat/PRIV-001/add-new-button/main
mkbranch

# Feature-dev Branch erstellen
# git branch feat/PRIV-001/add-new-button/dde
# git checkout feat/PRIV-001/add-new-button/dde
mkbranch
```

---

## 3. Lokale Umgebung aktualisieren 🌐
Wenn du eine lokale Umgebung wie **Minikube** verwendest, stelle sicher, dass deine Deployments oder Dienste auf dem neuesten Stand sind.

---

## 4. Abhängigkeiten aktualisieren 📦
Falls nötig, aktualisiere die Abhängigkeiten:

```shell
npm ci
```

---

## 5. Ticket lösen und Commit auf Feature-Dev-Branch pushen 🚀
- Stelle sicher, dass alle Tests erfolgreich sind. ✔️
- Hast du alle `.only`-Tests entfernt? 🧹
- Wurden alle ticketbezogenen Änderungen durchgeführt? (z. B. Migrationsskripte) 🔄

---

## 6. Feature-Dev-Branch auf Feature-Branch squashen 🔨
Führe einen **Squash-Merge** des Feature-Dev-Branches in den Feature-Branch durch.

```shell
git checkout feat/PRIV-001/add-new-button/main
git merge --squash feat/PRIV-001/add-new-button/dde

# Wenn Breaking Changes vorhanden sind, nutze eine spezielle Commit-Nachricht!
# git commit -m "fix(ABC-232): Edit custom block"
# git push

mkcommit
```

**Falls Breaking Changes vorliegen, füge einen Footer zur Commit-Nachricht hinzu:**

```shell
git commit -m 'fix(ABC-232): Edit custom block' -m 'BREAKING CHANGE: Route xyz wurde zu abc umbenannt'

# Falls du versehentlich bereits ohne Breaking Change committed hast, kannst du dies nachträglich ändern:
# git add . && git commit --amend -m 'refactor(ABC-2139): MongoDB von Version 5 auf 7 aktualisiert' -m 'BREAKING CHANGE: MongoDB von Version 5 auf 7 aktualisiert'
```

Stelle sicher, dass nur ein Commit im Feature-Branch vorhanden ist. Falls du Änderungen vergessen hast, kannst du dies nachträglich tun:

<details><summary>Click to expand..</summary>

```shell
git add . && git commit --amend --reuse-message HEAD && git push -f
```

</details>




<br><br>


Falls du mehrere Commits im Feature-Branch hast, kannst du sie zurücksetzen:

<details><summary>Click to expand..</summary>

```shell
# Überprüfe mit git log, wie viele der letzten Commits zusammengeführt werden müssen. Alternativ kannst du dies auch im Merge Request (MR) sehen.

# Wechsel zum Ziel-Branch
git checkout your_branch

# Setze die letzten e.g. 4 Commits lokal zurück
# Alternativ kannst du dies auch einzeln mit git reset --soft HEAD^ tun, was sicherer ist.
git reset --soft HEAD~4

# Stelle sicher, dass die betroffenen Commits nicht mehr im Verlauf erscheinen:
git log

# Überprüfe, ob die Dateien der vorherigen Commits immer noch vorhanden sind
git status

git add . && git commit --amend --reuse-message HEAD && git push -f
```

Es gibt Ausnahmen. Zum Beispiel, wenn du an einem Ticket mit Sub-Tasks arbeitest, kannst du mehrere Commits für verschiedene Ticket-IDs erstellen:

- **Branching-Strategie:** Feature-Branches
- **Wichtig:** Falls dein MR aufgrund eines Problems im ersten Commit abgelehnt wird, musst du die Dateien erneut stagen und die Commits neu erstellen.

```shell
# Wenn du bereits committed hast, kannst du es später anpassen:
git add . && git commit --amend -m 'refactor(CCS-2139): MongoDB von Version 5 auf 7 aktualisiert' -m 'BREAKING CHANGE: MongoDB von Version 5 auf 7 aktualisiert'

# Stelle sicher, dass du folgende Schritte machst:
git add . && git commit --amend --reuse-message HEAD && git push -f

# Nutze git log, um zu überprüfen, wie viele der letzten Commits zusammengeführt werden müssen.

# Wechsel zum Ziel-Branch
git checkout your_branch

# Setze z.B. die letzten 4 Commits lokal zurück
git reset --soft HEAD~4

# Überprüfe, ob die betroffenen Commits nicht mehr im Verlauf erscheinen:
git log

# Stelle sicher, dass die Dateien der vorherigen Commits noch vorhanden sind
git status

# Stage die Dateien für den 1. Commit:
git add README.md

# Überprüfe, ob die gewünschten Dateien gestaged sind:
git status

# Force-Push, um die Commit-Historie auf GitLab mit den lokalen Änderungen zu überschreiben
git push -f

# Wiederhole den Prozess, wenn du einen zweiten Commit hast.
```

</details>




---

## 7. `develop`-Branch auf den Feature-Branch rebasen 🔄
Führe einen **Rebase** des `develop`-Branches auf den Feature-Branch durch.

```shell
git checkout develop
git pull

git checkout feat/PRIV-001/add-new-button/main
git rebase develop

# ----- Konflikte lösen ----
# Wenn es Merge-Konflikte gibt und du diese löst, benutze:

# Bei Konflikten mit der package-lock.json:
# rm -f package-lock.json
# npm i

# git add .
# git rebase --continue



### If you get this error when try to git rebase --continue
### hint: Waiting for your editor to close the file... C:\Users\denni\AppData\Local\Programs\cursor\Cursor.exe: line 1: C:UsersdenniAppDataLocalProgramscursorCursor.exe: command not found
### error: there was a problem with the editor 'C:\Users\denni\AppData\Local\Programs\cursor\Cursor.exe'
### Please supply the message using either -m or -F option.

### Option 1 - Use different terminal window

### Option 2 - To get unblocked immediately and continue your rebase, you can tell Git to use a different editor (like Notepad) for this specific operation. In your PowerShell terminal, run:
### ```
### git -c core.editor=notepad rebase --continue
### ```



# --------------------------

# Wenn nach dem Wechseln der Branches untracked Files übrig bleiben:
git clean -f -d -x -i -e node_modules

# Stelle sicher, dass die Unit-Tests und Integrationstests lokal wieder erfolgreich sind.
```

Führe diesen Befehl immer aus! Falls es Änderungen an NPM-Paketen gibt, die auf eine höhere Version aktualisiert wurden, kannst du die aktuelle Version mit folgendem Befehl installieren:

```shell
npm ci
```

---

## 8. Feature-Branch pushen ⬆️
Push deinen Feature-Branch:

```shell
git push --set-upstream origin fix/ABC-232/edit-custom-block/main

# git push --force --set-upstream origin fix/ABC-232/edit-custom-block/main
# --force ist nur nötig, wenn der Branch vor dem Rebase bereits gepusht wurde.
# Beim Rebase wird die Commit-Historie immer überschrieben,
# daher erfordert der nächste Push immer das --force-Flag.
```

---

## 9. Warten bis die GitLab-Pipeline abgeschlossen ist ⏳
Warte, bis die GitLab-Pipeline abgeschlossen ist.

---

## 10. Deployment und Ticketstatus 🚀
- Deploye auf den Test-Cluster und stelle sicher, dass alles funktioniert.
- Setze das Ticket auf **`FINISHED`**, wenn alles erfolgreich ist.

---

## 11. Merge Request (MR) erstellen ➡️
Erstelle den Merge Request im GitLab:  
**Feature-Branch** → **Develop-Branch**


<br><br>


### Änderungen auf dem develop Branch seit Erstellung des PR

<details><summary>Click to expand..</summary>

```shell
git checkout develop
git pull

git checkout feat/PRIV-10/create-evident-abb-v2/main
git rebase develop

# ----- Konflikte lösen ----
# Wenn es Merge-Konflikte gibt und du diese löst, benutze:

# Bei Konflikten mit der package-lock.json:
# rm -f package-lock.json
# npm i

# git add .
# git rebase --continue


### If you get this error when try to git rebase --continue
### hint: Waiting for your editor to close the file... C:\Users\denni\AppData\Local\Programs\cursor\Cursor.exe: line 1: C:UsersdenniAppDataLocalProgramscursorCursor.exe: command not found
### error: there was a problem with the editor 'C:\Users\denni\AppData\Local\Programs\cursor\Cursor.exe'
### Please supply the message using either -m or -F option.

### Option 1 - Use different terminal window

### Option 2 - To get unblocked immediately and continue your rebase, you can tell Git to use a different editor (like Notepad) for this specific operation. In your PowerShell terminal, run:
### ```
### git -c core.editor=notepad rebase --continue
### ```



# --------------------------

# Wenn nach dem Wechseln der Branches untracked Files übrig bleiben:
git clean -f -d -x -i -e node_modules

# Stelle sicher, dass die Unit-Tests und Integrationstests lokal wieder erfolgreich sind.
```

Dann force push:
```
git push --force
```
  
</details>





<br><br>





### 🔄 PR-Workflow mit gesquashten Review-Fixes
- Dieser Guide beschreibt den Workflow, wie du nach einem Review gezielt Änderungen in einem sauberen Commit auf deinen bestehenden **Pull Request Feature Branch** bringst – ohne die Commit-History zu vermüllen und ohne eure Squash-Konvention zu brechen.

<details><summary>Click to expand..</summary>


## 🧠 Ziel
- **Review-Fixes** iterativ auf eigenem Branch durchführen.
- Alle Fixes **squashen zu einem sauberen Commit**.
- Den Commit als **zweiten Commit** auf den ursprünglichen PR-Branch bringen.
- Kein Force-Push notwendig. PR bleibt offen und sauber.

---

## 🔧 Ausgangssituation

| Branch                                  | Zweck                        |
|----------------------------------------|------------------------------|
| `feat/PRIV-10/create-evident-abb-v2/main`         | Ursprünglicher PR-Branch     |
| `feat/PRIV-10/create-evident-abb-v2-pr-changes/dde` | Neuer Dev-Branch für Fixes   |

---

## ✅ Step-by-Step Guide

### 1. Wechsle auf deinen Review-Fix-Branch
```bash
git checkout feat/PRIV-10/create-evident-abb-v2-pr-changes/dde
```

### 2. Squashe alle Commits zu einem einzelnen Fix-Commit
```bash
git reset --soft origin/feat/PRIV-10/create-evident-abb-v2/main
git commit -m "fix(PRIV-10): Review-Fixes & Ergänzungen nach Feedback"
```

> 🧠 **Erklärung:**  
> Du befindest dich hier **auf dem Fix-Branch**, der 10–20 Commits enthalten kann.  
> `git reset --soft` setzt deinen HEAD auf den Stand des PR-Branches – aber **behält alle Änderungen gestaged**.  
> Dann erzeugst du **einen neuen, einzigen Commit**, der alle Fixes zusammenfasst.  
> Ergebnis: ein sauberer, gesquashter Fix-Commit auf dem Fix-Branch.

- Normalerweise wird es hier **NIEMALS** Probleme geben. Es gibt aber **Edgecase-Szenarien**, wenn man zum Beispiel von einem **Pull-Request-Branch** abzweigen **MUSS**, weil dieser noch nicht angenommen werden kann, um weiterzuarbeiten. Und dann Später wird dieser **Branch** modifiziert durch einen **Amend-Push** zum Beispiel. In diesem Szenario **MUSST** du folgendem **Dokument** folgen:
  - docs\workflows\git\troubleshooting\merges\squash-merges\feature-dev-into-feature-branch.md


### 3. Wechsle zurück auf den PR-Branch
```bash
git checkout feat/PRIV-10/create-evident-abb-v2/main
```

### 4. Mergest den neuen Commit rein – ohne Squash oder Fast-Forward
```bash
git merge --no-ff feat/PRIV-10/create-evident-abb-v2-pr-changes/dde -m "chore(PRIV-10): Merge Review-Fixes from dde branch"
```

> 🔍 Alternativ: Wenn du den Commit einfach nur übernehmen willst:
> ```bash
> git cherry-pick feat/PRIV-10/create-evident-abb-v2-pr-changes/dde
> ```

### 5. Push zurück zum Remote-PR-Branch
```bash
git push origin feat/PRIV-10/create-evident-abb-v2/main
```

---

## 🧼 Ergebnis

- Dein ursprünglicher PR-Branch enthält:
  1. ✅ Den ersten Commit aus der ursprünglichen Arbeit
  2. 🧼 Einen sauberen Fix-Commit mit allen Änderungen aus dem Review

- Die Review-Historie bleibt nachvollziehbar.
- Kein Force-Push nötig.
- Git-Log bleibt klar und durchdacht.
- Reviewer sieht: Was war, was wurde gefixt.

  
</details>







---

## 12. Postman-Collection aktualisieren 📬
Falls nötig, aktualisiere die Postman-Collection.




</details>
