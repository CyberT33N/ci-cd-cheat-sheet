# Force merge feature-dev in frisch abgwezigten Branch von develop´
- Das folgende Szenario ist normalerweise eher selten und **MUSS** nicht passieren. Falls aber der **Feature-Branch** korrumpiert ist oder nicht mehr vorhanden ist, ist dieser **Workflow**, wenn man dann vom **Develop-Branch** einen aktuellen Stand abzweigen möchte und dann seinen **Feature-Dev-Branch** dort reinziehen will.

```
git checkout develop
git pull

git branch feat/PRIV-146/added-ds-caching-v5/main

git checkout feat/PRIV-146/added-ds-caching-v5/main
git merge --squash feat/PRIV-146/added-ds-caching/dde

git checkout --theirs -- .
git add -A
```

Falls Konflikte sind wenn Dateien gelöscht sind:
```
# 1) Diese Dateien sind in "theirs" gelöscht -> also löschen
git rm -- src/services/charly/patient-documentations.ts `
         src/services/charly/treatment-plans/index.ts `
         src/services/infrastructure/file-transfer/index.ts `
         src/services/message-handlers/FileTransferHandler.ts `
         src/services/message-handlers/GetPrivyouDetailsHandler.ts `
         src/services/nelly/treatment.ts `
         src/services/privyou-app-action.ts

# 2) Danach erneut "theirs" für den Rest nehmen (jetzt ohne diese Fehler)
git checkout --theirs -- .

# 3) Alles stagen
git add -A

# 4) Check: keine Konflikte mehr?
git diff --name-only --diff-filter=U
```

Dann commit:
```
git commit -m "feat(PRIV-146): <beschreibung>"
```
