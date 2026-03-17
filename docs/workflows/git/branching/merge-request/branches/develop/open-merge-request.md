
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



# Related


## Workflow: Erneuter Rebase nach Änderungen auf `develop`
- docs\workflows\git\merge-request\branches\develop\troubleshooting\rebase-develop-again-on-feat-branch.md

## Spezialfall: Von einem offenen PR-Branch abzweigen (Feature-on-Feature)
- docs\workflows\git\merge-request\branches\feature\feature-on-feature.md