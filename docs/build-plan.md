# Plaatsbeschrijving.be build plan

Work top to bottom, one task at a time. Tick `[x]` when done, tests green and committed.
References: D and O ids in `docs/decisions.md`; FR, RULE and T ids in the analysis and build specification (Claude Doc). A task that needs an open question (O-xx) waits for Sam's decision.
MVP = move-in only (D-23).

## S0 Setup

- [x] S0-01 Repo, CLAUDE.md, decision log, this plan
- [ ] S0-02 Analysis and build specification approved by Sam
- [ ] S0-03 Mockups of the move-in flow approved by Sam (login, new inventory, room type, capture loop with voice note, meters and keys, share, tenant review and comments, accept or reject, pay and sign, not-agreed file)
- [ ] S0-04 Next.js 16 scaffold copied from WFSR patterns: TypeScript strict, App Router, Tailwind 4, ESLint, Prettier, Vitest, Supabase SSR clients, `proxy.ts`, `messages/{nl,fr,en}.json` via `lib/i18n.ts`, own theme tokens (D-11, D-16)
- [ ] S0-05 Core migration: landlord profiles, partners, addresses, inventories (kind, region, report language, state), tenant links, rooms, versions, photos, annotations, items, close-up requests, notes, comments, meter readings, keys, signatures, payments, append-only hash-chained `ledger_events` (D-06), audit log. Personal data in separate tables (D-20). Database tests first (`scripts/db-test/run.sh`)
- [ ] S0-06 Migrations applied to Supabase `tmjmwkxpwqpxiqhnqmxy` (Sam confirms first), types generated
- [ ] S0-07 CI: lint, typecheck, unit tests, wording check in nl, fr and en (D-00), database tests, build
- [ ] S0-08 Accounts and secrets: Vercel project, Anthropic key with spend limit, speech-to-text provider (O-11), Resend domain; all in Vercel/Supabase settings, `.env.example` only in the repo

## S1 Landlord and inventory

- [ ] S1-01 Landlord auth: sign up, email verification, sign in, password reset, sign out, account page, language choice (FR-ACC-01, FR-ACC-05)
- [ ] S1-02 Optional TOTP two-step login and recent re-auth helper (FR-ACC-02, FR-ACC-03)
- [ ] S1-03 New inventory: address with region detection, date, landlord and tenant name and email, report language, weather and people present (FR-INV-01, FR-INV-02); saved addresses (FR-INV-03)
- [ ] S1-04 Rooms: add by room type, name, reorder, remove, status per room (FR-INV-04, FR-INV-05)

## S2 Capture and AI

- [ ] S2-01 Storage: private bucket, path `<inventory_id>/<room_id>/...`, access for the landlord and the tenant link only (D-12)
- [ ] S2-02 Camera capture (no gallery, D-04): guided wide shots per room type, orient, resize, re-encode (drops EXIF), hash; server re-hash and ledger event (FR-CAP-01 to 05)
- [ ] S2-03 Edge Function `list-items`: wide shots in, items with location, material, colour, condition and description in the report language out; tenant belongings excluded; test set from the 6 Nov living room (FR-AI-01, FR-AI-03, FR-AI-06)
- [ ] S2-04 Close-up loop: one request at a time until the room is done; `assess-closeup`; manual fill-in marked in the report (FR-AI-04, FR-AI-05)
- [ ] S2-05 Item review by the landlord: confirm, edit, add, remove; every change a ledger event
- [ ] S2-06 Voice notes: dictate, transcribe, edit text, delete audio (FR-CAP-06, D-18)
- [ ] S2-07 Meters and keys from photos: `read-meter`, `count-keys`, confirmed by the landlord (FR-INV-06, FR-AI-07)
- [ ] S2-08 Translations for viewing in another language, marked as such (FR-AI-02)

## S3 Review, sign, export

- [ ] S3-01 Finish the report: freeze a version with its fingerprint (FR-REV-01, D-07)
- [ ] S3-02 Share for review: tenant link plus one-time code, 72-hour window, reminder and extension (FR-REV-02, FR-ACC-04, FR-REV-07, D-13, D-14)
- [ ] S3-03 Tenant review: view everything with capture times, comment with text, voice and live photos (FR-REV-03)
- [ ] S3-04 Landlord accepts or rejects comments; accepted comments create a new version; withdraw and reopen; signing blocked while a comment is open or rejected (FR-REV-04 to 06, D-15)
- [ ] S3-05 Not-agreed file: latest version, all comments with both positions, photos with times, marked unsigned (FR-REV-08, D-22)
- [ ] S3-06 Payment of €15 before signing (FR-PAY-01, D-19; provider O-05)
- [ ] S3-07 Signing by both behind a signing interface: landlord re-auth, tenant new code, typed name, drawn signature; nobody signs alone (FR-SIGN-01 to 03, D-22)
- [ ] S3-08 PDF per region template in the report language (waits for O-01): watermarked preview, final PDF emailed to both (FR-PDF-01 to 03, D-21)
- [ ] S3-09 Correction after signing: new version, both sign again, no new payment (FR-SIGN-02)

## S4 Proof and hardening

- [ ] S4-01 Nightly signed checkpoint job outside Supabase, own key, own proofs repository (D-06)
- [ ] S4-02 Check page: anyone with the PDF can check the version fingerprint (FR-LED-03)
- [ ] S4-03 Rate limits (sign-in, codes, links, uploads, AI, speech), security review (RLS, function grants, storage), advisors clean
- [ ] S4-04 Privacy: retention and deletion (O-02), account deletion, export of own data, processor terms checked (D-12, D-20)

## S5 Pilot

- [ ] S5-01 Legal check of the PDF, wording and the not-agreed route per region (O-01)
- [ ] S5-02 Pilot with 10 landlords and their tenants; feedback round

## Later: Move-out (D-23, not in the MVP)

- [ ] L-01 Start a move-out from the signed move-in of the same address
- [ ] L-02 Guided capture with the move-in photo as reference
- [ ] L-03 Edge Function `compare-items`: unchanged, changed or not comparable, no liability wording (D-00)
- [ ] L-04 Review, comments, signing and PDF with the comparison
