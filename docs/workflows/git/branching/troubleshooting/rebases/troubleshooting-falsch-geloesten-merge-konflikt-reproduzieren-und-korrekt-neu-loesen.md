# Troubleshooting — Falsch gelösten Merge-Konflikt reproduzieren und korrekt neu lösen

## Zweck

Dieses Cheat-Sheet beschreibt, was du tun musst, wenn du vermutest oder weißt, dass ein **Merge- oder Rebase-Konflikt falsch gelöst** wurde und du den ursprünglichen Konflikt **noch einmal sichtbar machen** willst, um ihn sauber neu zu lösen.

Der Fokus liegt auf diesem sicheren Standardpfad:

- den **aktuellen Arbeitsbranch nicht kaputtmachen**
- den **Stand vor dem Rebase** identifizieren
- einen **temporären Branch** oder eine **separate Worktree** verwenden
- den Konflikt erneut erzeugen
- die **betroffene Datei** gezielt vergleichen, übernehmen und korrekt in den Feature-Branch zurückbringen

---

## Typisches Problembild

Du hattest einen Konflikt beim `merge` oder `rebase`.
Du hast ihn aufgelöst, später merkst du aber:

- die Lösung war vermutlich falsch
- ein bestimmter Teil aus `develop` fehlt
- oder deine eigene Änderung wurde falsch übernommen

Dann ist das Ziel **nicht**, blind irgendetwas im aktuellen Branch zu überschreiben.

Das Ziel ist:

1. den **historischen Konfliktzustand** wiederherstellen
2. den Konflikt **noch einmal sichtbar machen**
3. die Datei **korrekt** neu zusammenführen

---

## Wichtige Grundregel

### Nur die Datei zurücksetzen reicht meistens nicht

Ein Konflikt hängt **nicht nur am Dateiinhalt**, sondern an:

- dem **gesamten Branchzustand**
- dem **Rebase- oder Merge-Ziel**
- dem **Base-Commit**
- und der Frage, welche Commits Git gerade neu anwendet

Deshalb gilt:

- **Nur Datei zurücksetzen** → stellt nur Inhalt zurück
- **Konflikt reproduzieren** → stellt das **gesamte historische Git-Szenario** wieder her

---

## Wann dieses Troubleshooting-Dokument benutzt werden sollte

Nutze diesen Ablauf, wenn mindestens einer dieser Fälle zutrifft:

- du **weißt**, dass du einen Konflikt falsch gelöst hast
- du **vermutest**, dass deine Auflösung fachlich falsch war
- du bekommst den Konflikt **nicht mehr angezeigt**, obwohl du ihn noch einmal sehen willst
- du willst die **Datei sauber aus dem Konflikt heraus neu übernehmen**

---

# Standardvorgehen

## 1) Den Vor-Rebase- oder Vor-Merge-Stand identifizieren

Zuerst musst du herausfinden, **welcher Commitstand vor dem problematischen Rebase oder Merge** aktiv war.

### Reflog prüfen

```powershell
git reflog --date=local
```

Suche dort nach Einträgen wie:

- `rebase (start)`
- `rebase (finish)`
- `merge`
- `checkout`
- `reset`

Der relevante Punkt ist normalerweise:

- der **Feature-Branch-Stand direkt vor dem Rebase**
- und der **damalige develop-Stand**, gegen den rebased wurde

### Beispiel-Denkschema

- `<pre_rebase_feature_sha>` = Feature-Branch vor dem Rebase
- `<target_develop_sha>` = damaliger `develop`-Stand

Diese beiden SHAs sind der Schlüssel, um denselben Konflikt wiederherzustellen.

---

## 2) Nicht auf dem echten Feature-Branch testen

Wenn du das Konfliktszenario reproduzieren willst, solltest du **nicht direkt auf dem aktiven Branch** testen.

### Warum?

Weil du sonst:

- deinen aktuellen Arbeitsstand verschiebst
- weitere History-Operationen auf dem echten Branch machst
- und die Analyse unnötig riskant wird

Der richtige Standard ist deshalb:

- **temporären Branch** erzeugen
- oder noch besser: **separate Worktree** benutzen

---

# Variante A — Temporären Branch verwenden

## Ziel

Das historische Konfliktszenario wieder sichtbar machen, ohne den echten Feature-Branch zu beschädigen.

### Befehl

```powershell
git switch -c tmp/reproduce-conflict <pre_rebase_feature_sha>
```

Beispiel:

```powershell
git switch -c tmp/reproduce-findings-conflict dc8e9db7b
```

Damit hast du einen neuen Branch auf genau dem Stand, auf dem dein Feature-Branch **vor dem Rebase** war.

---

## 3) Optional `rerere` deaktivieren

Git kann frühere Konfliktauflösungen wiederverwenden.
Das kann dazu führen, dass ein früherer Konflikt **nicht mehr offen angezeigt** wird.

### Prüfen

```powershell
git config --get rerere.enabled
```

### Für den Reproduktionstest deaktivieren

```powershell
git config rerere.enabled false
```

Das hilft, damit Git den Konflikt nicht stillschweigend wiederverwendet.

---

## 4) Den Rebase gegen denselben historischen `develop`-Stand erneut ausführen

Wenn du den historischen Konflikt möglichst exakt wiederhaben willst, rebase **nicht einfach gegen den heutigen `origin/develop`**, sondern gegen den damaligen Zielstand.

### Befehl

```powershell
git rebase <target_develop_sha>
```

Beispiel:

```powershell
git rebase d336cdd01
```

Wenn du nicht den historischen Konflikt, sondern einen **aktuellen Konflikt gegen den heutigen Remote-Stand** sehen willst:

```powershell
git fetch origin
git rebase origin/develop
```

### Merksatz

- `git rebase <historischer_sha>` = historisches Szenario nachbauen
- `git rebase origin/develop` = aktuelles Szenario gegen heutigen `develop`

---

## 5) VS Code öffnen und Konflikt ansehen

Wenn der Konflikt entsteht, öffne das Repository in VS Code:

```powershell
code .
```

Dann siehst du die konfliktierte Datei direkt im Editor oder Merge-Editor.

---

# Variante B — Separate Worktree verwenden

## Warum diese Variante oft besser ist

Eine separate Worktree ist noch sauberer als ein temporärer Branch im selben Arbeitsverzeichnis, weil du:

- den aktuellen Branch unverändert lässt
- die Reproduktion in einem eigenen Ordner laufen lässt
- und die Konfliktlösung isoliert prüfen kannst

### Worktree anlegen

```powershell
git worktree add ..\privyou-rebase-repro -b tmp/reproduce-conflict <pre_rebase_feature_sha>
```

Beispiel:

```powershell
git worktree add ..\privyou-rebase-repro -b tmp/reproduce-findings-conflict dc8e9db7b
```

### In die Worktree wechseln

```powershell
cd ..\privyou-rebase-repro
```

### `rerere` optional deaktivieren

```powershell
git config rerere.enabled false
```

### Rebase erneut ausführen

```powershell
git rebase d336cdd01
```

### Worktree in VS Code öffnen

```powershell
code ..\privyou-rebase-repro
```

---

# Einzelne Datei gezielt prüfen

Wenn der Konflikt sichtbar ist und du speziell nur eine Datei sauber untersuchen willst, dann vergleiche genau diese Datei.

Beispiel mit [`findings.ts`](src/modules/pvs/z1/findings.ts):

```powershell
git --no-pager diff -- src/modules/pvs/z1/findings.ts
```

Wenn du die Unterschiede zu `develop` für genau diese Datei sehen willst:

```powershell
git fetch origin
git --no-pager diff origin/develop...HEAD -- src/modules/pvs/z1/findings.ts
```

---

# Wenn der Konflikt korrekt reproduziert wurde

Dann gibt es drei sinnvolle nächste Schritte:

## A) Datei im Konflikt manuell korrekt zusammenführen

Das ist der Standardfall.
Du nimmst:

- die nötigen Änderungen aus `develop`
- deine eigenen Änderungen aus dem Feature-Branch
- und baust daraus die fachlich richtige Endversion

Danach:

```powershell
git add src/modules/pvs/z1/findings.ts
git rebase --continue
```

---

## B) Dateiinhalt aus dem reproduzierten Konfliktszenario herauskopieren

Wenn du in VS Code den Konflikt sichtbar hast und die Datei dort manuell richtig gebaut hast, kannst du diese finale Version anschließend in den eigentlichen Feature-Branch übernehmen.

Der sichere Weg ist:

1. Konflikt in temporärem Branch oder Worktree sauber lösen
2. Dateiinhalt prüfen
3. Datei in den echten Branch übertragen
4. dort committen

---

## C) Nur eine Datei aus einem historischen Commit zurückholen

Wenn du gar nicht den ganzen Konflikt erneut brauchst, sondern nur einen alten Dateistand, dann kannst du die Datei direkt aus einem historischen Commit holen.

Beispiel:

```powershell
git restore --source=<pre_rebase_feature_sha> -- src/modules/pvs/z1/findings.ts
```

Das ist aber **nicht** die Konfliktreproduktion, sondern nur eine Dateiwiederherstellung.

---

# Was tun, wenn der Konflikt nicht wieder erscheint?

Wenn du den temporären Branch gebaut hast und der Konflikt trotzdem nicht wieder kommt, prüfe diese Ursachen:

## 1) Falscher Ausgangs-Commit

Du hast nicht den echten Vor-Rebase-Feature-Stand genommen.

## 2) Falscher Ziel-Commit

Du rebast gegen den heutigen `origin/develop` statt gegen den damaligen historischen Zielstand.

## 3) `rerere` verwendet frühere Konfliktlösung wieder

Dann Konfliktreproduktion erneut mit deaktiviertem `rerere` testen.

## 4) Du hast nur die Datei zurückgesetzt, aber nicht den ganzen Branchzustand

Dann fehlt der eigentliche Rebase-Kontext.

---

# Minimaler reproduzierbarer Ablauf

Wenn du den Konflikt schnell wiederhaben willst, ist das der Kernablauf:

```powershell
git reflog --date=local
git switch -c tmp/reproduce-conflict <pre_rebase_feature_sha>
git config rerere.enabled false
git rebase <target_develop_sha>
code .
```

---

# Beispiel für eine einzelne Datei mit temporärem Branch

## Ziel

Einen falsch gelösten Konflikt für [`findings.ts`](src/modules/pvs/z1/findings.ts) erneut sichtbar machen.

## Beispielablauf

```powershell
git switch -c tmp/reproduce-findings-conflict dc8e9db7b
git config rerere.enabled false
git rebase d336cdd01
code .
```

## Danach gezielt nur die Datei ansehen

```powershell
git --no-pager diff -- src/modules/pvs/z1/findings.ts
```

## Unterschiede zum aktuellen Remote-`develop` prüfen

```powershell
git fetch origin
git --no-pager diff origin/develop...HEAD -- src/modules/pvs/z1/findings.ts
```

---

# Beispiel mit einzelner Datei auschecken

Wenn du eine Datei aus einem historischen Stand gezielt zurückholen willst:

```powershell
git restore --source=dc8e9db7b -- src/modules/pvs/z1/findings.ts
```

Wenn du die Datei aus einem Konflikttest-Branch in deinen echten Branch übernehmen willst, ist das konzeptionell etwas anderes:

- erst Konflikt korrekt lösen
- dann Datei in den eigentlichen Branch übernehmen

---

# Entscheidungshilfe

## Nur Datei kaputt, aber Konflikt nicht wichtig?

Dann reicht oft:

```powershell
git restore --source=<historischer_sha> -- <datei>
```

## Konflikt wirklich noch einmal sehen und korrekt lösen?

Dann brauchst du:

- **temporären Branch oder Worktree**
- **Vor-Rebase-Stand**
- **denselben Rebase-Zielstand**

---

# Best Practice

## Was du nicht tun solltest

- nicht direkt auf dem echten Feature-Branch experimentieren
- nicht blind `reset --hard` auf dem Arbeitsbranch machen
- nicht nur die Datei zurücksetzen und erwarten, dass der Konflikt dadurch automatisch wiederkommt

## Was du tun solltest

- Reflog prüfen
- Vor-Rebase-Stand exakt bestimmen
- Testbranch oder Worktree erzeugen
- historischen Rebase erneut ausführen
- Konflikt in VS Code lösen
- finale Datei gezielt übernehmen

---

# Cheat-Sheet-Kurzfassung

## Wenn du glaubst, dass ein Merge-Konflikt falsch gelöst wurde

1. `git reflog --date=local`
2. Vor-Rebase-Feature-SHA finden
3. historischen `develop`-Ziel-SHA finden
4. temporären Branch oder Worktree erzeugen
5. optional `git config rerere.enabled false`
6. Rebase erneut ausführen
7. Konflikt in VS Code prüfen und korrekt lösen
8. finale Datei in den echten Branch übernehmen

---

# Schnellbefehle

## Temporärer Branch

```powershell
git switch -c tmp/reproduce-conflict <pre_rebase_feature_sha>
git config rerere.enabled false
git rebase <target_develop_sha>
code .
```

## Separate Worktree

```powershell
git worktree add ..\privyou-rebase-repro -b tmp/reproduce-conflict <pre_rebase_feature_sha>
cd ..\privyou-rebase-repro
git config rerere.enabled false
git rebase <target_develop_sha>
code ..\privyou-rebase-repro
```

## Einzeldatei-Diff

```powershell
git --no-pager diff -- src/modules/pvs/z1/findings.ts
```

## Diff gegen Remote-`develop`

```powershell
git fetch origin
git --no-pager diff origin/develop...HEAD -- src/modules/pvs/z1/findings.ts
```

## Einzeldatei aus historischem Stand wiederherstellen

```powershell
git restore --source=<historischer_sha> -- src/modules/pvs/z1/findings.ts
```

---

# Abschlussregel

**Ein falsch gelöster Merge-Konflikt wird sauber reproduziert, indem du nicht nur die Datei, sondern den gesamten historischen Rebase-Kontext wiederherstellst — am besten auf einem temporären Branch oder in einer separaten Worktree.**
