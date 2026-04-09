Du hast keinen Rebase gemacht, sondern einen **Merge (durch `git pull`)**.
Bei einem Merge mit Konflikten läuft es so:

1. Git stoppt den Merge wegen Konflikten
2. Du löst die Konflikte manuell
3. **Jetzt musst du Git sagen: „Konflikte sind gelöst“ → mit `git add`**
4. Dann den Merge abschließen → mit `git commit`
5. Danach kannst du `git push` machen

---

### Konkrete Schritte

```bash
git add <dateien>
git commit
git push
```

👉 Wenn du alle geänderten Dateien hinzufügen willst:

```bash
git add .
git commit
git push
```

---

### Sonderfall: Automatischer Merge-Commit

Manchmal hat Git schon einen Merge-Commit vorbereitet. Dann reicht oft:

```bash
git add .
git commit
```

(ohne Commit-Message öffnet sich ggf. dein Editor mit einer vorgegebenen Merge-Message)

---

### Merksatz

* **Rebase → `git rebase --continue`**
* **Merge (pull) → `git add` + `git commit`**

---

Wenn du willst, kannst du mir kurz sagen, was `git status` gerade zeigt — dann sage ich dir exakt, was dein nächster Befehl sein sollte.
