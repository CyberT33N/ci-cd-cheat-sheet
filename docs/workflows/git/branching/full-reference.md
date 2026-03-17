# Referenzdokument: Enterprise Max-Governance Branching Architecture
[INTENT: KONTEXT]

---

## 1. Aufgabenuebersicht
[INTENT: KONTEXT]

Dieses Dokument definiert die finale Branching-Architektur fuer eine hochskalierte Enterprise-Max-Governance-Umgebung mit starker Historienintegritaet, klaren Schutzebenen, versionierter Release-Steuerung und GitHub-zentrierter PR-Governance.

Die finale Architekturentscheidung ist:

- **Finale Zielarchitektur:** `Option A - Maximum-control governed release-line model`
- **Finale Gesamtbewertung:** `47%`
- **Enterprise-Zielbild:** maximale Branching-Korrektheit, explizite Integrations- und Release-Linien, stabile Review-Historie, keine implizite Vereinfachung zugunsten von Geschwindigkeit oder geringerem Arbeitsaufwand

Die Architektur dieses Dokuments legt fest:

- welche permanenten und temporaeren Branch-Rollen existieren
- wie `main`, `develop`, `release/*`, `support/*`, `feature/*`, `fix/*`, `docs/*`, `refactor/*`, `chore/*`, `test/*`, `perf/*` und `hotfix/*` fachlich zu verstehen sind
- warum ein offizieller `feature-dev`-Branch **nicht** Teil der Zielarchitektur ist
- warum eine optionale private `scratch/*`-Ebene tolerierbar ist, aber keine offizielle Governance-Schicht bildet
- warum Force Push auf offiziellen `feature/*`-Branches aus Max-Governance-Sicht verboten sein soll
- wie Commit-Historie, Conventional Commits, Review-Fixes, Rebase, Merge und Squash strategisch einzuordnen sind
- wie `release/*`- und `support/*`-Branches einfach und operativ korrekt zu verstehen sind
- wie Hotfixes von der real betroffenen Linie ausgehen und danach gezielt weitergeleitet werden
- wie nach mehreren PR-Merges auf `develop` ein Release-Cut architekturell korrekt erfolgt
- welche Governance-Trigger ein `release/*` ausloesen duerfen und was davon manuell oder automatisiert sein soll
- wie PRs, Tags und Pipeline-Verantwortung im Release-Prozess verteilt sind
- warum andere Branching-Konventionen prozentual schlechter bewertet wurden
- warum der Default fuer Branch-Namen nur **einen** Slash nach dem Prefix verwendet
- wann ein zweiter Slash in sehr grossen Monorepos mit echter Domainen- oder Governance-Bedeutung trotzdem legitim sein kann
- wie GitHub-Rulesets, Branch-Schutz und optionale CI/CD-Pruefungen architekturell einzuordnen sind

---

## 2. Informationsregister (INHALT-Einheiten)
[INTENT: REFERENZ]

| ID | Typ | Beschreibung | Veraenderung | Status |
|----|-----|-------------|-------------|--------|
| REQ-001 | ANFORDERUNG | Finale Auswahl der Max-Governance-Branching-Architektur als `Option A` | Ja | Akzeptiert |
| REQ-002 | ANFORDERUNG | Definition der permanenten und temporaeren Branch-Rollen | Ja | Akzeptiert |
| REQ-003 | ANFORDERUNG | Offizielle Naming-Konvention fuer Arbeitsbranches, Release-Branches und Support-Linien | Ja | Akzeptiert |
| REQ-004 | ANFORDERUNG | Verwerfung eines offiziellen `feature-dev`-Standards bei gleichzeitiger Toleranz einer privaten `scratch/*`-Ebene | Ja | Akzeptiert |
| REQ-005 | ANFORDERUNG | Commit-Historie, Conventional Commits und semantische Commit-Struktur | Ja | Akzeptiert |
| REQ-006 | ANFORDERUNG | Force-Push-Politik, Branch-Schutz und Review-Stabilitaet | Ja | Akzeptiert |
| REQ-007 | ANFORDERUNG | Merge-, Rebase- und Squash-Strategie fuer Arbeits-, Release- und Hotfix-Branches | Ja | Akzeptiert |
| REQ-008 | ANFORDERUNG | Ticket-Zuschnitt, Epic-Aufteilung und AI-era Governance fuer PR-Groesse und Branch-Verantwortung | Ja | Akzeptiert |
| REQ-009 | ANFORDERUNG | Rolle von `release/*`, `support/*` und `hotfix/*` sowie Propagationsregeln | Ja | Akzeptiert |
| REQ-010 | ANFORDERUNG | Prozentuale Bewertung alternativer Branching-Konventionen und Begruendung ihrer Ablehnung | Ja | Akzeptiert |
| REQ-011 | ANFORDERUNG | Default-Slash-Struktur in Branch-Namen und Monorepo-Ausnahmefall | Ja | Akzeptiert |
| REQ-012 | ANFORDERUNG | GitHub-Governance-Modell mit Rulesets als Primaerschutz und CI/CD als Ergaenzung | Ja | Akzeptiert |
| REQ-013 | ANFORDERUNG | Release-Cut nach `develop`, moegliche Governance-Trigger und Release-Reihenfolge | Ja | Akzeptiert |
| REQ-014 | ANFORDERUNG | Verantwortung fuer PRs, Tags, Pipeline-Automatisierung und manuellen Fallback im Release-Prozess | Ja | Akzeptiert |

---

## 3. Informationseinheiten
[INTENT: SPEZIFIKATION]

### 3.1 REQ-001: Finale Auswahl der Max-Governance-Branching-Architektur
[INTENT: SPEZIFIKATION]

**Typ:** ANFORDERUNG

**Beschreibung:**
Die finale Enterprise-Zielarchitektur fuer dieses Dokument ist `Option A - Maximum-control governed release-line model`. Diese Architektur wurde nicht deshalb gewaehlt, weil sie schneller ausliefert oder weniger Komplexitaet besitzt, sondern weil sie unter strikter Governance-Gewichtung die hoechste architekturelle Korrektheit fuer Integrationskontrolle, Release-Trennung, Historienintegritaet, Versionierung und Shared-Branch-Schutz bietet.

Die Architektur basiert auf folgenden Grundsaetzen:

- klare organisatorische Branch-Rollen statt impliziter Mischrollen
- explizite Trennung von Integration, Release-Stabilisierung, Produktion und Support
- keine Umdefinition von `develop` als "Developer-Branch" oder "Beta-Environment-Branch"
- keine branch-lokalen Schattenarchitekturen als Ersatz fuer Governance
- kein Vereinfachungs-Bias zugunsten von Geschwindigkeit, weniger Ebenen oder weniger Arbeitsaufwand

**Ist-Zustand:**
Vor der finalen Entscheidung wurden mehrere Modelle verglichen, darunter:

- `Option A - Maximum-control governed release-line model`
- `Option B - Balanced enterprise default`
- `Option C - Fast-flow trunk model`

Zusaetzlich wurden lokale oder teambezogene Spezialmuster diskutiert, darunter:

- offizieller `feature-dev -> feature -> develop -> main`-Workflow
- globaler Auto-Squash auf allen PRs
- dogmatischer Ein-Commit-Zwang
- Force Push auf offiziellen `feature/*`-Branches

**Soll-Zustand:**
Die Referenzarchitektur dieses Dokuments verwendet final `Option A` mit folgenden Kernrollen:

- `main`
- `develop`
- `release/<semver>`
- `support/<major.minor>` bei realer Mehrversionspflege
- offizielle kurzlebige Arbeitsbranches
- `hotfix/<ticket>-<slug>`

**✅ Positivbeispiel(e):**

```text
Finale Architektur:
main
develop
release/2.8.0
support/2.7
feature/ABC-123-add-export-button
hotfix/ABC-999-payment-timeout
```

**❌ Negativbeispiel(e):**

```text
Finale Architektur:
main
feature-dev
feature
developer
beta
```

Dieses Beispiel ist falsch, weil hier keine saubere Trennung zwischen Integrationslinie, Release-Linie, Produktionslinie und Support-Linie existiert. Mehrere Branch-Namen tragen keine klare Governance-Bedeutung.

---

### 3.2 REQ-002: Definition der permanenten und temporaeren Branch-Rollen
[INTENT: SPEZIFIKATION]

**Typ:** ANFORDERUNG

**Beschreibung:**
Jeder Branch-Typ muss eine klare organisatorische und technische Rolle besitzen. Branches sind in dieser Architektur keine frei austauschbaren Etiketten, sondern Trager von Freigabe-, Review-, Release- oder Support-Semantik.

**Ist-Zustand:**
Es bestand Erklaerungsbedarf dazu, was `main`, `develop`, `release/*`, `support/*`, `feature/*`, `fix/*` und `hotfix/*` jeweils tun. Es wurde ausserdem gefragt, ob `develop` ein "Developer-Branch" oder "Beta-Release-Branch" sei.

**Soll-Zustand:**
Die Branch-Rollen sind wie folgt definiert:

| Branch | Rolle | Lebensdauer | Merge-Herkunft | Merge-Ziel |
|---|---|---|---|---|
| `main` | Produktionswahrheit | dauerhaft | `release/*`, `hotfix/*` | Produktion |
| `develop` | Integrationslinie fuer die naechste Version | dauerhaft | Arbeitsbranches, Forward-Ports | Release-Schnitt |
| `release/<semver>` | Stabilisierung einer konkreten kommenden Version | temporaer | `develop` | `main`, danach zurueck nach `develop` |
| `support/<major.minor>` | Pflege einer bereits veroeffentlichten aelteren Linie | dauerhaft bei Bedarf | `main` oder Release-Cut | Wartungs- und Hotfix-Ziel |
| `feature/<ticket>-<slug>` | offizieller Ticket-Branch fuer neue Features | kurzlebig | `develop` | `develop` |
| `fix/<ticket>-<slug>` | offizieller Ticket-Branch fuer normale Bugfixes | kurzlebig | `develop` | `develop` |
| `docs/<ticket>-<slug>` | offizieller Ticket-Branch fuer Dokumentation | kurzlebig | `develop` | `develop` |
| `refactor/<ticket>-<slug>` | offizieller Ticket-Branch fuer Umbauten | kurzlebig | `develop` | `develop` |
| `chore/<ticket>-<slug>` | offizieller Ticket-Branch fuer Maintenance oder Tooling | kurzlebig | `develop` | `develop` |
| `test/<ticket>-<slug>` | offizieller Ticket-Branch fuer Testarbeit | kurzlebig | `develop` | `develop` |
| `perf/<ticket>-<slug>` | offizieller Ticket-Branch fuer Performance-Themen | kurzlebig | `develop` | `develop` |
| `hotfix/<ticket>-<slug>` | dringender Fix fuer eine real betroffene Linie | kurzlebig | `main`, `release/*` oder `support/*` | betroffene Linie, danach Forward-Port oder Backport |

**✅ Positivbeispiel(e):**

```text
develop = naechste Integrationslinie
release/2.8.0 = Stabilisierung von Version 2.8.0
main = aktuell freigegebene Produktionslinie
support/2.7 = gepflegte aeltere Linie
```

**❌ Negativbeispiel(e):**

```text
develop = Developer-Branch
beta = permanenter Release-Branch
main = allgemeiner Arbeitsbranch
```

Dieses Beispiel ist falsch, weil Branch-Rollen mit Umgebung, Personen oder ungeklaerter Arbeitsnutzung vermischt werden.

---

### 3.3 REQ-003: Offizielle Naming-Konvention fuer Arbeitsbranches, Release-Branches und Support-Linien
[INTENT: SPEZIFIKATION]

**Typ:** ANFORDERUNG

**Beschreibung:**
Die Naming-Konvention muss maschinenfreundlich, reviewfreundlich, gut filterbar und organisatorisch eindeutig sein. Sie soll den Branch-Typ sofort erkennbar machen und gleichzeitig Ticket-ID und Kurzbeschreibung in einem stabilen Muster abbilden.

**Ist-Zustand:**
Es wurden verschiedene Branch-Muster diskutiert, darunter:

- `feature/ABC-123-add-export-button`
- `feat/ABC-123/add-export-button/main`
- `feat/ABC-123/add-export-button/dde`
- `developer`
- `beta`

**Soll-Zustand:**
Die offizielle Branch-Namenskonvention lautet:

- `feature/<ticket>-<slug>`
- `fix/<ticket>-<slug>`
- `docs/<ticket>-<slug>`
- `refactor/<ticket>-<slug>`
- `chore/<ticket>-<slug>`
- `test/<ticket>-<slug>`
- `perf/<ticket>-<slug>`
- `hotfix/<ticket>-<slug>`
- `release/<semver>`
- `support/<major.minor>`

Zusaetzliche Regeln:

- genau ein Slash nach dem Prefix im Default-Fall
- Ticket-ID direkt vor dem Slug
- Slug in `kebab-case`
- kein offizielles `/main`
- kein offizielles `/dde`
- kein Entwicklername im offiziellen Branch-Namen

**✅ Positivbeispiel(e):**

```text
feature/ABC-123-add-export-button
fix/ABC-456-handle-null-customer-id
hotfix/ABC-999-payment-timeout
release/2.8.0
support/2.7
```

**❌ Negativbeispiel(e):**

```text
feat/ABC-123/add-export-button/main
feat/ABC-123/add-export-button/dde
developer
beta
feature/daniel/add-export-button
```

Diese Beispiele sind falsch, weil sie entweder unnoetige Unterebenen, personengebundene Namensanteile oder keine klare Governance-Semantik tragen.

---

### 3.4 REQ-004: Verwerfung eines offiziellen `feature-dev`-Standards bei gleichzeitiger Toleranz einer privaten `scratch/*`-Ebene
[INTENT: SPEZIFIKATION]

**Typ:** ANFORDERUNG

**Beschreibung:**
Ein offizieller `feature-dev`-Standard gehoert nicht zur Zielarchitektur. Er erzeugt keine echte institutionelle Kontrollschicht, sondern verdoppelt nur die ticket-lokale History-Oberflaeche. Eine optionale private `scratch/*`-Ebene kann jedoch toleriert werden, wenn Entwickler vor offizieller Branch-Verantwortung explorieren oder AI-bedingtes Refactoring ausprobieren muessen.

**Ist-Zustand:**
Diskutiert wurde ein Modell, in dem Entwickler:

- auf `feature-dev` frei arbeiten
- dort beliebig committen
- spaeter per Squash auf `feature/*` zurueckfuehren
- `feature/*` als "saubere obere Ebene" behalten

Es wurde ausserdem gefragt, ob diese Spielwiese wegen AI-bedingter Unsicherheit notwendig sei.

**Soll-Zustand:**
Die finale Architektur unterscheidet klar zwischen:

- **offizieller Branching-Konvention:** genau ein offizieller Ticket-Branch
- **optionaler privater Explorationspraxis:** lokale Arbeit oder `scratch/*`

Regeln fuer `scratch/*`:

- nur privat oder kurzlebig
- keine PRs aus `scratch/*`
- keine Team-Governance auf `scratch/*`
- Force Push nur dort optional tolerierbar
- transferierter Stand muss sauber auf den offiziellen `feature/*`-Branch gebracht werden

**✅ Positivbeispiel(e):**

```text
scratch/ABC-123-mssql-pool-registry-exploration
feature/ABC-123-mssql-pool-registry
```

```text
Private Exploration auf scratch/*
-> stabiler Erkenntnisstand
-> sauberer Commit auf feature/*
-> PR aus feature/*
```

**❌ Negativbeispiel(e):**

```text
feature-dev/ABC-123
feature/ABC-123
develop
```

```text
Jedes Ticket muss verpflichtend ueber feature-dev laufen.
PRs duerfen auch aus feature-dev erstellt werden.
```

Diese Beispiele sind falsch, weil `feature-dev` damit zu einer offiziellen Governance-Schicht gemacht wird, obwohl dort keine echte institutionelle Freigabe- oder Integrationsfunktion entsteht.

---

### 3.5 REQ-005: Commit-Historie, Conventional Commits und semantische Commit-Struktur
[INTENT: SPEZIFIKATION]

**Typ:** ANFORDERUNG

**Beschreibung:**
Die Commit-Historie muss semantisch sauber, reviewbar und release-tauglich sein. Die Branching-Architektur verlangt **nicht** exakt einen Commit pro Ticket. Sie verlangt ein koharentes Change-Set mit sinnvoller Historie. Fuer Commit-Messages wird die Conventional-Commit- beziehungsweise Angular-Commit-Grammatik klar empfohlen.

**Ist-Zustand:**
Diskutiert wurden:

- dogmatischer Ein-Commit-Zwang
- mehrere Commits nur bei "extrem hoher" Daseinsberechtigung
- Angular Commit Convention fuer Commits und Branches

**Soll-Zustand:**
Es gelten folgende Regeln:

- Conventional Commits fuer Commit-Messages: **ja**
- Angular-/Conventional-Semantik als komplette Branching-Architektur: **nein**
- ein Branch darf 1 bis 3, in begruendeten Faellen auch 4 semantisch saubere Commits enthalten
- separater Commit fuer Tests, Migrationen, Doku oder klar getrennte Teilaspekte ist legitim
- mehrere ungeordnete WIP-Commits auf dem offiziellen Branch sind unerwuenscht
- Review-Fixes sollen additive, klar benannte Commits sein

**✅ Positivbeispiel(e):**

```text
feat(ABC-123): add MSSQL pool registry
test(ABC-123): cover registry bootstrap behavior
docs(ABC-123): document registry configuration
```

**❌ Negativbeispiel(e):**

```text
wip
fix
tmp
again
final final
```

Dieses Beispiel ist falsch, weil die Commit-Historie keine semantische Bedeutung traegt und Review, Release und Rueckverfolgung unnoetig erschwert.

---

### 3.6 REQ-006: Force-Push-Politik, Branch-Schutz und Review-Stabilitaet
[INTENT: SPEZIFIKATION]

**Typ:** ANFORDERUNG

**Beschreibung:**
Offizielle `feature/*`-Branches sollen aus Max-Governance-Sicht von Anfang an gegen Force Push geschuetzt sein. Der offizielle Ticket-Branch ist ein verantwortetes Arbeits- und Review-Artefakt und darf nicht erst ab PR-Start historisch stabil werden.

**Ist-Zustand:**
Es wurden zwei finale Optionen verglichen:

- `Option A`: Force Push auf offiziellen `feature/*`-Branches generell verbieten
- `Option B`: Force Push technisch zulassen und Verstosse spaeter ueber GitHub Actions oder CI/CD pruefen

**Soll-Zustand:**
Die finale Entscheidung ist `Option A`.

Regeln:

- kein Force Push auf offiziellen `feature/*`-Branches
- kein Force Push auf `develop`, `release/*`, `support/*`, `main`
- private `scratch/*`-Branches duerfen optional umgeschrieben werden
- Review-Fixes werden auf offiziellen Branches als neue Commits geliefert
- GitHub Rulesets bilden den Primaerschutz

**✅ Positivbeispiel(e):**

```text
feature/*:
normal push = erlaubt
force push = verboten

scratch/*:
normal push = erlaubt
force push = optional erlaubt
```

**❌ Negativbeispiel(e):**

```text
feature/*:
force push vor PR erlaubt
force push nach PR nur per Action beanstandet
```

Dieses Beispiel ist schlechter, weil die Governance hauptsaechlich nachgelagert und detektiv arbeitet. Der problematische Rewrite ist bereits passiert, bevor die Automatisierung reagieren kann.

---

### 3.7 REQ-007: Merge-, Rebase- und Squash-Strategie fuer Arbeits-, Release- und Hotfix-Branches
[INTENT: SPEZIFIKATION]

**Typ:** ANFORDERUNG

**Beschreibung:**
Merge-, Rebase- und Squash-Strategien muessen der Branch-Rolle folgen. Die Strategie darf nicht global fuer alle PRs identisch sein. Ziel ist maximale Nachvollziehbarkeit auf Release- und Hotfix-Linien und saubere Integrationshistorie auf `develop`.

**Ist-Zustand:**
Es wurde diskutiert:

- ob globaler Auto-Squash ein Anti-Pattern ist
- ob Review-Fixes immer per `amend` + Force Push behandelt werden sollen
- wann Rebase, Squash oder Merge Commit sinnvoll sind

**Soll-Zustand:**
Es gelten folgende Regeln:

- Rebase nur auf privaten oder kurzlebigen nicht-shared Branches
- kein Rebase auf `develop`, `release/*`, `support/*`, `main`
- kein globaler Auto-Squash
- selektiver Squash nur, wenn internes Rauschen in einen sauberen offiziellen Commit ueberfuehrt wird
- `feature/* -> develop`: saubere Commit-Serie bevorzugt erhalten; `rebase merge` oder normaler Merge sind zulaessig, selektiver Squash nur bei Bedarf
- `release/* -> main`: `merge commit`
- `release/* -> develop`: `merge commit`
- `hotfix/* -> main`, `support/*` oder `release/*`: `merge commit`
- Propagation in weitere Linien per `cherry-pick -x` oder bewusstem Folge-PR

**✅ Positivbeispiel(e):**

```text
feature/* -> develop:
commit series stays semantically clean

release/2.8.0 -> main:
merge commit

hotfix/ABC-999-payment-timeout -> main:
merge commit
```

**❌ Negativbeispiel(e):**

```text
All PRs use auto-squash by default.
release/* is rebased before merge.
hotfix/* gets squashed into one anonymous commit on main.
```

Diese Beispiele sind falsch, weil Release- und Hotfix-Lineage dadurch unnoetig verflacht oder umgeschrieben wird.

---

### 3.8 REQ-008: Ticket-Zuschnitt, Epic-Aufteilung und AI-era Governance fuer PR-Groesse und Branch-Verantwortung
[INTENT: SPEZIFIKATION]

**Typ:** ANFORDERUNG

**Beschreibung:**
AI-unterstuetztes Programmieren erhoeht die Menge geaenderter Dateien, aber nicht die zulaessige Unschaerfe einer PR. Unter Max-Governance muessen Tickets, Epics und PRs so geschnitten werden, dass die Integrations- und Review-Einheit beherrschbar bleibt.

**Ist-Zustand:**
Es bestand die Frage, ob mehrere Commits oder groessere Refactorings auf einem `feature/*`-Branch ein Architekturfehler seien und ob ein offizieller `feature-dev`-Branch aufgrund AI-bedingter Datei-Mengen notwendig werde.

**Soll-Zustand:**
Es gelten folgende Regeln:

- AI ist kein Grund fuer groessere offizielle Branching-Komplexitaet
- grosse Vorhaben muessen in Epics und kleinere Tickets zerlegt werden
- jedes Ticket bildet eine klare Integrations- und Review-Einheit
- mehrere saubere Commits auf einem Ticket-Branch sind normal
- ein zu grosser, unreviewbarer Ticket-Branch ist Governance- und Delivery-Schuld
- gestapelte PRs, Feature Flags und sequentielle Tickets sind zulaessige Gegenmassnahmen

**✅ Positivbeispiel(e):**

```text
Epic: MSSQL integration
- Ticket 1: add pool registry
- Ticket 2: wire registry into module bootstrap
- Ticket 3: update operational documentation
```

**❌ Negativbeispiel(e):**

```text
One ticket:
- add pool registry
- refactor unrelated modules
- update six docs
- change CI config
- rewrite many files with AI
- hide complexity in feature-dev
```

Dieses Beispiel ist falsch, weil mehrere unabhaengige Integrationsentscheidungen in einer einzigen Review- und Branch-Einheit vermischt werden.

---

### 3.9 REQ-009: Rolle von `release/*`, `support/*` und `hotfix/*` sowie Propagationsregeln
[INTENT: SPEZIFIKATION]

**Typ:** ANFORDERUNG

**Beschreibung:**
`release/*`, `support/*` und `hotfix/*` muessen einfach, aber korrekt verstanden werden. `release/*` stabilisiert die kommende Version, `support/*` pflegt eine bereits veroeffentlichte aeltere Linie, und `hotfix/*` entsteht immer von der real betroffenen Linie.

**Ist-Zustand:**
Es bestand Erklaerungsbedarf dazu, was `release/*` und `support/*` konkret tun und wie Hotfixes auf Basis der neuen Konvention erstellt werden muessen.

**Soll-Zustand:**
Definitionen:

- `release/2.8.0`: kommende Version 2.8.0 wird eingefroren und stabilisiert
- `support/2.7`: bereits freigegebene Linie 2.7 wird weiterhin gepflegt
- `hotfix/<ticket>-<slug>`: Branch fuer dringenden Fix auf der Linie, die den Fehler wirklich traegt

Propagationsregeln:

- Hotfix entsteht von `main`, `release/*` oder `support/*`, je nachdem wo der Fehler tatsaechlich sitzt
- nach dem Merge in die betroffene Linie wird der Fix in alle weiteren aktiven Linien bewusst weitergeleitet
- keine stillschweigende Annahme, dass `develop` alles automatisch korrekt enthaelt

**✅ Positivbeispiel(e):**

```text
Bug exists in production 2.7
-> create hotfix from support/2.7
-> merge back into support/2.7
-> forward-port to develop if upcoming versions also need the fix
```

**❌ Negativbeispiel(e):**

```text
Production bug exists on support/2.7
-> create hotfix from develop
-> merge to develop
-> assume support and main will be fine later
```

Dieses Beispiel ist falsch, weil der Hotfix nicht von der real betroffenen Linie ausgeht und die Produktionsprovenance dadurch unscharf wird.

---

### 3.10 REQ-010: Prozentuale Bewertung alternativer Branching-Konventionen und Begruendung ihrer Ablehnung
[INTENT: SPEZIFIKATION]

**Typ:** ANFORDERUNG

**Beschreibung:**
Die Zielarchitektur muss auch dokumentieren, welche Alternativen bewertet wurden, wie hoch ihre prozentuale Eignung ausfiel und warum sie final nicht gewaehlt wurden. Diese Dokumentation ist Teil der Referenzgrundlage und erklaert die architektonische Ablehnung schlechter bewerteter Muster.

**Ist-Zustand:**
Mehrere Thematiken wurden explizit prozentual gegeneinander bewertet.

**Soll-Zustand:**
Die folgenden Bewertungen gelten als Referenz:

| Thema | Bewertung | Ergebnis |
|---|---:|---|
| `Option A - Maximum-control governed release-line model` | 47% | final gewaehlt |
| `Option B - Balanced enterprise default` | 37% | stark, aber nicht final |
| `Option C - Fast-flow trunk model` | 16% | nicht passend fuer Max-Governance |
| `main` als oberste Linie | 97% | final |
| `develop` als permanenter Integrationszweig in Option A | 89% | final |
| `release/<semver>` | 96% | final |
| `support/<major.minor>` bei Mehrversionsbetrieb | 93% | final |
| `feature/<ticket>-<slug>` | 94% | final |
| offizieller `feature-dev`-Standard | 9% | abgelehnt |
| offizieller `feature-dev-dev`-Standard | 1% | abgelehnt |
| `developer` als Branch-Name | 5% | abgelehnt |
| `beta` als permanenter Branch-Name | 19% | abgelehnt |
| globaler Auto-Squash | 21% | abgelehnt |
| dogmatisches "immer exakt ein Commit" | 27% | abgelehnt |
| Force Push auf offiziellem `feature/*` nach PR-Start | 12% | abgelehnt |
| Force Push auf offiziellem `feature/*` generell | 9% bis 12% | abgelehnt |
| private `scratch/*`-Ebene | 52% bis 56% als tolerierte Praxis | toleriert, aber nicht offizieller Workflow-Kern |

**✅ Positivbeispiel(e):**

```text
Finale Auswahl:
Option A wins because it creates the strongest explicit control boundaries.
Lower-rated patterns remain documented as rejected alternatives.
```

**❌ Negativbeispiel(e):**

```text
Choose the lower-rated pattern because it feels easier or faster.
Remove rejected patterns from the reference because they are not final.
```

Dieses Beispiel ist falsch, weil die Referenzdokumentation dann weder die Auswahlbegruendung noch die Bewertung verworfener Alternativen vollstaendig abbildet.

---

### 3.11 REQ-011: Default-Slash-Struktur in Branch-Namen und Monorepo-Ausnahmefall
[INTENT: SPEZIFIKATION]

**Typ:** ANFORDERUNG

**Beschreibung:**
Der Default fuer Branch-Namen verwendet nach dem Prefix genau **einen** Slash. Ticket-ID und Branch-Beschreibung werden in einem gemeinsamen Segment gefuehrt. Ein zweiter Slash ist nur zulaessig, wenn er echte Governance-Bedeutung traegt, zum Beispiel Domainen-, Produkt- oder Bounded-Context-Semantik in einem grossen Monorepo.

**Ist-Zustand:**
Es wurde gefragt, warum Ticketnummer und Branch-Beschreibung nicht noch einmal mit einem Slash getrennt werden und ob ein zusaetzlicher Slash in einem Monorepo sinnvoller sei.

**Soll-Zustand:**
Default:

- `feature/ABC-123-add-export-button`

Monorepo-Ausnahme nur bei echter Bedeutung:

- `feature/payments/ABC-123-add-export-button`

Nicht zulaessig ist ein zusaetzlicher Slash nur aus optischen Gruenden.

**✅ Positivbeispiel(e):**

```text
Default:
feature/ABC-123-add-export-button

Monorepo with real governance meaning:
feature/payments/ABC-123-add-export-button
```

**❌ Negativbeispiel(e):**

```text
feature/ABC-123/add-export-button
```

Dieses Beispiel ist im Default-Fall falsch, weil der zusaetzliche Slash keine neue Governance-Information traegt und nur die Branch-Tiefe erhoeht.

---

### 3.12 REQ-012: GitHub-Governance-Modell mit Rulesets als Primaerschutz und CI/CD als Ergaenzung
[INTENT: SPEZIFIKATION]

**Typ:** ANFORDERUNG

**Beschreibung:**
Die technische Durchsetzung der Branching-Architektur soll auf GitHub primar ueber rollenbasierte Rulesets erfolgen. CI/CD oder GitHub Actions sind nur Ergaenzungen fuer Policy-Checks und Dokumentationsdurchsetzung, nicht die erste Schutzschicht gegen historisch problematische Aktionen.

**Ist-Zustand:**
Es wurde gefragt, ob GitHub einen offiziellen `feature/*`-Branch erst bei PR-Erstellung einfrieren kann und ob GitHub Actions oder CI/CD diese Aufgabe uebernehmen sollen.

**Soll-Zustand:**
Die finale Regel lautet:

- Schutz durch Branch-Rolle, nicht durch PR-Zustand
- `feature/*` von Anfang an ohne Force Push
- `scratch/*` optional mit Rewrite-Freiheit
- `develop`, `release/*`, `support/*`, `main` strikt geschuetzt
- GitHub Rulesets oder Branch-Protection-Patterns bilden den Primaerschutz
- Actions duerfen zusaetzlich pruefen, dass keine PRs aus `scratch/*` erstellt werden
- Actions duerfen Naming, Ticket-Praefixe oder andere Workflow-Verstoesse markieren

**✅ Positivbeispiel(e):**

```text
GitHub ruleset:
feature/* => push allowed, force push denied
scratch/* => push allowed, force push optionally allowed
main/develop/release/*/support/* => protected, no direct push, no force push
```

**❌ Negativbeispiel(e):**

```text
No branch ruleset.
Allow force push on feature/*.
Use CI only to detect the problem after a forced update already happened.
```

Dieses Beispiel ist falsch, weil die Sicherheitslogik hauptsaechlich nachgelagert und nicht praeventiv ist.

---

### 3.13 REQ-013: Release-Cut nach `develop`, moegliche Governance-Trigger und Release-Reihenfolge
[INTENT: SPEZIFIKATION]

**Typ:** ANFORDERUNG

**Beschreibung:**
Nach normalen Ticket-PRs endet der Entwickler-Workflow auf `develop`. Ab dort beginnt Release-Management. `develop` sammelt freigegebene Arbeit fuer die naechste Version. Ein `release/<semver>` wird nicht nach jedem Merge automatisch erzeugt, sondern nur dann, wenn ein definierter Governance-Trigger den Scope fuer eine konkrete Version einfriert.

**Ist-Zustand:**
Es bestand die Frage, ob nach ein oder mehreren PRs auf `develop` sofort ein `release/*`-Branch entsteht und ob diese Entscheidung manuell oder programmatisch erfolgt.

**Soll-Zustand:**
Die korrekte Reihenfolge lautet:

1. mehrere `feature/*`, `fix/*` oder andere Arbeitsbranches werden nach `develop` gemerged
2. `develop` repraesentiert dadurch den integrierten Stand fuer die naechste Version
3. ein definierter Governance-Trigger loest den Release-Cut aus
4. aus `develop` wird `release/<semver>` erzeugt
5. auf `release/<semver>` erfolgen nur noch Stabilisierung, Versionspflege, letzte Fixes und Freigabevorbereitung
6. `release/<semver>` geht per PR nach `main`
7. nach erfolgreicher Freigabe wird `release/<semver>` zurueck nach `develop` gefuehrt
8. wenn die Version laenger gepflegt werden muss, kann daraus oder vom freigegebenen Stand zusaetzlich `support/<major.minor>` entstehen

Zulaessige Governance-Trigger fuer einen Release-Cut:

- **teamuebergreifende manuelle Freigabeentscheidung**
- **kalender- oder release-train-basierter Zeittrigger**
- **policy-, milestone- oder governance-basierter Trigger**

Wichtige architekturelle Regel:

- nicht jeder Merge nach `develop` erzeugt ein Release
- CI/CD darf die technische Ausfuehrung automatisieren
- die Entscheidung **wann** und **welcher Scope** als Release geschnitten wird, bleibt eine Governance-Frage

**✅ Positivbeispiel(e):**

```text
feature/* -> develop
fix/* -> develop
docs/* -> develop

Governance decision:
Cut release/2.8.0 from develop

release/2.8.0 -> main
release/2.8.0 -> develop
```

```text
Scheduled release train:
Every Wednesday 12:00 a pre-approved release cut may create release/<semver> from develop.
The schedule is governance-defined first, then executed automatically.
```

**❌ Negativbeispiel(e):**

```text
Every merge to develop automatically creates a new release branch.
```

Dieses Beispiel ist falsch, weil damit `develop` faktisch zu einer sofortigen Release-Linie umdefiniert wird und kein bewusster Scope-Freeze mehr existiert.

---

### 3.14 REQ-014: Verantwortung fuer PRs, Tags, Pipeline-Automatisierung und manuellen Fallback im Release-Prozess
[INTENT: SPEZIFIKATION]

**Typ:** ANFORDERUNG

**Beschreibung:**
Im Release-Prozess muessen Governance-Entscheidungen, Pull Requests, Tags und Pipeline-Verantwortung klar getrennt sein. Zielarchitektur ist: Menschen oder formale Regeln entscheiden den Release-Cut; CI/CD fuehrt die technische Freigabe soweit wie moeglich automatisiert aus.

**Ist-Zustand:**
Es bestand die Frage, ob Pull Requests fuer Releases erstellt werden, ob Tags besser von der Pipeline statt manuell gesetzt werden und wie zu verfahren ist, wenn keine Pipeline vorhanden ist.

**Soll-Zustand:**
Es gelten folgende Regeln:

- der Release-Cut selbst ist eine Governance-Entscheidung
- `release/<semver>` wird aus `develop` erzeugt
- der Uebergang `release/<semver> -> main` erfolgt per PR
- der Rueckfluss `release/<semver> -> develop` erfolgt ebenfalls per PR
- Tags und technische Release-Artefakte sollen **im Zielbild** durch die Pipeline erzeugt werden
- manuelles Tagging ist kein ideales Zielbild, aber ein zulaessiger Fallback, wenn keine belastbare Release-Pipeline existiert
- wenn manuell gearbeitet wird, muss der manuelle Ablauf explizit dokumentiert und kontrolliert sein

Zielbild fuer Tags:

- Merge von `release/<semver>` nach `main`
- Pipeline validiert den Merge-Zustand
- Pipeline erstellt den Tag, zum Beispiel `v2.8.0`
- Pipeline erzeugt Artefakte, GitHub Release, Deployment oder weitere Release-Metadaten

Fallback ohne Pipeline:

- PR nach `main`
- Merge nach Freigabe
- kontrolliertes manuelles Tagging auf `main`
- optional kontrolliertes manuelles Erstellen eines GitHub Releases
- Rueck-PR nach `develop`

Wichtige architekturelle Einordnung:

- manuelles Tagging ist **nicht automatisch** ein Anti-Pattern, wenn keine Pipeline existiert
- ein dauerhaft manuell betriebener Release-Prozess ist aber architekturell schwaecher als eine belastbare automatisierte Pipeline
- die Pipeline soll die technische Freigabe uebernehmen, nicht die inhaltliche Scope-Entscheidung

**✅ Positivbeispiel(e):**

```text
Target architecture:
Governance approves release cut
-> release/2.8.0 is created from develop
-> PR release/2.8.0 -> main
-> pipeline creates tag v2.8.0 and release artifacts
-> PR release/2.8.0 -> develop
```

```text
Fallback without pipeline:
PR release/2.8.0 -> main
merge after approval
git tag -a v2.8.0 -m "Release 2.8.0"
git push origin v2.8.0
PR release/2.8.0 -> develop
```

**❌ Negativbeispiel(e):**

```text
Release manager creates random local tag before the release PR is merged and without a controlled process.
```

Dieses Beispiel ist falsch, weil Tag, Freigabestand und kontrollierter Merge-Zeitpunkt nicht mehr sauber gekoppelt sind.

---

## 4. Konventionen & Constraints
[INTENT: CONSTRAINT]

Die folgenden Konventionen und Constraints gelten fuer die gesamte Zielarchitektur:

- `Option A` ist die finale Branching-Architektur.
- `main` ist die oberste Produktionslinie.
- `develop` ist die Integrationslinie fuer die naechste Version.
- `release/<semver>` ist die Stabilisierungslinie fuer eine konkrete kommende Version.
- `support/<major.minor>` existiert nur bei echter Mehrversionspflege.
- nicht jeder Merge nach `develop` erzeugt automatisch einen `release/*`-Branch.
- ein `release/<semver>` entsteht nur bei bewusstem Governance-Trigger.
- zulaessige Trigger sind teamuebergreifende Freigabe, kalender- oder train-basierte Zeitregel oder policy-/milestone-basierter Governance-Trigger.
- `release/<semver>` wird aus `develop` erzeugt und nicht durch normalen Entwickler-PR-Verkehr zufaellig aufgebaut.
- `release/<semver>` geht per PR nach `main` und danach per PR zurueck nach `develop`.
- Tags sollen im Zielbild von der Pipeline erstellt werden.
- wenn keine Release-Pipeline existiert, muessen manuelle Tag- und Freigabeschritte explizit dokumentiert und kontrolliert erfolgen.
- Es gibt genau **einen offiziellen Ticket-Branch pro Ticket**.
- Ein offizieller `feature-dev`-Standard ist nicht Teil der Architektur.
- Eine private `scratch/*`-Praxis kann toleriert werden, ist aber nicht offizieller Workflow-Kern.
- Offizielle Arbeitsbranches verwenden die Prefixe `feature`, `fix`, `docs`, `refactor`, `chore`, `test`, `perf`.
- Dringende produktionsnahe Fixes verwenden `hotfix/<ticket>-<slug>`.
- Ticket-ID und Beschreibung teilen sich im Default-Fall **ein** Segment nach dem Prefix.
- Ein zusaetzlicher Slash ist nur bei echter Monorepo- oder Domainenbedeutung zulaessig.
- Commit-Messages folgen Conventional Commits / Angular-Style fuer Commits.
- Branching selbst folgt **nicht** 1:1 der Angular-/Conventional-Typologie.
- Es gibt keinen dogmatischen Ein-Commit-Zwang.
- Offizielle `feature/*`-Branches sind gegen Force Push geschuetzt.
- Review-Fixes werden als neue Commits geliefert.
- Rebase ist nur auf privaten oder kurzlebigen nicht-shared Branches zulaessig.
- Release- und Hotfix-Lineage wird nicht durch globalen Auto-Squash verflacht.
- GitHub Rulesets sind die primaere Schutzschicht; CI/CD ist ergaenzend.

---

## 5. Dateipfad-Index
[INTENT: REFERENZ]

Keine Dateien referenziert.

---

## 6. Ausfuehrungskontext fuer LLM-Agents
[INTENT: KONTEXT]

Dieses Dokument ist als eigenstaendige Referenz fuer LLM-Agents und fuer menschliche Architekturentscheidungen verwendbar.

Relevanter Ausfuehrungskontext:

- Die Zielplattform fuer Governance ist primaer GitHub.
- Das Zielbild ist **Max-Governance**, nicht "balanced default" und nicht "fast flow".
- Die Architektur geht von PR-/MR-basierter Integration aus.
- Der Weg nach `develop` wird durch einen separaten Release-Cut in `release/<semver>` fortgesetzt, sobald ein definierter Governance-Trigger aktiv wird.
- Release-Cuts koennen manuell freigegeben, kalenderbasiert oder policy-/milestone-basiert ausgelost werden; die technische Ausfuehrung kann danach durch CI/CD automatisiert erfolgen.
- Das Zielbild fuer Tags und Release-Artefakte ist Pipeline-Automatisierung; manuelle Schritte sind ein dokumentierter Fallback, kein bevorzugter Zielzustand.
- AI-unterstuetztes Programmieren erhoeht die Notwendigkeit fuer kleinere Tickets, kleinere PRs und klarere Branch-Rollen.
- Die Dokumentation umfasst bewusst auch prozentual schlechter bewertete Alternativen, weil ihre Ablehnung Teil der Referenzbegruendung ist.
- Der Default fuer Branch-Namen ist fuer Standard-Repositories optimiert; in grossen Monorepos darf ein zusaetzlicher Pfadteil nur bei echter Domainen- oder Governance-Bedeutung eingefuehrt werden.
- Alle notwendigen Informationen sind in den Sektionen 1-5 vollstaendig enthalten.
