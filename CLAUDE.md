# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Small Express REST API (users + health check) backed by an in-memory store; it is the starter project for the Claude Code course.

## Commands

- `npm run dev` — start the API on http://localhost:3000 with `node --watch` (port via `PORT`)
- `npm test` — run all tests with the built-in `node --test` runner
- `node --test tests/users.test.js` — run one test file; add `--test-name-pattern="404"` to run a single test
- `npm run lint` — ESLint (`eslint:recommended`); CI runs lint then test on Node 22, so both must pass

## Conventions

- Use CommonJS (`require` / `module.exports`), not ES modules (`import` / `export`) — ESLint is set to `sourceType: "script"`.
- Use the functions exported by `db/store.js` for all data access, not the `users` array directly from a route.
- Add a new resource as its own file in `routes/` exporting an `express.Router()` and mount it in `server.js`, not as extra handlers in an existing route file.
- Return errors as JSON `{ "error": "<message>" }` with the matching status (400 validation, 404 not found); return 201 on create.
- Write tests with `node:test` + `node:assert` + `supertest` against the exported `app`, not Jest/Mocha and not a running server.

## Architecture

- `server.js` builds the app, mounts the routers (`/users`, `/health`) and only calls `listen()` when run directly (`require.main === module`), so tests can `require("../server")` without opening a port.
- `db/store.js` holds module-level state seeded with users id 1 and 2. It is shared by every test in a process and resets only on restart, so a test that creates users changes what later tests see.
- Path params arrive as strings; `routes/users.js` converts them with `Number()` because the store compares ids with `===`.
