# Hotfix Workflow (Feature/Dev-Logik)

<details><summary>Click to expand..</summary>



### 1️⃣ Hotfix-Feature-Branch erstellen

* Ausgangspunkt: **Master**

```bash
git checkout master
git pull origin master
git checkout -b hotfix/ABC-999/main
```

* **Zweck:** sauberer Commit-Container für den späteren Squash.
* Kein Arbeiten direkt hier, nur Branch als „Single-Commit-Ziel" vorbereiten.

---

### 2️⃣ Hotfix-Dev-Branch erstellen

* Basis: **Hotfix-Feature-Branch**

```bash
git checkout hotfix/ABC-999/main
git checkout -b hotfix/ABC-999/dde
```

* **Zweck:** Spielwiese für mehrere Commits, Tests, Experimente.
* **Multiple Commits erlaubt** → lokale Iterationen, Refactoring, Tests.

---

### 3️⃣ Hotfix entwickeln

* Bearbeitung auf `hotfix/ABC-999/dde`
* Lokale Tests laufen lassen:

```bash
npm ci
npm test
```

* Mehrere Commits wie üblich:

```bash
git add .
git commit -m "fix(ABC-999): bugfix step 1"
git commit -m "fix(ABC-999): bugfix step 2"
```

---

### 4️⃣ Squash Merge in Hotfix-Feature-Branch

* Wechsel zum Feature-Branch:

```bash
git checkout hotfix/ABC-999/main
git merge --squash hotfix/ABC-999/dde
git commit -m "hotfix(ABC-999): critical bugfix payment processing"
```

* Ergebnis: **ein sauberer Commit** im Feature-Branch.

---

### 5️⃣ Merge Hotfix in Master

```bash
git checkout master
git merge --no-ff hotfix/ABC-999/main
git push origin master
```

* CI/CD baut den Hotfix und deployed in Produktion.

---

### 6️⃣ Merge Hotfix in Develop

* Develop up-to-date halten:

```bash
git checkout develop
git pull origin develop
git merge --no-ff hotfix/ABC-999/main
git push origin develop
```

* Konflikte lösen falls nötig
* Tests erneut ausführen.

---

### 7️⃣ Branch Cleanup (optional)

```bash
git branch -d hotfix/ABC-999/dde
git branch -d hotfix/ABC-999/main
git push origin --delete hotfix/ABC-999/dde
git push origin --delete hotfix/ABC-999/main
```

* Branches existieren nur temporär, um CI/CD und Historie sauber zu halten.

---

### ✅ Prinzipien

1. **Feature-Branch zuerst**, Dev-Branch danach → sauberer Squash möglich
2. **Multiple Commits nur im Dev-Branch** → Master bleibt sauber
3. **Master = stabile Basis**, Develop bekommt Hotfix **nach Master**
4. **CI/CD-Checks für Hotfix verpflichtend** → Qualitätssicherung



</details>
