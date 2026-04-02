## 1. Remove gitnexus from pathora/package.json

- [ ] 1.1 Remove `gitnexus` entry from the `dependencies` field in `pathora/package.json`
- [ ] 1.2 Verify the removal does not break any import or require statements in the pathora workspace

## 2. Update vercel.json for production-only install

- [ ] 2.1 Update `pathora/vercel.json` to use `npm install --omit=dev` in the install step
- [ ] 2.2 Verify `pathora/frontend/vercel.json` uses the same pattern (optional, defense-in-depth)

## 3. Verify

- [ ] 3.1 Confirm `gitnexus` no longer appears in `pathora/package.json` dependencies or devDependencies
- [ ] 3.2 Confirm `vercel.json` install command includes `--omit=dev` flag
