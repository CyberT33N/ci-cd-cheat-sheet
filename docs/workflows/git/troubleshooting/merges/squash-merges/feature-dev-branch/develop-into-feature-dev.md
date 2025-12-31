# Develop Branch in Feature-dev Branch mergen
- Szenario kann passieren, wenn man lange auf einem Feature-dev Branch arbeitet und seitdem Änderungen auf  dem develop Branch gekommen sind, welche man nicht erst rebasen kann, wenn man auf dem Feature Branch ist


```shell
git checkout develop
git pull
git checkout feat/PRIV-146/added-ds-caching/dde
git merge --squash develop

# ----- Konflikte lösen ----
# Wenn es Merge-Konflikte gibt und du diese löst, benutze:

# Bei Konflikten mit der package-lock.json:
# Remove-Item -Path "pnpm-lock.yaml" -Force
# pnpm i

# git add .
# git commit
```
