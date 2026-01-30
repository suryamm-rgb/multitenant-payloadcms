# 🚦 Git Hooks + Code Quality Workflow

**Husky + Lint-Staged + TypeScript + Tests**

This project uses automated Git hooks to keep code quality high
**without slowing down development**.

✅ Commits stay fast\
🛡️ Pushes stay safe\
🤖 CI stays green

------------------------------------------------------------------------

## 🧠 Workflow Overview

  -----------------------------------------------------------------------
  Action             What Runs                    Purpose
  ------------------ ---------------------------- -----------------------
  `git commit`       ESLint on staged files       Fast feedback, clean
                                                  code

  `git push`         Typecheck → Tests → Build    Prevent broken code
                                                  from reaching repo
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 🛠 Step 1 --- Install Required Dev Dependencies

Run this **in the project root** (where `package.json` exists):

``` bash
pnpm add -D husky lint-staged eslint typescript
```

------------------------------------------------------------------------

# 🐶 Step 2 --- Initialize Husky

``` bash
pnpm dlx husky init
```

This creates a `.husky` folder and a sample hook.

Then ensure **package.json** has:

``` json
"scripts": {
  "prepare": "husky"
}
```

------------------------------------------------------------------------

# ⚡ Step 3 --- Setup Pre-Commit Hook (Fast Checks Only)

Open the file:

``` bash
code .husky/pre-commit
```

Replace its contents with:

``` sh
#!/usr/bin/env sh
pnpm lint-staged
```

This ensures **only changed files** are linted --- super fast.

------------------------------------------------------------------------

# 🛡 Step 4 --- Setup Pre-Push Hook (Heavy Safety Checks)

Create the file:

``` bash
touch .husky/pre-push
chmod +x .husky/pre-push
code .husky/pre-push
```

Add:

``` sh
#!/usr/bin/env sh

pnpm typecheck && pnpm test && pnpm build
```

Now Git will **block pushes** if: - TypeScript has errors ❌\
- Tests fail ❌\
- Production build fails ❌

------------------------------------------------------------------------

# 📦 Step 5 --- Configure `lint-staged`

Add this to **package.json**:

``` json
"lint-staged": {
  "*.{js,jsx,ts,tsx}": [
    "next lint --fix"
  ]
}
```

This lints only staged files, not the whole project.

------------------------------------------------------------------------

# 📜 Step 6 --- Required Scripts in package.json

Make sure these scripts exist:

``` json
"scripts": {
  "dev": "next dev",
  "build": "next build",
  "start": "next start",
  "lint": "next lint",
  "lint:fix": "next lint --fix",
  "typecheck": "tsc --noEmit",
  "test": "pnpm run test:int && pnpm run test:e2e",
  "prepare": "husky"
}
```

------------------------------------------------------------------------

# ✅ ESLint Rules Enforced

Your custom ESLint setup ensures:

-   ⚠️ Warn on `any` types\
-   ⚠️ Warn on `@ts-ignore`\
-   ⚠️ Warn on unused variables (except `_ignored`)\
-   ❌ Blocks `console.log` (only `warn` and `error` allowed)

------------------------------------------------------------------------

# 🚀 CI Still Protects the Repo

Even if someone skips hooks locally, GitHub Actions runs:

``` yaml
- pnpm lint
- pnpm typecheck
- pnpm build
```

So the branch is always protected.

------------------------------------------------------------------------

# 🧪 How to Test the Hooks

### Test Pre-Commit Hook

``` bash
git add .
git commit -m "test lint hook"
```

✔ Only ESLint should run

------------------------------------------------------------------------

### Test Pre-Push Hook

``` bash
git push
```

✔ You should see: - TypeScript check\
- Tests running\
- Production build

If something fails, the push is blocked.

------------------------------------------------------------------------

# 🧩 Why This Setup is Powerful

✔ Fast commits (no waiting for tests)\
✔ Safe pushes (no broken builds in repo)\
✔ Enforced team standards\
✔ Fewer CI failures\
✔ Used in real production teams

------------------------------------------------------------------------

# 🏁 Final Result

This system ensures:

⚡ Developers move fast\
🛡️ The repository stays stable\
🤖 Code quality is automatically enforced
