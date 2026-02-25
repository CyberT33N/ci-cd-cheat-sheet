

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



