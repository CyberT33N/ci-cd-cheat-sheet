# Referenzdokument: Release-Workflow nach `develop` unter Max Governance
[INTENT: KONTEXT]

---

## 1. Aufgabenuebersicht
[INTENT: KONTEXT]

Dieses Dokument beschreibt den kompletten Release-Workflow **nach** dem normalen Entwickler-Workflow.

Der normale Entwickler-Workflow endet mit:

- Ticket-Branch -> Pull Request -> `develop`

Ab diesem Punkt beginnt **nicht** mehr die normale Entwicklerarbeit, sondern **Release-Management**.

Dieses Dokument definiert:

- was nach mehreren Merges auf `develop` geschieht
- wann ein `release/<semver>` erzeugt wird
- welche Governance-Trigger den Release-Cut ausloesen duerfen
- welche Schritte manuell von verantwortlichen Rollen entschieden werden
- welche Schritte durch CI/CD oder GitHub-Automatisierung erfolgen sollen
- wie Pull Requests, Tags und Releases sauber aufeinander folgen
- wie ein manueller Fallback aussieht, wenn keine belastbare Release-Pipeline vorhanden ist

---

## 2. Grundprinzipien
[INTENT: CONSTRAINT]

- `develop` ist die Integrationslinie fuer die naechste Version.
- Nicht jeder Merge nach `develop` erzeugt automatisch einen Release-Branch.
- Ein `release/<semver>` entsteht nur bei einem **bewussten Governance-Trigger**.
- Ein Release-Cut ist zuerst eine **Governance-Entscheidung**, danach ein technischer Ausfuehrungsvorgang.
- CI/CD soll die technische Freigabe soweit wie moeglich automatisieren.
- CI/CD soll **nicht** autonom entscheiden, ob der Scope jetzt freigegeben wird.
- Der Uebergang von `release/<semver>` nach `main` erfolgt per Pull Request.
- Nach Freigabe wird `release/<semver>` wieder nach `develop` zurueckgefuehrt.
- Wenn eine Version laenger gepflegt werden muss, kann zusaetzlich `support/<major.minor>` entstehen.

---

## 3. Rollen-Schnellreferenz
[INTENT: REFERENZ]

| Branch | Rolle im Release-Prozess | Wer entscheidet typischerweise darueber? |
|---|---|---|
| `develop` | integrierter Stand fuer die naechste Version | Teams, Maintainer, Release-Verantwortliche |
| `release/<semver>` | eingefrorene kommende Version in Stabilisierung | Release-Verantwortliche oder definierte Governance-Regel |
| `main` | freigegebene Produktionswahrheit | Release-Verantwortliche, Maintainer, Governance-Freigabe |
| `support/<major.minor>` | langlebige Wartungslinie einer aelteren Version | Release-/Support-Verantwortliche |

Beispiel:

- `develop` = naechste Version wird gesammelt
- `release/2.8.0` = Version 2.8.0 ist eingefroren und wird final stabilisiert
- `main` = offiziell freigegebene Version
- `support/2.7` = Version 2.7 wird weiter gepflegt

---

## 4. Moegliche Governance-Trigger fuer einen Release-Cut
[INTENT: REFERENZ]

Ein `release/<semver>` darf durch drei architekturell saubere Trigger entstehen.

### 4.1 Szenario A - Teamuebergreifende manuelle Freigabeentscheidung

Typisches Muster:

- mehrere PRs wurden in `develop` integriert
- Release-Verantwortliche oder teamuebergreifende Governance entscheiden bewusst:
  "Der aktuelle Stand von `develop` wird jetzt als Version `2.8.0` eingefroren."

Das ist die klassischste Max-Governance-Variante.

### 4.2 Szenario B - Kalender- oder Release-Train-basierter Trigger

Typisches Muster:

- es gibt eine feste Release-Regel, zum Beispiel:
  - jeden Mittwoch 12:00
  - jeden zweiten Freitag
  - immer am Ende eines Sprints
- dieser Zeittrigger ist **nicht** spontan, sondern vorab als Governance-Regel festgelegt

In diesem Modell ist der Zeitplan Governance, nicht nur Technik.

### 4.3 Szenario C - Policy-, Milestone- oder Freigabestatus-basierter Trigger

Typisches Muster:

- ein Milestone ist freigegeben
- ein Release Board steht auf "approved"
- alle erforderlichen Governance-Felder oder Freigaben sind gesetzt
- daraufhin darf ein `release/<semver>` erzeugt werden

Auch hier bleibt die inhaltliche Release-Entscheidung governance-getrieben, selbst wenn der technische Ausloeser spaeter automatisiert erfolgt.

---

## 5. Release-Reihenfolge auf hoher Ebene
[INTENT: REFERENZ]

Die korrekte Reihenfolge lautet:

1. mehrere PRs werden in `develop` gemerged
2. `develop` repraesentiert den integrierten Stand fuer die naechste Version
3. ein definierter Governance-Trigger feuert
4. `release/<semver>` wird aus `develop` erzeugt
5. auf `release/<semver>` erfolgen nur noch Stabilisierung und Freigabevorbereitung
6. `release/<semver>` geht per PR nach `main`
7. nach erfolgreicher Freigabe wird der Release-Stand getaggt
8. `release/<semver>` wird wieder nach `develop` zurueckgefuehrt
9. optional wird `support/<major.minor>` angelegt, wenn die Version laenger gepflegt werden muss

---

## 6. Workflow A - Teamuebergreifend manuell entschiedener Release-Cut
[INTENT: ANFORDERUNG]

Dieses Szenario gilt, wenn Release-Verantwortliche den Cut bewusst freigeben.

### 6.1 Vorbedingungen

- relevante Tickets sind in `develop` integriert
- Scope fuer die Version ist klar
- Release-Verantwortliche bestaetigen den Cut
- Zielversion steht fest, zum Beispiel `2.8.0`

### 6.2 `develop` synchronisieren

```bash
git fetch --prune origin
git switch develop
git pull --ff-only origin develop
```

### 6.3 Release-Branch erstellen

```bash
git switch -c release/2.8.0
git push -u origin release/2.8.0
```

### 6.4 Release-Stabilisierung auf `release/2.8.0`

Ab jetzt sind auf `release/2.8.0` nur noch folgende Arten von Aenderungen zulaessig:

- Release-blockierende Bugfixes
- letzte technische oder operative Doku
- Versions- und Freigabeanpassungen
- Freigabevorbereitung

Nicht mehr zulaessig:

- neue Features
- ungeplante Refactors
- themenfremde Tickets

### 6.5 Pull Request nach `main`

Es wird ein PR erstellt:

- `release/2.8.0` -> `main`

Empfohlene Merge-Strategie:

- `merge commit`

Warum:

- Release-Lineage bleibt sichtbar
- der eingefrorene Release-Stand ist klar nachvollziehbar

### 6.6 Tag und Release-Erstellung

**Zielarchitektur: Die Pipeline soll Tag und Release erzeugen.**

Idealbild:

- Merge nach `main`
- Pipeline validiert den Merge-Stand
- Pipeline erzeugt `v2.8.0`
- Pipeline erzeugt Artefakte und optional GitHub Release

### 6.7 Rueckfuehrung nach `develop`

Es wird ein zweiter PR erstellt:

- `release/2.8.0` -> `develop`

Damit landen alle Stabilisierungsfixes wieder auf der Integrationslinie.

### 6.8 Optionaler Support-Branch

Wenn Version `2.8` laenger gepflegt werden soll:

```bash
git fetch origin
git switch main
git pull --ff-only origin main
git switch -c support/2.8
git push -u origin support/2.8
```

---

## 7. Workflow B - Kalender- oder Release-Train-basierter Release-Cut
[INTENT: ANFORDERUNG]

Dieses Szenario gilt, wenn ein vorher definierter Zeitplan den technischen Release-Cut ausloest.

### 7.1 Governance-Logik

Beispiel:

- jeden Mittwoch 12:00 ist Release-Cut-Zeit
- nur der aktuelle Stand von `develop` wird geschnitten
- falls Governance den Cut fuer diese Woche aussetzt, wird kein Release erstellt

### 7.2 Technische Zielarchitektur

Die Freigaberegel wird von Menschen oder Governance festgelegt, aber die technische Erzeugung des Release-Branchs kann automatisiert werden.

Typischer Ablauf:

1. Scheduler oder zeitbasierter Workflow startet
2. Workflow prueft, ob alle Freigabebedingungen fuer den Release-Cut vorliegen
3. Workflow erstellt `release/<semver>` aus `develop`
4. danach laeuft derselbe Stabilisierungspfad wie in Workflow A

### 7.3 Wichtige Architektureinschaetzung

Der Zeittrigger ist **nicht** die Architekturentscheidung selbst.  
Er ist nur die technische Ausfuehrung einer bereits vorher definierten Governance-Regel.

---

## 8. Workflow C - Policy- oder Milestone-basierter Release-Cut
[INTENT: ANFORDERUNG]

Dieses Szenario gilt, wenn ein definierter Status oder Milestone die Freigabe technisch ausloest.

### 8.1 Governance-Logik

Beispiele:

- Milestone `2.8.0` steht auf `approved`
- Release-Freigabe-Checkliste ist vollstaendig
- alle noetigen Governance-Gates wurden erfüllt

### 8.2 Technische Zielarchitektur

Ein Workflow oder internes Tool kann den Release-Cut anstossen, sobald der genehmigte Zustand vorliegt.

Typischer Ablauf:

1. Governance-Status wird freigegeben
2. Automatisierung erstellt `release/2.8.0` aus `develop`
3. Stabilisierung findet auf `release/2.8.0` statt
4. PR nach `main`
5. Pipeline erzeugt Tag und Artefakte
6. Rueck-PR nach `develop`

### 8.3 Wichtige Architektureinschaetzung

Auch hier bleibt die zentrale Architekturregel bestehen:

- die **inhaltliche** Freigabe ist Governance
- die **technische** Branch-Erzeugung und Release-Abwicklung kann automatisiert werden

---

## 9. Soll die Pipeline die Tags setzen?
[INTENT: CONSTRAINT]

### 9.1 Zielarchitektur

Ja. Im Zielbild **sollte** die Pipeline die Tags setzen und die technische Release-Erstellung uebernehmen.

Warum:

- Tag entsteht exakt am kontrollierten Merge-Punkt
- Reproduzierbarkeit steigt
- Artefakt-Erstellung und Tag sind sauber gekoppelt
- weniger manuelle Fehler
- Auditierbarkeit steigt

### 9.2 Was die Pipeline uebernehmen sollte

- Tag-Erstellung, zum Beispiel `v2.8.0`
- GitHub Release oder vergleichbare Release-Metadaten
- Build und Packaging
- Artefakt-Publikation
- Deployment oder nachgelagerte Freigabe-Deployments

### 9.3 Was die Pipeline nicht allein entscheiden sollte

- ob die Version inhaltlich release-reif ist
- welcher Scope in die Version gehoert
- ob der Cut heute oder spaeter stattfinden soll

---

## 10. Manueller Fallback ohne Pipeline
[INTENT: ANFORDERUNG]

Wenn keine belastbare Release-Pipeline existiert, muessen die manuellen Schritte trotzdem klar dokumentiert und kontrolliert ablaufen.

### 10.1 Manueller Minimal-Ablauf

#### Release-Branch schneiden

```bash
git fetch --prune origin
git switch develop
git pull --ff-only origin develop
git switch -c release/2.8.0
git push -u origin release/2.8.0
```

#### PR nach `main`

- `release/2.8.0` -> `main`

#### Nach Merge nach `main` kontrolliert taggen

```bash
git fetch origin
git switch main
git pull --ff-only origin main
git tag -a v2.8.0 -m "Release 2.8.0"
git push origin v2.8.0
```

#### Optional GitHub Release manuell erzeugen

Falls keine Pipeline oder Release-Automatisierung existiert, kann ein GitHub Release kontrolliert manuell angelegt werden.

#### Rueck-PR nach `develop`

- `release/2.8.0` -> `develop`

### 10.2 Architekturelle Einordnung des manuellen Fallbacks

- manueller Fallback ist zulaessig
- manueller Fallback ist **nicht** das bevorzugte Zielbild
- manueller Fallback ist nur dann sauber, wenn er kontrolliert, dokumentiert und wiederholbar ist

---

## 11. Wann entsteht ein `support/*`-Branch?
[INTENT: REFERENZ]

Ein `support/<major.minor>`-Branch entsteht nur, wenn die freigegebene Version laenger separat gepflegt werden muss.

Typische Gruende:

- Kunden nutzen eine aeltere Minor-Version weiter
- regulatorische oder vertragliche Support-Zusagen bestehen
- Sicherheits- oder Wartungsfixes fuer eine aeltere Linie bleiben erforderlich

Beispiel:

```bash
git fetch origin
git switch main
git pull --ff-only origin main
git switch -c support/2.8
git push -u origin support/2.8
```

Wichtig:

- `support/*` ist keine allgemeine Entwicklungs- oder Feature-Linie
- `support/*` ist eine kontrollierte Wartungslinie

---

## 12. Schnell-Checklisten
[INTENT: REFERENZ]

### 12.1 Vor Release-Cut

- genuegend PRs sind in `develop` integriert
- Scope fuer die Version ist klar
- Governance-Trigger ist erreicht
- Zielversion ist festgelegt

### 12.2 Vor PR nach `main`

- `release/<semver>` existiert
- nur Stabilisierungsaenderungen sind enthalten
- Freigabepruefungen sind bestanden
- PR-Ziel ist `main`

### 12.3 Nach Merge nach `main`

- Tag wird durch Pipeline oder kontrolliert manuell erzeugt
- Artefakte werden erstellt
- optional GitHub Release wird erzeugt
- Rueck-PR nach `develop` wird erstellt

### 12.4 Bei Support-Bedarf

- Entscheidung fuer `support/<major.minor>` ist dokumentiert
- Startpunkt fuer `support/*` ist freigegebener Stand

---

## 13. Dateipfad-Index
[INTENT: REFERENZ]

Keine Dateien referenziert.

---

## 14. Ausfuehrungskontext fuer LLM-Agents
[INTENT: KONTEXT]

Dieses Dokument ist der operative Release-Leitfaden fuer die Phase **nach** `develop`.

Wichtiger Kontext:

- Der Entwickler-Workflow endet mit PRs nach `develop`.
- Release-Cuts sind Governance-getrieben.
- Die technische Ausfuehrung eines Release-Cuts kann manuell oder automatisiert erfolgen.
- Das Zielbild fuer Tags und Release-Artefakte ist Pipeline-Automatisierung.
- Wenn keine Pipeline vorhanden ist, bleibt ein dokumentierter manueller Fallback zulaessig.
- `support/*` entsteht nur bei echter Wartungsnotwendigkeit.
