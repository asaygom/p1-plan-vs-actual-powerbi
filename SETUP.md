# P1 repo skeleton — setup notes

## Option A (recommended): keep `.pbix` out of Git
- `.gitignore` already excludes `*.pbix` to avoid heavy repos.
- Publish `PDF` and `screenshots` to `/assets` from Power BI.

## Option B: track `.pbix` with Git LFS
- See `.gitattributes-LFS.example` for commands.
- Mind LFS quotas on your Git host.

## Expected final structure
p1-plan-vs-actual-powerbi/
├─ README.md ✅
├─ p1.pbix (local only, not in Git)
├─ /data ✅
├─ /assets (screenshots here)
└─ Git files ✅

## Suggested first commit
```bash
git init
git add .gitattributes .gitignore data assets
git commit -m "Initialize P1 repo skeleton (data/assets + git settings)"
```

## Next
- Add `README.md` from our template.
- Build `p1.pbix` locally and export screenshots to `/assets`.
