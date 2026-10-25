# Notes — Claude Code set-up

## What's in `CLAUDE.md`, and what I left out

I kept the four parts the project asks for, each written so Claude can act on it without asking:

- **Description** — one line on what the API is.
- **Commands** — `dev`, `test`, `lint`, plus how to run a single test file or a single test (`node --test <file> --test-name-pattern=...`), which isn't obvious from `package.json`.
- **Conventions** — "use X, not Y" rules that come from the code and ESLint config: CommonJS instead of ES modules, data access only through `db/store.js`, one router file per resource, the `{ error }` JSON shape with its status codes, and `node:test` + `supertest` instead of another test framework.
- **Architecture** — only the things you have to read several files to understand: why `server.js` calls `listen()` only when run directly, that the in-memory store is shared and never reset between tests, and why route ids are converted with `Number()`.

I deliberately left out:

- **A file-by-file listing.** Claude can see the tree itself, and the list would go stale.
- **The course instructions and set-up steps from the README.** They are one-off tasks, not guidance for working on the code.
- **Generic advice** ("write tests", "handle errors"). It adds length without changing behaviour.
- **Anything from `.env` or other secrets.** `CLAUDE.md` is committed and loaded into every session.

## Permission rules (`.claude/settings.json`)

- **allow:** `npm test`, `npm run lint`, `node --test`. They are read-only checks I run constantly, so pre-approving them removes prompts without adding risk.
- **ask:** `git push`. Pushing publishes work to the shared remote, so I want to confirm it each time.
- **deny:**
  - `Read(./.env)` and `Edit(./.env)` keep real secrets out of Claude's context.
  - `git push --force` and `git push -f` block rewriting shared history. Both forms are listed because a rule for the long flag does not match the short one.

**What could go wrong without the deny rules:**

- Claude could read `.env` while exploring and repeat the secrets in its output, a commit or a file.
- That risk is higher in this repo because the lines in `.gitignore` after the first one are indented. Git does not strip leading spaces, so `.env` and `.claude/settings.local.json` are **not** actually ignored, and a real `.env` could be committed by accident.
- A force-push could overwrite teammates' commits on the remote with no easy way back.

The deny rules cover Claude's Read and Edit tools only, so they do not stop a shell command such as `cat .env`. Fixing the `.gitignore` is the real protection.

## Verification

I started a fresh `claude` session in the project folder and checked:

- [x] **`/memory`** shows `./CLAUDE.md` loaded as project memory.
- [x] **`/permissions`** shows my rules: allow `npm test`, `npm run lint`, `node --test`; ask `git push`; deny `Read(./.env)`, `Edit(./.env)`, `git push --force`, `git push -f`.
- [x] Asking "How do I run the tests here?" gets an answer from `CLAUDE.md` (`npm test`, plus the single-test command) without extra explanation from me.
