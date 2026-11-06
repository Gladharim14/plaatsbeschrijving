# Plaatsbeschrijving.be: instructions for Claude

AI move-in and move-out inventory (plaatsbeschrijving) for Belgian rentals. Room photos become an item list with condition; landlord and tenant check and sign in the app; every photo and entry carries a tamper-evident timestamp; at move-out the app compares per item. Built by Sam (product owner) with Claude. No other developers.

Spin-off of WFSR (`Gladharim14/wfsr`): same stack and patterns, copied and adapted, never linked (D-10).

## Sources of truth (read before any task)

1. `docs/decisions.md`: binding decisions D-00 onwards, and open questions O-xx. Wins over everything else.
2. `docs/build-plan.md`: ordered task list. Work on the next unchecked task only.
3. `supabase/migrations/`: authoritative data model.

If the decisions, the plan and the code disagree, stop and ask Sam. **Never resolve a product question by guessing.** If a task needs an open question (O-xx) or a decision that isn't logged, write the question, propose a default, and wait. Record every new decision in `docs/decisions.md` (new D-number, date, reason) before coding it.

## Working with Sam

- Concise output, plain wording, no em-dashes anywhere (code comments, docs, UI copy, commits).
- All questions upfront, in one go, before producing output.
- Show designs before building, unless Sam says "Build it".
- Never paste secrets in chat, docs, commits or logs. Secrets live only in Vercel and Supabase settings; the repo has `.env.example`.
- Destructive SQL (DROP, DELETE, TRUNCATE, destructive ALTER, `db reset` on a shared project) needs Sam's explicit confirmation first.
- Each iteration must be a real improvement, not cosmetic.

## What the app is not (D-00)

The app records what both parties saw and signed, and when. It does not judge liability or replace an expert.
- Forbidden until the legal check (O-01): "gecertificeerd", "officieel", "juridisch geldig", "expert", "gegarandeerd", and English equivalents.
- AI output is labelled as a proposal ("voorstel") until a person confirms it.
- Move-out shows "veranderd", "ongewijzigd", "niet te vergelijken". Never "schuld", "schade door huurder", or amounts.

## Stack

- Next.js **16** (App Router, TypeScript strict, Server Components by default, Server Actions for mutations, Proxy instead of Middleware). Read `node_modules/next/dist/docs/` before using an API you think you know.
- Tailwind CSS 4, own theme tokens (D-11). UI strings in `messages/nl.json` via `t()` in `lib/i18n.ts` (Dutch only in the MVP, D-01).
- Supabase project `plaatsbeschrijving`, ref `tmjmwkxpwqpxiqhnqmxy`, eu-central-1: Postgres, Auth (email + optional TOTP), private Storage, Edge Functions, pg_cron.
- Claude API for photo reading, only from Edge Functions or server code, never the browser. Send photos and room names only, no personal data (D-05).
- Resend for email, own domain. Vercel hosting.
- Tests: database tests (`scripts/db-test/run.sh`), Vitest for units, Playwright for end-to-end.
- Supabase clients: `lib/supabase/server.ts`, `lib/supabase/client.ts`, `proxy.ts` refreshes sessions. Use `getClaims()` on the server, never `getSession()`.
- After every migration: regenerate `lib/database.types.ts`.

## Security rules (non-negotiable)

1. **Every table has RLS enabled** and starts with `revoke all ... from anon, authenticated`. Grant the minimum, column-level where possible.
2. **Inventories, items, signatures and the ledger change only through `SECURITY DEFINER` functions.** No direct client writes.
3. Every function: `set search_path = ''`, fully qualified names, explicit `grant execute` to the right role only.
4. **The ledger is append-only (D-06).** Never UPDATE/DELETE `ledger_events`, never weaken its triggers or grants. Corrections are new events.
5. **The fingerprint formula is frozen** once applied. A change needs a new hash version and a decision entry.
6. **No personal data in ledger payloads.** Internal UUIDs, item facts and hashes only. User-written text goes to `event_notes`, its hash to the payload.
7. **Access is per inventory:** only its landlord and its tenant. Not-found and not-allowed look the same (no existence leaks). Nothing is exposed to `anon`.
8. **Photos are camera-only (D-04).** Never add a gallery or file upload path for inventory photos.
9. Storage: private buckets only, short-lived signed URLs after an access check, EXIF stripped on every derivative.
10. **A signed version is frozen (D-07).** Signing needs a recent sign-in. Any change after a signature creates a new version.
11. Service role key only in Edge Functions and `server-only` code. Never in `NEXT_PUBLIC_*`.
12. Validate input twice: Zod in the server action, constraints in the database. Rate-limit sign-in, invites, uploads and AI calls.

## Database conventions

- Migrations in `supabase/migrations/` named `YYYYMMDDHHMMSS_short_name.sql`. Never edit an applied migration; add a new one.
- Functions raise with SQLSTATE and a `hint` for machine-readable cases (`reauth_required`, `version_signed`, `not_party`). The app maps hints to Dutch messages.
- Every new table, function or policy gets tests: happy path, other party, stranger, guest, each state transition.
- Apply to the cloud project only after Sam confirms.

## Workflow per task

1. Read the task in `docs/build-plan.md` and the referenced D/O ids.
2. Unclear or needs a decision: ask first, all questions at once.
3. Migration and database tests first, until green.
4. Then server code, then UI.
5. Lint, typecheck, unit tests, db tests: all pass.
6. Tick the task, commit with the task id (e.g. `S2-03: list-items edge function`).
7. Report to Sam in a few lines: what changed, what to look at, what is open.

## Style

- Mobile-first: capture and signing happen on phones, often in an empty room with poor light.
- UI copy short, neutral, factual, in Dutch ("u" or "je": see mockups once approved).
- Every AI proposal and every check result shows what it means and what it does not mean.
