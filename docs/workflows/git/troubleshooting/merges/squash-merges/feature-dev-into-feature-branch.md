## Referenzdokument: Feature‑ / Feature‑Dev‑Branches, Squash‑Merge & PR‑Branches

### Ziel

Dieses Dokument beschreibt den **Standard‑Workflow** für Ticket‑Branches (Feature/Main + Feature‑Dev), erklärt, **warum** der Squash‑Merge dort in der Regel **konfliktfrei** ist, und dokumentiert das **Problemszenario**, das auftritt, wenn man von einem **PR‑Feature‑Branch** abzweigt, der sich später durch `commit --amend` oder Rebase **strukturell ändert** (neue Commit‑ID).  

Es dient als **Referenz & Anleitung** für zukünftige Tickets.

---

## 1. Standard‑Workflow (Ticket → Feature/Main + Feature/Dev)

### 1.1 Branching-Modell

- **Ausgangspunkt**: `develop`
- **Pro Ticket**:
  - **Feature‑Branch (Main)**: `feat/PRIV-123/my-feature/main`
  - **Feature‑Dev‑Branch**: `feat/PRIV-123/my-feature/dde` (oder persönliches Kürzel)

**Erstellung (vereinfacht)**:

```bash
git checkout develop
git pull

# Feature-Main
git checkout -b feat/PRIV-123/my-feature/main

# Feature-Dev (nur für die eigentliche Ticket-Arbeit)
git checkout -b feat/PRIV-123/my-feature/dde
```

Alle **Commits des Tickets** landen ausschließlich auf `.../dde`.  
Der `.../main`‑Branch bleibt **unverändert**, bis der Squash‑Merge erfolgt.

### 1.2 Squash‑Merge ohne Komplikationen

Am Ende des Tickets:

```bash
git checkout feat/PRIV-123/my-feature/main
git merge --squash feat/PRIV-123/my-feature/dde
# mkcommit (oder git commit -m "...") – genau EIN Commit
```

**Architektonischer Hintergrund**:

- Beide Branches haben denselben Basis‑Commit:

  ```text
  develop ──●── A
              \
               ●── F_main
                \
                 ●●●●●── F_dde
  ```

- Zwischen A und dem Squash‑Zeitpunkt gilt:
  - `F_main` bekommt **keine eigenen Commits** (nur von A geerbt).
  - `F_dde` enthält alle Ticket‑Änderungen.
- Aus Git-Sicht bedeutet `merge --squash F_dde` auf `F_main`:
  - Nimm **Diff(A → F_dde)** und wende es auf **F_main** an.
  - Da `F_main` == A (inhaltlich), ist das eine reine „Apply Patch“-Operation.
  - Die Wahrscheinlichkeit für Konflikte ist minimal und beschränkt sich auf:
    - eigene lokale Änderungen auf `F_main` (die es laut Workflow **nicht geben sollte**),
    - oder parallele Änderungen auf `develop` nach dem Abzweig und vor dem Squash (die im Standardprozess vermieden werden).

**Fazit:**  
Solange du:

- **immer von `develop` abzweigst** und
- **nur auf dem Dev‑Branch arbeitest**,

ist ein `git merge --squash` vom Dev‑Branch zurück auf den Feature‑Main‑Branch **architektonisch sauber** und praktisch **im Normalfall konfliktarm/-frei**.

---

## 2. Arbeiten auf Basis eines PR‑Feature‑Branches (Stacked Branches)

Manchmal soll ein neues Ticket **auf einem noch nicht gemergten Feature** aufbauen, z.B.:

- PR für Feature A ist offen.
- Du musst Ticket B umsetzen, das **fachlich auf A** aufsetzt.

Dann passiert:

```bash
# PR-Branch für Feature A
feat/PRIV-100/feature-a/main

# Neuer Ticket-Branch B:
git checkout feat/PRIV-100/feature-a/main
git checkout -b feat/PRIV-143/refactor-dbf-file-lib/main
git checkout -b feat/PRIV-143/refactor-dbf-file-lib/dde
```

Das ist ein **Stacked Branch**: Feature B baut bewusst auf dem Stand von Feature A auf.

**Wichtig:**  
Das ist **kein Anti‑Pattern**, aber es ist **ein fortgeschrittener Workflow**, der sauber funktioniert, solange:

- der Parent‑Branch (`feat/PRIV-100/...`) **nachträglich nicht mehr per Rebase/Amend verändert** wird, oder
- du alle Child‑Branches danach **sofort darauf rebasest**.

---

## 3. Problemszenario: Amend/Rebase auf PR‑Branch nach Abzweig

### 3.1 Was ist passiert?

Vereinfacht war deine Situation:

1. **PR‑Branch A** existiert (`feat/OLD/main`), Commit-ID `A1`.
2. Du erstellst darauf deinen Ticket‑Branch B:

   ```text
   A1 ──●── B_main
          \
           ●●●... B_dde
   ```

3. Später machst du auf dem **PR‑Branch A** (oder jemand anders) ein:

   ```bash
   git commit --amend
   git push --force
   ```

   → aus `A1` wird **`A1'`** (neue Commit‑ID, auch wenn der Inhalt ähnlich ist).

4. Dein `B_dde` ist aber immer noch von **A1** abgezweigt, während dein „wiederhergestellter“ `B_main` inzwischen auf A1' (oder sogar einem anderen Stand) basiert.

Der Commit‑Graph sieht dann aus wie zwei parallele Linien:

```text
develop ──●── A0
            \
             ●── A1 ──●●● B_dde   (dein Dev-Branch, alte Basis)
              \
               ●── A1' ──● B_main (PR-Branch nach Amend, oder neu erzeugter Branch)
```

Wenn du jetzt:

```bash
git checkout B_main
git merge --squash B_dde
```

machst, sieht Git:

- Gemeinsamer Vorgänger ist **A0 oder A1**, nicht A1'.
- Beide Seiten haben seitdem **eigene Änderungen** unter denselben Pfaden vorgenommen:

  - „both added“ neue Dateien,
  - „both deleted“ alte Dateien,
  - „both modified“ denselben Code.

Ergebnis: **viele Merge-Konflikte und „Merge Changes“**, obwohl du subjektiv „auf dem gleichen Feature-Stand“ arbeitest.

### 3.2 Warum reichen gleiche Branch‑Namen nicht?

- Branch‑Name gleich → **irrelevant** für Git.
- Inhalt „sieht gleich aus“ → kann trotzdem unterschiedliche Commit-Historie sein.
- `git commit --amend` und `git rebase` erzeugen **immer neue Commit-IDs**.

Aus Git-Sicht sind das dann **verschiedene Welten**, die beim Squash wieder zusammengesetzt werden müssen.

---

## 4. Architekturell korrekte Lösung im Problemszenario

Wenn du in dieser Situation landest (wie in deinem aktuellen Ticket), gibt es zwei Ebenen:

### Option a) Kurzfristig: Konflikte sauber auflösen

- `git status` ist die Wahrheit:  
  Nur Dateien unter **„Unmerged paths“** blockieren dich.
- Pro Datei:
  - Merge‑Editor öffnen.
  - In jedem Konflikt‑Block entscheiden: **Current**, **Incoming** oder **Both**.
  - Datei speichern → `git add <file>`.
- Nicht erschrecken, wenn VS Code viele „Merge Changes“ zeigt:
  - Dateien nur unter „Changes to be committed“ sind **schon automatisch gemerged**.
  - Das Result‑Pane mit „No Changes Accepted“ ist in solchen Fällen nur eine Review‑Hilfe.














<br><br>
<br><br>


### Option b) Force add incoming (PowerShell, mit Altlasten-Bereinigung & Force-Push)
- In Fällen, wo man explizit weiß, welche Ordner oder Dateien nur relevant sind für die **Merge-Konflikte**, können alle anderen einfach akzeptiert und **editiert** werden.


<details><summary>Click to expand..</summary>


- **Use-Case**: Du weißt, dass dein **Feature‑Dev‑Branch** (`dde`) fachlich der Wahrheit entspricht und du nur in einem klar abgegrenzten Ordner (z.B. `src/http`) manuell mergen willst.
- Annahme: Du befindest dich auf dem **Feature‑Main‑Branch** und hast bereits:

```bash
git merge --squash refactor/PRIV-143/refactor-dbf-file-lib/dde
```

- Für diesen Squash gilt:
  - **ours** = aktueller Branch (`.../main`, PR‑Stand),
  - **theirs** = Feature‑Dev‑Branch (`.../dde`).

---

### PowerShell-Skript: „Alles von Dev, außer `src/http` – inklusive Löschungen“

```powershell
param(
    # Name deines Feature-Dev-Branches (theirs)
    [string]$DevBranch = "refactor/PRIV-143/refactor-dbf-file-lib/dde",
    # HTTP-Root, das NICHT automatisch übernommen werden soll
    [string]$HttpRoot = "src/http"
)

Write-Host "🔧 Using Dev branch (theirs): $DevBranch"
Write-Host "🔧 HTTP root excluded from auto-merge: $HttpRoot"
Write-Host ""

# 0) Stale index.lock aufräumen (falls vorheriger Git-Prozess abgestürzt ist)
if (Test-Path ".git\index.lock") {
    Write-Host "⚠️  Removing stale .git/index.lock"
    Remove-Item ".git\index.lock" -Force
}

# 1) Alle ungemergten Blobs (Stages) holen
# Format: <mode> <object> <stage>\t<path>
$unmerged = git ls-files -u

if (-not $unmerged) {
    Write-Host "✅ No unmerged paths found. Nothing to do."
    git status
    exit 0
}

# 2) Alle Dateien ermitteln, die eine 'theirs'-Version (Stage 3) haben
$theirsFiles = $unmerged |
    ForEach-Object {
        $parts = $_ -split '\s+'
        # parts[2] = Stage (1,2,3), parts[3] = Pfad
        if ($parts[2] -eq '3') { $parts[3] }
    } |
    Sort-Object -Unique

# 3) Für alle diese Dateien außerhalb von HTTP: "theirs" übernehmen (Dev-Branch)
$toAcceptTheirs = $theirsFiles | Where-Object { -not $_.StartsWith($HttpRoot) }

Write-Host "📂 Accepting 'theirs' (DevBranch) for files outside $HttpRoot..."
foreach ($f in $toAcceptTheirs) {
    Write-Host "  → theirs: $f"
    git checkout --theirs -- $f
    git add $f
}

# 4) Verbleibende unmerged paths prüfen (typische Fälle: both deleted etc.)
$remainingUnmerged = git diff --name-only --diff-filter=U

if ($remainingUnmerged) {
    Write-Host ""
    Write-Host "📂 Handling remaining unmerged paths outside $HttpRoot (typisch: both deleted / Altlasten)..."
    $toDelete = $remainingUnmerged | Where-Object { -not $_.StartsWith($HttpRoot) }

    foreach ($f in $toDelete) {
        Write-Host "  → removing (rm): $f"
        git rm -- $f
    }
}

# 5) Altlasten entfernen: Dateien, die NUR im aktuellen Branch existieren (nicht im Dev-Branch),
#    sollen außerhalb von HTTP gelöscht werden (Baum == Dev, außer HTTP).
Write-Host ""
Write-Host "🧹 Removing files that exist only in current branch (Altlasten), compared to $DevBranch..."

# git diff <DevBranch> --name-status:
# A = Datei nur im aktuellen Branch (Altlast aus main)
$diff = git diff --name-status $DevBranch --

$extraFiles = $diff |
    Where-Object { $_ -match '^\s*A\s+' } |
    ForEach-Object {
        ($_ -split '\s+')[1]
    } |
    Where-Object { -not $_.StartsWith($HttpRoot) }

foreach ($f in $extraFiles) {
    Write-Host "  → removing extra file: $f"
    git rm -- $f
}

Write-Host ""
Write-Host "✅ Auto-Übernahme abgeschlossen."
Write-Host "   - Alle Dateien außerhalb '$HttpRoot' sind jetzt auf dem Stand von '$DevBranch' (inkl. Löschungen)."
Write-Host "   - '$HttpRoot' wurde NICHT angefasst – dort kannst du Konflikte manuell lösen."
Write-Host ""

git status
Write-Host ""
Write-Host "👉 Nächste Schritte:"
Write-Host "   1. Manuelle Konflikte nur noch unter '$HttpRoot' im Merge-Editor lösen und 'git add' ausführen."
Write-Host "   2. Squash-Commit erstellen (mkcommit / git commit)."
Write-Host "   3. Anschließend den Feature-Main-Branch mit 'git push --force' (bzw. '--force-with-lease') zum Remote pushen,"
Write-Host "      weil der Squash-Merge die Historie des Feature-Branches neu schreibt."
```

### Was macht das Skript (kurz)?

- **Schritt 1–3**:  
  Findet alle Konfliktdateien, die eine `theirs`‑Version haben (Stage 3) und **nicht** unter `src/http/**` liegen, und übernimmt für diese vollständig den Stand des **Feature‑Dev‑Branches** (`git checkout --theirs` + `git add`).

- **Schritt 4**:  
  Räumt verbleibende Konflikte außerhalb von `src/http` auf, bei denen es typischerweise um gelöschte Altlasten (`both deleted`) geht (`git rm`).

- **Schritt 5**:  
  Entfernt zusätzlich alle Dateien außerhalb von `src/http`, die **nur im aktuellen Branch** existieren (Altlasten, die im Dev‑Branch schon nicht mehr da sind), sodass der Baum außerhalb von `src/http` wirklich **1:1 dem Dev‑Branch entspricht**.

- **Danach**:
  - Manuell nur noch `src/http/**` mergen.
  - Squash‑Commit erstellen.
  - **Wichtig**: Da du mit Squash und dieser Bereinigung die Historie des Feature‑Main‑Branches geändert hast, **MUSS** der Push zum Remote mit `--force` (oder besser `--force-with-lease`) erfolgen:

    ```bash
    git push --force --set-upstream origin refactor/PRIV-143/refactor-dbf-file-lib/main
    ```

</details>

















<br><br>
<br><br>


### 4.2 Mittelfristig: Wie macht man es „richtig“, wenn man auf PR‑Stand weiterarbeiten MUSS?

**Option A – Stabiler PR‑Branch (empfohlen bei stacked Branches)**

- Sobald ein Child‑Branch (dein Ticket) von einem PR‑Branch abzweigt, gelten:
  - **Kein `commit --amend` mehr auf dem PR‑Branch.**
  - **Kein Rebase** auf dem PR‑Branch, der ohne Rebase des Child‑Branches bleibt.
- Wenn doch nötig:
  - PR‑Branch rebasen/amenden,
  - danach **alle Child‑Branches direkt darauf rebasen**:

    ```bash
    git checkout feat/PRIV-143/refactor-dbf-file-lib/dde
    git rebase feat/OLD/main   # der neue Stand des PR-Branches
    ```

**Option B – Disziplin: Nur von `develop` für neue Tickets**

- Für **normale Tickets** gilt weiterhin:
  - **immer von `develop` abzweigen**,
  - pro Ticket `feat/.../main` + `feat/.../dde`,
  - `feat/.../main` bis zum Squash **nie anfassen**.
- Wenn ein Ticket fachlich auf einem offenen PR aufbauen soll:
  - explizit als **Stacked Branch** markieren (z.B. in Jira: „depends on PRIV‑XYZ“),
  - Team abstimmen, dass der PR‑Branch ab diesem Zeitpunkt **nicht mehr rebaset/amendet** wird.

---

## 5. Zusammenfassung (als „Regelwerk“)

- **Standard‑Pfad (empfohlen, 80–90 % aller Tickets):**
  - Abzweig **immer** von `develop`.
  - Pro Ticket: `feat/.../main` (Basis) + `feat/.../dde` (Arbeit).
  - `feat/.../main` bleibt unverändert bis zum `merge --squash`.
  - Squash‑Merge auf `feat/.../main` ist damit **stabil und weitgehend konfliktfrei**.

- **Stacked Branches (Feature auf Feature / PR‑Stand):**
  - Nur nutzen, wenn fachlich nötig.
  - Ab Abzweig des Child‑Branches:
    - PR‑Branch **nicht mehr** per `commit --amend` oder `rebase` verändern **ohne** das Child mitzuziehen.
  - Wenn Rebase/Amend doch nötig → Child‑Branch direkt auf den neuen PR‑Commit rebasen.

- **Anti‑Pattern in deinem Fall:**
  - PR‑Branch nachträglich mit `--amend` / Rebase verändert,
  - Child‑Branch weiter auf der alten Historie gelassen,
  - späterer Squash‑Merge führt zu riesigen „Merge Changes“.

Dieses Dokument kannst du direkt als **Team‑Referenz** verwenden („Branching‑Guidelines für Feature/Main + Feature/Dev und Arbeiten auf PR‑Ständen“).