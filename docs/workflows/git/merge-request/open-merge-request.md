```markdown
## Umgang mit offenen Merge Requests (Feature-/Feature-Dev-Branches)

### Standard-Szenario: Mehrere offene MRs auf `develop`

- **Ausgangspunkt**: Alle Feature-Branches werden **immer von `develop`** abgezweigt.
- Pro Ticket:
  - `feat/XYZ/main` (Feature-Main, enthält am Ende genau **einen** Squash-Commit),
  - `feat/XYZ/dde` (Feature-Dev, alle Zwischen-Commits nur hier).
- Workflow:
  1. Ticket-Branch von `develop` abzweigen (`feat/XYZ/main`).
  2. Dev-Branch von `feat/XYZ/main` abzweigen (`feat/XYZ/dde`).
  3. Ticket auf `dde` implementieren, testen, squash-merge mit `git merge --squash` nach `main`.
  4. MR von `feat/XYZ/main` → `develop` erstellen.

- **Mehrere offene MRs**:
  - Plattformen wie GitHub/GitLab rechnen immer gegen den **Ziel-Branch** (z.B. `develop`).
  - Wenn ein anderer MR zuerst gemerged wird, zeigt die Plattform die Diff automatisch **relativ zum aktualisierten `develop`**.
  - In den meisten Fällen:
    - ist nur ein „visueller Rebase“ nötig (MR diff passt sich an),
    - entstehen **keine** oder sehr wenige Konflikte, weil jeder Branch sauber von `develop` abgezweigt wurde und Feature-Main nur den einen Squash-Commit enthält.

---

## Spezialfall: Von einem offenen PR-Branch abzweigen (Feature-on-Feature)

### Motivation

- Feature B baut **fachlich direkt** auf Feature A auf.
- MR für Feature A ist noch **offen**, aber du **MUSST** weiterarbeiten.
- Statt auf `develop` abzweigen (und Arbeit duplizieren) wird ein neuer Ticket-Branch **auf dem PR-Branch** von Feature A erstellt.

Beispiel:

```bash
# A: vorhandener PR-Branch
git checkout feat/PRIV-100/feature-a/main

# B: neues Ticket, das auf A aufsetzt
git checkout -b feat/PRIV-143/my-new-feature/main
git checkout -b feat/PRIV-143/my-new-feature/dde
```

### Wichtig: Ab diesem Zeitpunkt gelten strengere Regeln

- Dieses Vorgehen ist **VALIDE**, aber es wird zum **„Stacked Branch“**:
  - Feature B basiert **nicht** auf `develop`, sondern auf dem **aktuellen Commit** von Feature A.
- **Konvention ab dem Zeitpunkt des Abzweigens**:
  - Auf dem **PR-Branch (Feature A)** sind **keine History-Rewrites mehr erlaubt**, d.h.:
    - **Kein** `git commit --amend` mehr.
    - **Kein** `git rebase` + `git push --force`, das die Commit-ID verändert.
  - Hintergrund:
    - `--amend` und Rebase erstellen **NEUE Commit-IDs**, auch wenn der Inhalt „gleich“ bleibt.
    - Dein Feature-Dev-Branch für B (`feat/PRIV-143/.../dde`) hängt dann auf der **alten Historie**,  
      der PR-Branch auf der **neuen Historie**.
    - Späterer `merge --squash` von B nach B-main führt dann zu massiven „Merge Changes“ (both added / both modified / both deleted), obwohl fachlich „nur“ aufeinander aufbauende Features implementiert wurden.

### Erlaubt vs. Verboten nach Abzweig von einem offenen PR-Branch

- **Erlaubt**:
  - Normale zusätzliche Commits auf dem PR-Branch **vor** dem Abzweig von B.
  - Normale Arbeit auf `feat/PRIV-143/.../dde` (Child-Branch).
- **Ab dem Zeitpunkt, wo B von A abzweigt**:
  - PR-Branch **nur noch fast-forward** weiterführen (zusätzliche Commits sind ok),
  - **aber keine History-Rewrites** mehr:
    - Kein `git commit --amend` mehr auf A.
    - Kein `git rebase A ...` gefolgt von `git push --force` auf A.
- Nur solange diese Regel eingehalten wird, bleibt die Commit-Basis zwischen A und B stabil und ein späterer `git merge --squash` von `dde` → B-main verhält sich wie im Standard-Szenario.

### Konsequenz bei Regelverletzung (zur Einordnung)

- Wenn nach dem Abzweig von B:
  - auf A ein `--amend`/Rebase + Force-Push passiert,
- dann:
  - hat der PR-Branch **eine neue Commit-ID**,
  - dein Feature-Dev-Branch B basiert noch auf der **alten Commit-ID**,
  - spätere Squash-Merges sehen für Git so aus, als würden **zwei parallele Refactor-Zweige** zusammenlaufen → viele Konflikte, Altlasten und „Merge Changes“.

**Kurzregel:**  
- **Standard:** Tickets immer von `develop` abzweigen → Feature/Main + Feature/Dev → Squash-Merge, MR, fertig.  
- **Feature-on-Feature:** Vom PR-Branch abzuzweigen ist erlaubt, **aber ab dann NIE wieder `commit --amend` oder Rebase + Force-Push auf diesem PR-Branch**, solange Child-Branches davon abhängen.