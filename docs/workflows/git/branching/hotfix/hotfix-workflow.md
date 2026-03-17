# Referenzdokument: Hotfix-Workflow unter Max Governance
[INTENT: KONTEXT]

---

## 1. Aufgabenuebersicht
[INTENT: KONTEXT]

Dieses Dokument beschreibt den finalen Hotfix-Workflow auf Basis der neuen Max-Governance-Branching-Konvention.

Der Hotfix-Workflow unterscheidet sich bewusst vom normalen Ticket-Workflow:

- ein Hotfix startet **nicht** standardmaessig von `develop`
- ein Hotfix startet immer von der Linie, die den Fehler real traegt
- ein Hotfix wird nicht stillschweigend in andere Linien "mitgedacht", sondern bewusst weitergeleitet
- Hotfix-Lineage muss auf kritischen Linien sichtbar bleiben

Dieses Dokument erklaert:

- was `main`, `release/*` und `support/*` in Hotfix-Szenarien bedeuten
- wie die richtige Ausgangslinie fuer einen Hotfix gewaehlt wird
- wie ein Hotfix-Branch benannt wird
- welche Commits und PR-Ziele verwendet werden
- wie Forward-Port und Backport sauber erfolgen

---

## 2. Rollen-Schnellreferenz fuer Hotfixes
[INTENT: REFERENZ]

| Branch | Einfache Bedeutung | Wann fuer Hotfix relevant? |
|---|---|---|
| `main` | aktuell freigegebene Haupt-Produktionslinie | wenn der Fehler in der aktuellen Hauptproduktion steckt |
| `release/<semver>` | noch nicht freigegebene, aber bereits stabilisierte kommende Version | wenn der Fehler in der eingefrorenen kommenden Version steckt |
| `support/<major.minor>` | aeltere, weiterhin gepflegte Produktionslinie | wenn der Fehler nur oder auch in einer aelteren noch supporteten Version steckt |
| `hotfix/<ticket>-<slug>` | kurzlebiger Fix-Branch fuer eine konkrete betroffene Linie | immer der operative Branch fuer den eigentlichen Fix |

Einfaches Bild:

- `main` = aktueller Zug im Betrieb
- `release/2.8.0` = naechster Zug steht am Bahnsteig und wird final geprueft
- `support/2.7` = alter Zug faehrt noch und wird weiterhin gewartet

---

## 3. Naming-Konvention
[INTENT: CONSTRAINT]

Hotfixes verwenden immer:

- `hotfix/<ticket>-<slug>`

Beispiele:

- `hotfix/ABC-999-payment-timeout`
- `hotfix/ABC-1201-fix-oauth-token-refresh`

Verbindliche Regeln:

- kein `/main`
- kein `/dde`
- kein offizieller `hotfix-dev`
- kein Entwicklername im Branch-Namen
- der Branch-Typ beschreibt den Workflow, nicht den Commit-Typ

Wichtige Klarstellung:

- der Branch heisst `hotfix/...`
- die Commit-Message ist in den meisten Faellen trotzdem `fix(...)`

Beispiel:

```text
Branch: hotfix/ABC-999-payment-timeout
Commit: fix(ABC-999): resolve payment timeout during checkout
```

---

## 4. Entscheidungslogik: Von welcher Linie startet der Hotfix?
[INTENT: REFERENZ]

Die wichtigste Hotfix-Regel lautet:

**Der Hotfix startet von der Linie, die den Fehler real enthaelt.**

### 4.1 Entscheidungs-Matrix

| Situation | Startlinie | PR-Ziel |
|---|---|---|
| Fehler betrifft die aktuelle Hauptproduktion | `main` | `main` |
| Fehler betrifft eine noch nicht freigegebene kommende Version | `release/<semver>` | dieselbe `release/<semver>` |
| Fehler betrifft eine weiterhin gepflegte aeltere Version | `support/<major.minor>` | dieselbe `support/<major.minor>` |
| Fehler betrifft mehrere aktive Linien | starte von jeder betroffenen Linie separat oder fuehre zunaechst den primaeren Fix auf der zuerst betroffenen Linie aus und portiere ihn anschliessend bewusst weiter | pro Linie eigener PR |

### 4.2 Was nicht gemacht wird

- kein Hotfix-Start von `develop`, wenn der Fehler in `main` oder `support/*` sitzt
- kein allgemeiner Start "immer von main"
- keine Annahme, dass spaetere Merge-Ketten das Problem automatisch korrekt verteilen

---

## 5. Schritt-fuer-Schritt-Hotfix-Workflow
[INTENT: ANFORDERUNG]

### 5.1 Ticket anlegen oder Incident referenzieren

Ein Hotfix braucht eine stabile Referenz:

- Ticket-ID
- Incident-ID oder Störungskontext
- klare Beschreibung des Fehlers
- betroffene Version oder Linie

### 5.2 Betroffene Linie eindeutig bestimmen

Vor Branch-Erstellung muss feststehen:

- steckt der Fehler in `main`?
- steckt der Fehler in `release/<semver>`?
- steckt der Fehler in `support/<major.minor>`?
- steckt der Fehler in mehreren Linien?

Ohne diese Entscheidung ist der Hotfix-Start architekturell nicht korrekt.

### 5.3 Ausgangslinie synchronisieren

Beispiel fuer `main`:

```bash
git fetch --prune origin
git switch main
git pull --ff-only origin main
```

Beispiel fuer `release/2.8.0`:

```bash
git fetch --prune origin
git switch release/2.8.0
git pull --ff-only origin release/2.8.0
```

Beispiel fuer `support/2.7`:

```bash
git fetch --prune origin
git switch support/2.7
git pull --ff-only origin support/2.7
```

### 5.4 Hotfix-Branch erstellen

```bash
git switch -c hotfix/ABC-999-payment-timeout
```

### 5.5 Hotfix implementieren und semantisch committen

Normalerweise wird fuer den Commit-Typ `fix` verwendet:

```bash
git add .
git commit -m "fix(ABC-999): resolve payment timeout during checkout"
```

Wenn ein Breaking Change unvermeidbar ist:

```bash
git add .
git commit -m "fix(ABC-999)!: rename payment timeout configuration" -m "BREAKING CHANGE: PAYMENT_TIMEOUT_MS is replaced by CHECKOUT_TIMEOUT_MS"
```

Hinweis:

- ein Hotfix ist nicht automatisch ein Ein-Commit-Dogma
- 1 bis 3 saubere Commits sind zulaessig
- die Historie muss dennoch kompakt und nachvollziehbar bleiben

### 5.6 Lokale Validierung ausfuehren

Fuehre die projektdefinierten Validierungen aus.

Da dieses Dokument ohne Projektsuche erstellt wurde, muessen die folgenden Platzhalter durch die Projektstandards ersetzt werden.

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

### 5.7 Hotfix-Branch pushen

```bash
git push -u origin hotfix/ABC-999-payment-timeout
```

### 5.8 Pull Request fuer die betroffene Linie erstellen

Beispiele:

- `hotfix/ABC-999-payment-timeout` -> `main`
- `hotfix/ABC-999-payment-timeout` -> `release/2.8.0`
- `hotfix/ABC-999-payment-timeout` -> `support/2.7`

### 5.9 Merge-Strategie

Fuer Hotfixes gilt:

- keine globale Squash-Pflicht
- keine History-Rewrites
- `merge commit` bevorzugen, damit die Hotfix-Lineage explizit bleibt

### 5.10 Hotfix in weitere aktive Linien weiterleiten

Nach dem Merge in die zuerst betroffene Linie muss entschieden werden, welche weiteren aktiven Linien den Fix ebenfalls brauchen.

Typische Ziele:

- `develop`
- eine aktuelle `release/<semver>`
- weitere `support/<major.minor>`-Linien

Empfohlenes Muster:

```bash
git fetch origin
git switch develop
git pull --ff-only origin develop
git switch -c fix/ABC-999-forward-port-payment-timeout
git cherry-pick -x <hotfix-commit-sha>
git push -u origin fix/ABC-999-forward-port-payment-timeout
```

Wenn eine weitere Support-Linie den Fix braucht:

```bash
git fetch origin
git switch support/2.7
git pull --ff-only origin support/2.7
git switch -c fix/ABC-999-forward-port-support-2.7
git cherry-pick -x <hotfix-commit-sha>
git push -u origin fix/ABC-999-forward-port-support-2.7
```

Warum `cherry-pick -x`?

- Herkunft bleibt sichtbar
- Rueckverfolgung bleibt einfach
- Review sieht klar, dass es sich um dieselbe inhaltliche Hotfix-Aenderung handelt

### 5.11 Branches nach Abschluss aufraeumen

Nach erfolgreichem Merge:

```bash
git branch -d hotfix/ABC-999-payment-timeout
git push origin --delete hotfix/ABC-999-payment-timeout
```

Forward-Port-Branches werden nach ihrem jeweiligen Merge ebenfalls geloescht.

---

## 6. Commit- und Review-Regeln fuer Hotfixes
[INTENT: CONSTRAINT]

- Hotfix-Branch startet immer von der real betroffenen Linie
- Hotfix-Branch ist offiziell und nicht fuer Force Push gedacht
- Review-Fixes werden als neue Commits geliefert
- `merge commit` ist fuer Hotfixes der bevorzugte Merge-Weg
- es wird nichts implizit "spaeter schon automatisch" in andere Linien gelangen
- Forward-Port oder Backport wird bewusst und nachweisbar ausgefuehrt

---

## 7. Beispiele fuer typische Hotfix-Szenarien
[INTENT: REFERENZ]

### 7.1 Produktionsfehler nur in aktueller Hauptlinie

```text
Start: main
Branch: hotfix/ABC-999-payment-timeout
PR: hotfix/ABC-999-payment-timeout -> main
Danach: pruefen, ob develop den Fix ebenfalls braucht
```

### 7.2 Fehler nur in supporteter Altversion

```text
Start: support/2.7
Branch: hotfix/ABC-1201-fix-token-refresh
PR: hotfix/ABC-1201-fix-token-refresh -> support/2.7
Danach: pruefen, ob main und develop ebenfalls betroffen sind
```

### 7.3 Fehler in eingefrorener, aber noch nicht freigegebener Version

```text
Start: release/2.8.0
Branch: hotfix/ABC-1300-fix-release-blocker
PR: hotfix/ABC-1300-fix-release-blocker -> release/2.8.0
Danach: denselben Fix nach develop zurueckfuehren
```

---

## 8. Schnell-Checkliste fuer Hotfixes
[INTENT: REFERENZ]

Vor Branch-Erstellung:

- Ticket oder Incident referenziert
- betroffene Linie eindeutig bestimmt
- klar, ob weitere Linien ebenfalls betroffen sind

Vor PR:

- Hotfix-Branch korrekt benannt
- Commit-Historie kompakt und semantisch sauber
- lokale Validierung ausgefuehrt
- Ziel-PR zeigt auf die wirklich betroffene Linie

Nach Merge:

- Forward-Port- oder Backport-Bedarf bewusst geprueft
- Folge-Branches bei Bedarf erstellt
- Herkunft per `cherry-pick -x` oder vergleichbar nachvollziehbar gehalten

---

## 9. Dateipfad-Index
[INTENT: REFERENZ]

Keine Dateien referenziert.

---

## 10. Ausfuehrungskontext fuer LLM-Agents
[INTENT: KONTEXT]

Dieses Dokument ist der operative Hotfix-Leitfaden fuer die festgelegte Max-Governance-Architektur.

Relevanter Kontext:

- `develop` ist keine Hotfix-Ausgangslinie fuer Produktionsfehler.
- `release/*` ist die Linie fuer eingefrorene kommende Versionen.
- `support/*` ist die Linie fuer gepflegte aeltere Versionen.
- Hotfixes muessen nach dem ersten Merge bewusst in weitere aktive Linien propagiert werden.
- Die konkreten Build- und Test-Kommandos sind projektspezifisch und muessen durch den Repository-Standard ersetzt werden, wenn sie von den Platzhaltern abweichen.
- Alle notwendigen Informationen fuer den Hotfix-Ablauf sind in den Sektionen 1-9 vollstaendig enthalten.
