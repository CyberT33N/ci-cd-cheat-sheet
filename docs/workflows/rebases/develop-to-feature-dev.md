# Develop -> Feature-dev Branch

<details><summary>Click to expand..</summary>

- Man **MUSS** auf jeden Fall hier kein Rebase vom Developbranch durchführen, weil wenn man auf seinem Feature-Devbranch hunderte von Comets hat, dann **MUSS** jeder einzelne Comet, den man hat, rebased werden mit dem Developbranch.


## 🔹 Dein Problem

* Dein Feature-Branch hat viele Commits (156).
* Develop hat zwischenzeitlich 21 Commits bekommen.
* Rebase auf Develop → Git will **alle 156 Commits einzeln auf die neue Basis spielen** → dauert ewig, viele Konflikte.
* Dein Ziel: Datei gelöscht behalten, PR sauber machen, später Squash-Merge.

---

## 🔹 Deine Alternativen

### 1️⃣ Rebase wie bisher

* **Vorteil:** Commit-Historie bleibt linear.
* **Nachteil:** Lange, mühsam, viele Konflikte bei langen Branches.
* **Fazit:** Sinnvoll, wenn du saubere Linearität willst, aber bei 156 Commits auf 21 neue sehr zäh.

---

### 2️⃣ Merge Develop in Feature-dev-Branch

```bash
git checkout feat/PRIV-001/add-new-button/dde
git merge develop
```
- `develop` ist incoming und `feat/PRIV-001/add-new-button/dde` is current

* Git erstellt **einen Merge-Commit** statt jeden Commit neu zu spielen.
* Konflikte müssen trotzdem gelöst werden, **aber nur einmal pro Datei**, nicht für jeden Commit.
* Danach: Pushen, Squash-Merge auf Feature-Branch → PR → Develop: nur **ein Commit**.

**Wichtig:**

* Für den PR später ist es egal, ob du vorher einen Merge statt Rebase gemacht hast, **weil du sowieso einen Squash-Merge machst**.
* Die Merge-Historie auf deinem Feature-Branch spielt keine Rolle, **der PR wird sauber auf einen Commit reduziert**.


## 🔹 Schritt 1: Konflikte lösen

1. Öffne alle Dateien mit Konflikten.
2. Entscheide, welche Änderungen übernommen werden (Feature oder Develop oder Mix).
3. Entferne die Git-Marker (`<<<<<<<`, `=======`, `>>>>>>>`).

---

## 🔹 Schritt 2: Änderungen stagen (**MUSS**)

```bash
git add .
```

* Das markiert alle Konflikte als **gelöst**.
* Ohne `git add` weiß Git nicht, dass du die Konflikte gelöst hast.

---

## 🔹 Schritt 3: Merge abschließen (**MUSS**)

```bash
git commit

# cursor if there are problems
# git -c core.editor=notepad commit
```

* Git öffnet standardmäßig den Merge-Commit-Editor.
* Du kannst die vorgeschlagene Nachricht übernehmen, z. B. `Merge branch 'develop' into feat/...`.
* Das erzeugt **einen Merge-Commit**, der alle Änderungen von Develop integriert.

---

## 🔹 Optional / empfohlen

* Prüfe, dass alles kompiliert / getestet ist:

```bash
npm ci
npm test   # oder test:all
```

* Dann kannst du deinen Feature-Branch pushen:

```bash
git push --set-upstream origin feat/PRIV-001/add-new-button/main
```

* **Squash-Merge später auf PR:**

  * Dein Feature-Branch enthält jetzt Merge-Commit + deine Änderungen.
  * Wenn du PR machst, wähle **Squash-Merge**, dann wird die gesamte Historie auf **einen Commit** reduziert → sauber in Develop.


</details>
