# Plaatsbeschrijving.be build plan

Work top to bottom, one task at a time. Tick `[x]` when done, tests green and committed.
References: D and O ids in `docs/decisions.md`. A task that needs an open question (O-xx) waits for Sam's decision.

## S0 Setup

- [x] S0-01 Repo, CLAUDE.md, decision log, this plan
- [ ] S0-02 Main screen mockups reviewed and approved by Sam
- [ ] S0-03 Next.js 16 scaffold copied from WFSR patterns: TypeScript strict, App Router, Tailwind 4, ESLint, Prettier, Vitest, Supabase SSR clients, `proxy.ts`, `messages/nl.json` via `lib/i18n.ts`, own theme tokens (D-11)
- [ ] S0-04 Core migration: profiles, homes, rooms, meters, inventories (move-in, move-out, version), parties, items, photos, event notes, append-only hash-chained `ledger_events` (D-06), audit log. Database tests first (`scripts/db-test/run.sh`)
- [ ] S0-05 Migrations applied to Supabase `tmjmwkxpwqpxiqhnqmxy` (Sam confirms first), types generated
- [ ] S0-06 CI: lint, typecheck, unit tests, wording check (D-00), database tests, build
- [ ] S0-07 Accounts and secrets: Vercel project, Anthropic key with spend limit, Resend domain; all in Vercel/Supabase settings, `.env.example` only in the repo

## S1 Accounts and homes

- [ ] S1-01 Auth screens: sign up, email verification, sign in, password reset, sign out, account page (Dutch)
- [ ] S1-02 Optional TOTP two-step login and recent re-auth helper (D-08)
- [ ] S1-03 Homes: create, edit, list (address, type, rooms from a template, meters)
- [ ] S1-04 Inventory: start a move-in inventory for a home; status overview per room
- [ ] S1-05 Invite the tenant by email, accept, link to the inventory (D-02); Resend templates

## S2 Capture and AI

- [ ] S2-01 Storage: private bucket, path `<inventory_id>/<room_id>/...`, policies for the two parties only (D-12)
- [ ] S2-02 Camera capture (no gallery, D-04): guided shots per room (walls, floor, ceiling), orient, resize, re-encode (drops EXIF), hash; server re-hash and ledger event
- [ ] S2-03 Edge Function `list-items`: room photos in, items with condition, location and uncertainty out (D-05); tenant belongings excluded; test set from the 6 Nov living room
- [ ] S2-04 Item review per room: confirm, edit, add, remove; every change a ledger event
- [ ] S2-05 Close-up requests: AI flags, guided close-up capture, item updated with the close-up
- [ ] S2-06 Meters and keys: photo in, reading and count proposed, confirmed by a person

## S3 Review, sign, export

- [ ] S3-01 Tenant review: remarks per item and close-ups (per O-04 decision)
- [ ] S3-02 Freeze a version; hash of the frozen content (D-07)
- [ ] S3-03 In-app signature behind a signing interface: re-auth, typed name, drawn signature, ledger event with PDF hash (D-07)
- [ ] S3-04 PDF export in the Flemish format (waits for O-01), with photos, capture times, signatures and how to check the hash
- [ ] S3-05 Correction after signing: new version, both sign again

## Later: Move-out (D-23, not in the MVP)

- [ ] S4-01 Start a move-out inventory from the signed move-in version
- [ ] S4-02 Guided capture matching the move-in shots per room
- [ ] S4-03 Edge Function `compare-items`: per item before/after, change flagged, no liability wording (D-00)
- [ ] S4-04 Comparison screen per item; remarks; both sign the exit inventory; PDF with the comparison

## S5 Proof and hardening

- [ ] S5-01 Nightly signed checkpoint job outside Supabase, own key, own proofs repository (D-06)
- [ ] S5-02 Check page: anyone with the PDF can check the version hash against the checkpoint
- [ ] S5-03 Rate limits, security review (RLS, function grants, storage), advisors clean
- [ ] S5-04 Privacy: retention and deletion (O-02), account deletion, export of own data, processor terms checked

## S6 Payment

- [ ] S6-01 Pay per inventory (provider O-05, moment O-06)

## S7 Pilot

- [ ] S7-01 Legal check of the PDF and wording (O-01)
- [ ] S7-02 Pilot with 10 private landlords and their tenants; feedback round
