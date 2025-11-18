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


### Option b) Force add incoming
- In Fällen, wo man explizit weiß, welche Ordner oder Dateien nur relevant sind für die **Merge-Konflikte**, können alle anderen einfach akzeptiert und **editiert** werden.


<details><summary>Click to expand..</summary>

- Bei deinem `git merge --squash feat/.../dde` gilt:
  - **ours** = aktueller Branch (`.../main`, PR‑Stand),
  - **theirs** = Feature‑Dev‑Branch (`.../dde`).
- Du willst: **für alle Konfliktdateien außer `src/http/**` → `theirs` akzeptieren**,  
  und **`src/http/**` erstmal offen lassen** (später manuell mergen).

---

### 1. Nur Konfliktdateien außerhalb von `src/http` automatisch auf `theirs` setzen

#### Variante A – PowerShell (deine Umgebung)

Im Root des Repos:

```powershell
# 1) Liste aller ungemergten Dateien holen
$conflicts = git diff --name-only --diff-filter=U

# 2) Alle Konfliktdateien außer src/http/** auswählen
$toAcceptTheirs = $conflicts | Where-Object { -not $_.StartsWith('src/http/') }

# 3) Für diese Dateien jeweils "theirs" in die Working Copy schreiben und als gelöst markieren
foreach ($f in $toAcceptTheirs) {
    git checkout --theirs -- $f
    git add $f
}
```

Ergebnis:

- Alle Konfliktdateien **außer** `src/http/**` sind jetzt:
  - mit dem Inhalt deines **Feature‑Dev‑Branches** (`dde`) im Working Tree,
  - im Index als „resolved“ (`git status` zeigt sie nicht mehr unter „unmerged paths“).

#### Variante B – Git Bash / WSL

Falls du lieber in einer Bash arbeitest:

```bash
git diff --name-only --diff-filter=U \
  | grep -v '^src/http/' \
  | while read -r f; do
      git checkout --theirs -- "$f"
      git add "$f"
    done
```

---

### 2. Was ist danach noch zu tun?

- `git status` wird jetzt nur noch **Konflikte unter `src/http/**`** anzeigen.
- Genau diese Dateien kannst du dann gezielt im VS‑Code‑Merge‑Editor öffnen und sauber mergen (Current vs. Incoming).
- Wenn **keine „Unmerged paths“** mehr übrig sind:

```bash
git status  # prüfen
# dann deinen Squash-Commit erzeugen
mkcommit    # oder: git commit -m "feat(PRIV-143): ..."
```

---

### 3. Warum das funktioniert

- `git checkout --theirs -- <file>` schreibt die Version deines **Dev‑Branches** (`feat/.../dde`) in die Working Copy, lässt die Version von `main` fallen und entfernt die Konfliktmarker.
- `git add <file>` sagt Git: „Konflikt ist hier gelöst, nimm diesen Inhalt für den Squash‑Commit.“
- Durch das Filtern auf `src/http/` steuerst du, **wo** du auto‑übernehmen willst (Dev‑Branch) und **wo** du bewusst manuell mergen willst (HTTP‑Layer).

Damit löst du in einem Rutsch ~230 Dateien und musst nur noch den Bereich anfassen, der dir fachlich wichtig ist (`src/http`).
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