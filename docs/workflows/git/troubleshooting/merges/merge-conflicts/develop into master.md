

### Architekturell “richtige” Reihenfolge (Develop → Master nach langer Zeit)
**Master bleibt “Release‑Source‑of‑Truth”.** Deshalb ist die robuste Reihenfolge:

- **Nicht direkt im PR‑UI “Merge” drücken**, solange Konflikte existieren.
- **Konflikt lokal auf einem Integrations‑Branch lösen**, *dort* bauen/testen, dann **PR Integrations‑Branch → master** mergen.
- Danach **master zurück nach develop mergen**, damit du nicht beim nächsten Mal wieder dieselbe Historien‑Divergenz hast.

---

### Konkretes Vorgehen (so “musst” du agieren)

#### 1) Integrations‑Branch von `master` erstellen
- Von `master` aus einen Branch erstellen, z. B. `merge/develop-into-master-2026-02-05`.

```shell
git checkout master
git pull
git branch merge/develop-into-master-2026-02-05
git checkout merge/develop-into-master-2026-02-05
```

#### 2) `develop` in diesen Branch mergen (lokal)
- Merge ausführen, Konflikt in `package.json` erscheint.

```shell
git checkout develop
git pull
git checkout merge/develop-into-master-2026-02-05
git merge develop
```

#### 3) Konflikt‑Regel für `package.json` (wichtigster Teil)
Behandle `package.json` logisch als **zwei Themen**:

- **A) Inhaltliche Änderungen** (dependencies/devDependencies/scripts/config):  
  → In der Regel **die “echte” Arbeitsbasis aus `develop` übernehmen**, weil dort Features/Fixes entstanden sind.
- **B) Release‑Version** (`version`):  
  → **Master bestimmt die finale Release‑Version**, weil `master` eure “Haupt‑Release”-Linie ist.

> Wichtig: Die `version`-Zeile ist oft der einzige “echte” Konfliktauslöser. **Hier bewusst entscheiden**, nicht “irgendeine Seite gewinnen lassen”.

---

#### 4) Versionierung (bitte ausklappen)

<details>
<summary><strong>Fall A: Sem-Versionierung auf <code>develop</code> mit <code>-beta</code> / <code>-rc</code> + sauberes CI/CD (du musst dich nicht kümmern)</strong></summary>

- **Prinzip**: Auf `develop` laufen Pre‑Releases, z. B. `1.2.4-beta.3` oder `1.2.4-rc.1`.  
  Auf `master` entsteht daraus die **stabile Release-Version** `1.2.4` (ohne Suffix).
- **Was du manuell machst**:
  - Im Merge-Konflikt sicherstellen, dass inhaltlich die richtige `package.json` (Dependencies/Scripts) drin ist.
  - **Keine manuelle Versionsakrobatik**, weil die Release-Version (ohne Pre‑Release‑Suffix) **durch CI/CD** erzeugt wird (z. B. Tagging/Release Job).
- **Warum kein Konflikt-Dauerzustand entsteht**:
  - `develop` hat *andere* Versionsstrings (mit `-beta` / `-rc`), `master` hat die finalen.
  - Die eigentliche “Quelle” der finalen Version ist euer Release-Prozess, nicht das Bauchgefühl im Merge.

</details>

<details>
<summary><strong>Fall B: Keine Beta/RC-Suffixes, Version wird manuell in <code>package.json</code> gepflegt (ihr müsst euch kümmern)</strong></summary>

- **Prinzip**: Weil `develop` und `master` dieselbe `version`-Zeile anfassen können, ist das ein häufiger Merge-Konfliktpunkt.
- **Regel für den Merge nach <code>master</code>**:
  - Wenn ihr mit dem Merge **genau diese Version** erstmals offiziell releast: Version kann **gleich bleiben** (z. B. `54.6.6` bleibt `54.6.6`).
  - Wenn die `develop`-Version **schon “verbraucht”** ist (bereits offiziell ausgeliefert/getaggt/kommuniziert): Dann auf `master` **inkrementieren** (z. B. `54.6.7`).
- **Regel nach dem Release (wichtig gegen Folgekonflikte)**:
  - **Immer `master → develop` zurück mergen**, damit beide Branches wieder konsistent sind.
  - Danach in `develop` frühzeitig die **nächste** Version setzen, damit nicht beide Branches wieder parallel an derselben `version` ziehen.

</details>


---

#### 5) Lockfile konsistent machen
Auch wenn der Konflikt nur `package.json` ist: **danach lokal `pnpm install` laufen lassen**, damit `pnpm-lock.yaml` zur finalen `package.json` passt, und diese Änderung **mit committen**. (Sonst “grüner Merge”, aber später kaputte Installs.)

#### 6) Minimal verifizieren (lokal)
Mindestens:
- `pnpm install`
- euer übliches `test:all` (oder das kleinste Set, das für Releases Pflicht ist)
- ggf. Windows‑Build, wenn das bei euch Release‑Kriterium ist

#### 7) Push + PR → `master`
- Integrations‑Branch pushen



```shell
git add package.json pnpm-lock.yaml
# (oder, wenn du sicher bist, dass alles passt:)
# git add .

git commit -m "merge: develop into master (resolve conflicts)"
git push --set-upstream origin merge/develop-into-master-2026-02-05
```





- PR nach `master` erstellen
- Merge in Bitbucket erst dann durchführen, wenn der PR **ohne Konflikte** und nach Review ok ist

#### 8) Danach: `master` → `develop` zurück mergen
Das verhindert, dass ihr beim nächsten Release wieder „seit Monaten nicht gemerged“ seid.

