# AGENTS.md

## Cursor Cloud specific instructions

### Overview

This is a Next.js 15 user authentication demo with SQLite, iron-session, and Playwright E2E tests. It is a single self-contained service with no external dependencies.

### Running the dev server

- `npm run dev` starts on port 3000 by default (Turbopack). Playwright config expects port **3001**, so start manually with: `npx next dev --turbopack --port 3001`
- The `SECRET_COOKIE_PASSWORD` env var must be set (>=32 chars) for session encryption. A `.env.local` file is created during setup with a default value. If missing, create one: `echo 'SECRET_COOKIE_PASSWORD=this_is_a_secret_password_at_least_32_chars_long_for_iron_session' > .env.local`

### Database

- SQLite is embedded (file-based at `main.db`). No external DB process needed.
- Schema auto-initializes on app startup. Delete `main.db` to reset.

### Testing

- **Lint**: `npm run lint` (pre-existing lint errors exist in the codebase)
- **Production build**: `npm run build` fails due to the same lint errors (Next.js runs lint during build). The dev server is unaffected.
- **E2E tests**: `npx playwright test --project=chromium` — uses port 3001. Some tests fail due to pre-existing issues (stagewise toolbar intercepting clicks, oklch vs rgb color format mismatches). Auth tests pass.
- **Playwright browsers**: Only Chromium is installed by default (sufficient for development). Install all browsers with `npx playwright install --with-deps` if needed.

### Gotchas

- The `@stagewise/toolbar-next` dev dependency renders a companion overlay in the browser that can interfere with Playwright click actions (intercepting pointer events). This causes timeouts in dark-mode and accessibility tests.
- Tailwind CSS 4 uses `oklch()` color format by default; some color-contrast tests expect `rgb()` format and fail.
- Standard commands are documented in `CLAUDE.md` and `package.json` scripts.
