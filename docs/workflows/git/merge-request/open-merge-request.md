
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




<br><br>
<br><br>

---


<br><br>
<br><br>









### Workflow: Erneuter Rebase nach Änderungen auf `develop`

Dieser Abschnitt beschreibt, wie du vorgehst, wenn nach deinem ersten Rebase bzw. nach Erstellung des PR weitere Änderungen auf `develop` landen und du **erneut rebasen** musst – unter Einhaltung der **Ein-Commit-Konvention** auf dem Feature-Branch.

---

### Ausgangssituation

- **Feature-Branch** (z. B. `feat/PRIV-001/add-new-button/main`) enthält **genau einen Commit** (erzeugt mit `mkcommit`).
- Auf Basis dieses Branches existiert bereits ein **Pull Request**.
- Nachträglich werden weitere Änderungen auf **`develop`** gemerged.
- Ziel: **Feature-Branch auf den neuesten Stand von `develop` bringen**, ohne die Ein-Commit-Konvention zu verletzen.

---

### Standard-Workflow bei einem erneuten Rebase

1. **`develop` aktualisieren**

   ```bash
   git checkout develop
   git pull
   ```

2. **Zurück auf den Feature-Branch wechseln**

   ```bash
   git checkout feat/PRIV-001/add-new-button/main
   ```

3. **Erneuten Rebase auf den aktuellen `develop`-Stand durchführen**

   ```bash
   git rebase develop
   ```

4. **Eventuelle Konflikte lösen**

   - Konflikte in den betroffenen Dateien manuell beheben.
   - Danach die gelösten Dateien hinzufügen:

     ```bash
     git add .
     ```

   - Rebase fortsetzen:

     ```bash
     git rebase --continue
     ```

5. **Tests erneut ausführen**

   - Sicherstellen, dass alle relevanten Tests wieder erfolgreich sind (Unit-, Integrations-, ggf. E2E-Tests).

6. **Feature-Branch erneut pushen (mit Force)**

   - Da der Rebase die Commit-Historie verändert, ist ein Force-Push erforderlich:

     ```bash
     git push --force-with-lease
     ```

   - `--force-with-lease` ist sicherer als ein einfaches `--force`, da es verhindert, dass du versehentlich fremde Änderungen überschreibst.

---

### Verhalten in Bezug auf die Ein-Commit-Konvention

- **Wichtig:** Ein erneuter Rebase auf `develop` erfordert **nicht automatisch** einen neuen `amend`-Commit.
- Solange auf deinem Feature-Branch weiterhin **nur ein Commit** existiert (also der von `mkcommit`), bleibt die Ein-Commit-Konvention erfüllt.
- Der Rebase „setzt“ deinen **einen Commit** lediglich erneut oben auf den aktualisierten `develop`-Stand:
  - Der **Inhalt** deines Commits bleibt gleich.
  - Nur der **Commit-Hash** ändert sich technisch im Hintergrund.
- In diesem Fall genügt:
  - **Rebase ausführen** und
  - **`git push --force-with-lease`**,  
  **ohne** einen zusätzlichen `git commit --amend`.

---

### Wann `git commit --amend` sinnvoll ist

Ein `amend` ist **nur dann erforderlich**, wenn du nachträglich:

- den **Inhalt** des Commits ändern willst (z. B. neue Code-Anpassungen nach Review),
- oder die **Commit-Message** anpassen musst (z. B. Breaking-Change-Footer ergänzen).

Typischer Ablauf in diesem Fall:

1. Code anpassen:

   ```bash
   git add .
   git commit --amend
   ```

2. Falls `develop` inzwischen weitergezogen ist, **danach** (oder davor) rebasen:

   ```bash
   git checkout develop
   git pull
   git checkout feat/PRIV-001/add-new-button/main
   git rebase develop
   ```

3. Zum Schluss wieder:

   ```bash
   git push --force-with-lease
   ```

---

### Kurz zusammengefasst

- **Erneuter Rebase allein** ⇒  
  **Kein** zusätzlicher `amend` nötig, **solange nur ein Commit** auf dem Feature-Branch existiert.  
  Workflow: `develop` pullen → Feature-Branch rebasen → Konflikte lösen → testen → `git push --force-with-lease`.
- **`amend` nur dann verwenden**, wenn du **Inhalt oder Message** deines einzigen Commits ändern möchtest.







<br><br>
<br><br>

---

<br><br>
<br><br>


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
