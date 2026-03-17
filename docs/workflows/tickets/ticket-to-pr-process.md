# Referenzdokument: Entwickler-Workflow von Ticket bis Pull Request unter Max Governance
[INTENT: KONTEXT]

---

## 1. Aufgabenuebersicht
[INTENT: KONTEXT]

Dieses Dokument beschreibt den finalen Entwickler-Workflow von der Ticket-Uebernahme bis zum Pull Request unter der festgelegten Max-Governance-Branching-Architektur.

Ziel dieses Dokuments ist nicht die Diskussion alternativer Architekturen, sondern die praktische, vollstaendige Arbeitsanweisung fuer Entwickler.

Der Workflow basiert auf folgenden festen Grundsaetzen:

- offizieller Arbeitsbranch pro Ticket
- offizieller Arbeitsbranch ist `append-only` und erlaubt **keinen Force Push**
- private oder lokale Exploration ist optional auf `scratch/*` oder lokal moeglich
- Review-Fixes werden auf offiziellen Branches als neue Commits geliefert
- PRs fuer normale Entwicklungsarbeit gehen nach `develop`
- `release/*`, `support/*` und `main` sind keine normalen Entwickler-Arbeitsbranches

---

## 2. Rollen-Schnellreferenz
[INTENT: REFERENZ]

| Branch | Einfache Bedeutung | Wer arbeitet dort direkt? | Typisches Ziel |
|---|---|---|---|
| `main` | aktuell freigegebene Produktionswahrheit | niemand direkt | Produktion |
| `develop` | Integrationslinie fuer die naechste Version | niemand direkt | naechste Release-Linie |
| `release/<semver>` | eingefrorene kommende Version in Stabilisierung | nur kontrollierte Stabilisierung | `main`, danach zurueck nach `develop` |
| `support/<major.minor>` | gepflegte aeltere Produktlinie | nur gezielte Wartung und Hotfixes | Wartung bestehender Version |
| `feature/<ticket>-<slug>` | offizieller Branch fuer ein neues Feature | ja | `develop` |
| `fix/<ticket>-<slug>` | offizieller Branch fuer einen normalen Bugfix | ja | `develop` |
| `docs/<ticket>-<slug>` | offizieller Branch fuer Dokumentation | ja | `develop` |
| `refactor/<ticket>-<slug>` | offizieller Branch fuer Umbauarbeit | ja | `develop` |
| `chore/<ticket>-<slug>` | offizieller Branch fuer Maintenance oder Tooling | ja | `develop` |
| `test/<ticket>-<slug>` | offizieller Branch fuer Test-Arbeit | ja | `develop` |
| `perf/<ticket>-<slug>` | offizieller Branch fuer Performance-Arbeit | ja | `develop` |
| `scratch/<ticket>-<slug>` | optionale private Spielwiese fuer Exploration | optional privat | kein PR-Ziel |

### 2.1 Wie `release/*` und `support/*` in den normalen Entwickler-Workflow passen

Fuer normale Ticket-Arbeit gilt:

- neue Entwicklungsarbeit startet von `develop`
- normale Feature- und Fix-PRs gehen nach `develop`
- `release/*` ist keine allgemeine Entwickler-Spielwiese, sondern eine kontrollierte Stabilisierungslinie
- `support/*` ist keine allgemeine Entwickler-Spielwiese, sondern eine kontrollierte Wartungslinie fuer eine aeltere Version
- wenn eine Aenderung in `release/*` oder `support/*` noetig ist, ist das kein normaler Ticket-Workflow mehr, sondern ein Release- oder Hotfix-Szenario

---

## 3. Naming-Konvention
[INTENT: CONSTRAINT]

### 3.1 Offizielle Branch-Muster

- `feature/<ticket>-<slug>`
- `fix/<ticket>-<slug>`
- `docs/<ticket>-<slug>`
- `refactor/<ticket>-<slug>`
- `chore/<ticket>-<slug>`
- `test/<ticket>-<slug>`
- `perf/<ticket>-<slug>`
- `scratch/<ticket>-<slug>`

### 3.2 Beispiele

- `feature/ABC-123-add-export-button`
- `fix/ABC-456-handle-null-customer-id`
- `docs/ABC-321-update-auth-guide`
- `refactor/ABC-654-split-user-service`
- `scratch/ABC-123-mssql-pool-registry-exploration`

### 3.3 Verbindliche Regeln

- genau ein offizieller Arbeitsbranch pro Ticket
- Ticket-ID direkt vor dem Slug
- Slug in `kebab-case`
- kein offizielles `/main`
- kein offizielles `/dde`
- kein Entwicklername im offiziellen Branch-Namen
- `scratch/*` ist optional und privat; es ist kein offizieller PR-Branch

---

## 4. Standard-Workflow von Ticket bis Pull Request
[INTENT: ANFORDERUNG]

### 4.1 Ticket uebernehmen und klassifizieren

Bevor ein Branch erstellt wird, muss das Ticket fachlich klar genug sein.

Vorgehen:

1. Ticket-ID bestaetigen
2. Ticket-Typ bestimmen:
   - neues Feature -> `feature/*`
   - normaler Bugfix -> `fix/*`
   - Dokumentation -> `docs/*`
   - interner Umbau -> `refactor/*`
   - Maintenance / Tooling -> `chore/*`
   - Tests -> `test/*`
   - Performance -> `perf/*`
3. pruefen, ob das Ticket reviewbar und als einzelne Integrations-Einheit tragfaehig ist
4. wenn das Ticket zu gross ist, vor Branch-Erstellung in kleinere Tickets oder ein Epic zerlegen

### 4.2 Ausgangsbranch synchronisieren

Normale Entwicklungsarbeit startet von `develop`.

```bash
git fetch --prune origin
git switch develop
git pull --ff-only origin develop
```

### 4.3 Offiziellen Ticket-Branch erstellen

```bash
git switch -c feature/ABC-123-add-export-button
```

Bugfix-Beispiel:

```bash
git switch -c fix/ABC-456-handle-null-customer-id
```

### 4.4 Optionale private Scratch-Phase bei Unsicherheit

Wenn du erst suchen, experimentieren oder mit AI groesser refactoren musst, ist eine private Scratch-Phase zulaessig.

Scratch ist sinnvoll, wenn:

- der Loesungsweg noch unklar ist
- du Such- und Explorationsarbeit machen musst
- du groessere AI-gestuetzte Umformungen erst pruefen willst
- du noch **nicht** den offiziellen Review-Stand publizieren willst

Beispiel:

```bash
git switch -c scratch/ABC-123-mssql-pool-registry-exploration
```

Regeln fuer `scratch/*`:

- kein PR aus `scratch/*`
- nur kurzlebig verwenden
- kein Teamstandard daraus machen
- Force Push ist dort optional tolerierbar, wenn euer Team das erlaubt

### 4.5 Stabilen Erkenntnisstand auf den offiziellen Branch ueberfuehren

Sobald die Richtung klar ist, bringst du das Ergebnis auf den offiziellen Ticket-Branch.

Variante A: sauber auf `feature/*` committen

```bash
git switch feature/ABC-123-mssql-pool-registry
git add .
git commit -m "feat(ABC-123): add MSSQL pool registry"
```

Variante B: privates Scratch-Ergebnis per Squash uebernehmen

```bash
git switch feature/ABC-123-mssql-pool-registry
git merge --squash scratch/ABC-123-mssql-pool-registry-exploration
git commit -m "feat(ABC-123): add MSSQL pool registry"
```

Variante C: einzelne Commits gezielt uebernehmen

```bash
git switch feature/ABC-123-mssql-pool-registry
git cherry-pick <commit-sha>
```

Empfehlung:

- nutze `merge --squash`, wenn Scratch nur Explorationsarbeit war
- nutze `cherry-pick`, wenn ein konkreter sauberer Commit aus Scratch erhalten bleiben soll
- committe direkt auf `feature/*`, wenn du gar keinen Scratch-Branch gebraucht hast

### 4.6 Semantisch sauber committen

Der offizielle Ticket-Branch soll eine reviewbare und semantische Commit-Historie tragen.

Empfohlene Muster:

```text
feat(ABC-123): add MSSQL pool registry
test(ABC-123): cover registry bootstrap behavior
docs(ABC-123): document registry configuration
```

Typische saubere Commit-Aufteilung:

- fachliche Implementierung
- Tests
- Doku
- klar getrennte Migration oder API-Anpassung

Nicht Ziel:

- dogmatisch genau ein Commit
- viele unstrukturierte WIP-Commits auf dem offiziellen Branch

### 4.7 Lokale Validierung ausfuehren

Fuehre vor dem ersten Push die projektdefinierten Validierungen aus.

Da dieses Dokument ohne Projektsuche erstellt wurde, sind die konkreten Build- und Test-Kommandos als Platzhalter zu verstehen und muessen durch den Projektstandard ersetzt werden.

```bash
<install-command>
<test-command>
<build-command>
```

Falls euer Projekt Node-basiert ist und `npm` verwendet, kann das beispielsweise so aussehen:

```bash
npm ci
npm test
npm run build
```

### 4.8 Letzte lokale Bereinigung vor dem ersten Push

Vor dem **ersten** Push des offiziellen Ticket-Branchs darfst du lokal noch bereinigen, weil der Branch noch nicht veroeffentlicht ist.

Beispiel:

```bash
git fetch origin
git rebase origin/develop
```

Wichtig:

- diese Bereinigung gilt nur, solange der offizielle Branch noch **nicht** veroeffentlicht ist
- nach dem ersten Push bleibt der offizielle Branch append-only
- kein Force Push auf den offiziellen Branch

### 4.9 Offiziellen Ticket-Branch pushen

```bash
git push -u origin feature/ABC-123-add-export-button
```

Ab jetzt gilt:

- kein Force Push auf `feature/*`
- keine Routine-Rewrites
- neue Aenderungen werden als neue Commits gepusht

### 4.10 Pull Request oder Draft Pull Request erstellen

Normales Ziel fuer Entwicklerarbeit:

- `feature/*` -> `develop`
- `fix/*` -> `develop`
- `docs/*` -> `develop`
- `refactor/*` -> `develop`
- `chore/*` -> `develop`
- `test/*` -> `develop`
- `perf/*` -> `develop`

Erstelle zunaechst einen Draft PR, wenn:

- noch offene Fragen bestehen
- groessere Review-Vorbereitung noetig ist
- du frueh Feedback auf Richtung und Scope willst

### 4.11 Mit Aenderungen auf `develop` waehrend der Arbeit umgehen

Es gibt zwei Phasen:

**Phase A - vor erstem Push:**  
lokales Rebase ist erlaubt

**Phase B - nach erstem Push:**  
kein Rewrite des offiziellen Branchs mehr

Wenn `develop` nach dem ersten Push neue relevante Aenderungen erhalten hat, hast du unter Max Governance drei saubere Wege:

1. auf weitere Synchronisation verzichten, wenn sie fuer den Branch nicht noetig ist
2. `origin/develop` in deinen Branch mergen
3. bei sehr frueher Phase den Branch verwerfen und einen frischen offiziellen Branch auf aktuellem `develop` neu beginnen

Beispiel fuer einen bewussten Merge nach dem ersten Push:

```bash
git fetch origin
git switch feature/ABC-123-add-export-button
git merge origin/develop
```

### 4.12 Review-Fixes einpflegen

Sobald der PR existiert, bleiben offizielle Branches historisch stabil.

Review-Fixes werden als neue Commits geliefert:

```bash
git switch feature/ABC-123-add-export-button
git add .
git commit -m "fix(ABC-123): address review feedback on registry wiring"
git push
```

Nicht Teil dieses Workflows:

- `git commit --amend`
- `git push --force`
- laufende Rewrite-Hygiene waehrend aktiver Review

### 4.13 Merge in `develop`

Der eigentliche Merge in `develop` erfolgt ueber die vom Repository vorgegebene Governance.

Arbeitsregel fuer Entwickler:

- PR sauber halten
- Commit-Serie semantisch halten
- keine globale Squash-Annahme treffen
- Review-Historie nicht durch Rewrite destabilisieren

Repository-seitig sind fuer `feature/* -> develop` zulaessig:

- `rebase merge`, wenn die Commit-Serie bewusst erhalten werden soll
- normaler Merge, wenn die Historie sauber und nachvollziehbar bleiben soll
- selektiver Squash nur dann, wenn internes Rauschen bereinigt werden muss

### 4.14 Aufraeumen nach dem Merge (Optional)
- Je nach Anbieter kann man auch die automatische Löschung des Branches anhaken, wenn man den Pull Request akzeptiert hat. Dadurch fallen diese Schritte unten weg. Das gilt aber natürlich nicht für den Scratch-Run.

Wenn der PR gemerged wurde:

```bash
git branch -d feature/ABC-123-add-export-button
git push origin --delete feature/ABC-123-add-export-button
```

Falls ein Scratch-Branch noch existiert:

```bash
git branch -D scratch/ABC-123-mssql-pool-registry-exploration
```

---

## 5. Commit- und Branch-Regeln fuer Entwickler
[INTENT: CONSTRAINT]

### 5.1 Branch-Regeln

- pro Ticket genau ein offizieller Arbeitsbranch
- keine PRs aus `scratch/*`
- kein offizieller `feature-dev`-Pflichtworkflow
- `scratch/*` ist optional und privat
- kein Force Push auf offiziellem `feature/*`

### 5.2 Commit-Regeln

- Conventional Commits / Angular-Style fuer Commit-Messages verwenden
- 1 bis 3 saubere Commits sind normal
- 4 Commits sind zulaessig, wenn fachlich begruendet
- Tests, Doku oder Migrationen duerfen eigene Commits erhalten
- ein offizieller Branch darf nicht mehrere unabhaengige Tickets mischen

### 5.3 Review-Regeln

- ab offener PR keine Routine-Rewrites
- Review-Fixes als neue Commits
- Branch-Historie bleibt stabil

---

## 6. Schnell-Checkliste fuer Entwickler
[INTENT: REFERENZ]

Vor dem ersten Push:

- Ticket ist klein genug und reviewbar
- richtiger Branch-Typ gewaehlt
- offizieller Branch korrekt benannt
- optionaler Scratch-Branch nur privat genutzt
- Commit-Historie semantisch sauber
- lokale Validierung ausgefuehrt

Vor der PR:

- offizieller Branch ist gepusht
- kein Force Push noetig
- Scope ist auf genau ein Ticket begrenzt
- PR-Ziel ist `develop`
- Commit-Nachrichten sind klar

Waerend der Review:

- neue Aenderungen als neue Commits
- kein `amend`
- kein Force Push

---

## 7. Dateipfad-Index
[INTENT: REFERENZ]

Keine Dateien referenziert.

---

## 8. Ausfuehrungskontext fuer LLM-Agents
[INTENT: KONTEXT]

Dieses Dokument ist als Entwickler-Workflow fuer den taeglichen Umgang mit normalen Tickets gedacht.

Wichtiger Kontext:

- Hotfixes werden **nicht** ueber diesen Workflow abgewickelt, sondern ueber den separaten Hotfix-Workflow.
- Die konkreten Projekt-Kommandos fuer Tests, Build und Installation muessen durch den Repository-Standard ersetzt werden, falls sie von diesem Dokument abweichen.
- `scratch/*` ist eine optionale private Hilfsebene, kein offizieller PR-Branch.
- Die finale Governance verlangt stabilen offiziellen Branch-Verlauf, auch wenn vorher explorative Arbeit noetig war.
