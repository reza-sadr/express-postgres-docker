# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**Start the full stack (app + database):**
```bash
docker-compose up
```

**Development mode (requires a running PostgreSQL instance):**
```bash
npm run dev   # nodemon with auto-restart on file changes
npm start     # production node
```

**Code formatting:**
```bash
npx prettier --write .
```

There are no automated tests. Manual API testing is done via the `users.rest` file using the VSCode REST Client extension.

## Architecture

This is a plain Express + Sequelize REST API containerized with Docker. There are no TypeScript, bundlers, or build steps — the JavaScript runs directly.

**Request lifecycle:**
```
index.js (middleware + routing) → routes/users.js → controllers/users.js → models/user.js → utils/database.js (Sequelize/PostgreSQL)
```

**Key design points:**
- `utils/database.js` exports the single Sequelize instance; all models import from it.
- `models/user.js` defines the schema. The database table is created automatically via `sequelize.sync()` called in `index.js` at startup — there are no migration files.
- `index.js` sets permissive CORS headers (`*`) on every response directly in middleware, not via a library.
- The Dockerfile installs only production dependencies (`npm ci --omit=dev`) and runs `node index.js` directly; `nodemon` is only for local development.

## Environment & Configuration

Copy `.env.template` to `.env` before running locally. Docker Compose injects these same variables into the `node_app` container; the `node_db` container (PostgreSQL 12 Alpine) uses them as its initial credentials.

```
PORT=3000
PG_DB=node_db
PG_USER=reza.sadr
PG_PASSWORD=12345
PG_HOST=127.0.0.1   # use "node_db" when running inside Docker Compose
```

## Code Style

Prettier is configured (`.prettierrc`) with:
- No semicolons (`"semi": false`)
- Single quotes (`"singleQuote": true`)
