# Plaatsbeschrijving.be decision log

Product owner: Sam. Started 6 November 2026.
Every decision here is binding for implementation. Changing one needs a new entry, never a silent code change.
Format: decision, why, consequences. Open questions at the bottom, each with a proposed default that is NOT binding until Sam decides.

---

## D-00 Product positioning

**Decision:** The app records what landlord and tenant saw, agreed and signed, and when. The AI proposes; people check and sign. The app does not judge who is right in a deposit dispute and does not replace a legal expert.

**Why:** We can stand behind "these photos were taken in the app at this time, both parties signed this list". We cannot stand behind "this damage was caused by the tenant".

**Consequences:**
- No UI text, email, PDF or variable name claims the inventory is "certified", "legally guaranteed", "expert" or "official", until the legal check (O-01) says what we may claim.
- AI output is always labelled as a proposal until a person confirms it.
- The move-out comparison shows differences. It never says who is liable or what it costs.

## D-01 MVP scope (superseded by D-16 for region and languages)

**Decision:** Flanders only, Dutch UI. Primary user: private landlord with 1 to 5 homes, plus the tenant.

**Consequences:**
- All UI strings in `messages/nl.json`. Code, comments, docs and commit messages in English.
- Brussels, Wallonia, French UI, agencies and student housing are post-MVP.

## D-02 Roles and accounts (superseded by D-13 for the tenant)

**Decision:** Both landlord and tenant have an account and sign in. The landlord creates the home and the inventory and invites the tenant by email.

**Consequences:**
- An inventory has exactly two parties in the MVP: one landlord account, one tenant account. Co-tenants and co-owners are post-MVP (O-04).
- Neither party depends on the other's photos: both see every photo with its capture time.

## D-03 Main flow

**Decision:**
1. Set up the home: address, rooms, meters. Invite the tenant.
2. Photograph each room: one wide photo per wall plus floor and ceiling.
3. AI lists the items and their visible condition. Tenant belongings are left out.
4. AI flags what it cannot judge and asks for a close-up. Meter readings and keys are read from a photo.
5. Both parties review, add remarks and sign in the app.
6. PDF export for attaching to the lease and registration.
7. Move-out: the app guides the same photos room by room and shows per item what changed. Both sign the exit inventory.

## D-04 Photos are camera-only (Sam, 6 Nov 2026)

**Decision:** Every inventory photo is taken live in the app. No gallery upload.

**Why:** The capture time is then known, and an older photo cannot be passed off as current. Same principle as the WFSR handover capture.

**Consequences:**
- Capture uses the in-browser camera (`getUserMedia`), not a file input with `accept=image/*`.
- Each photo gets a server receive time and a content hash, both recorded in the ledger.
- The PDF shows the capture time with every photo.

## D-05 AI role

**Decision:** Claude (Anthropic API) reads photos, proposes items per room with a condition from a fixed scale, flags uncertainty, and reads meters and keys. Called only from Edge Functions or server code.

**Consequences:**
- Every AI field carries `source = ai` until a person confirms or edits it; confirmation is a ledger event.
- Uncertain items get a close-up request, never a guessed condition.
- The condition scale is fixed and short (O-03), so landlord, tenant and AI use the same words.
- The AI never sees account names, email addresses or the address of the home.

## D-06 Tamper-evident history

**Decision:** Same technique as WFSR: an append-only, hash-chained ledger per inventory, and a nightly signed checkpoint made by a job outside Supabase.

**Consequences:**
- `ledger_events` cannot be updated, deleted or truncated (triggers plus grants). Corrections are new events.
- The fingerprint formula (`ledger_hash`) is frozen once the first migration is applied. A change needs a new hash version and a decision entry.
- No personal data in ledger payloads: internal UUIDs, item facts, photo hashes, and hashes of user-written text only. Text lives in a separate erasable table.
- The checkpoint job has its own key; only its sha256 is stored in the database. Own key, own repository for proofs, nothing shared with WFSR.

## D-07 Signing in the app (Sam, 6 Nov 2026)

**Decision:** MVP uses our own in-app signature: signed-in account, recent re-authentication, typed full name plus drawn signature, and a ledger event with the hash of the final PDF. Signing sits behind one interface so an external provider (DocuSign, a Belgian provider, itsme) can be added later as a paid option without changing the flow.

**Why:** A Flemish inventory does not need a qualified signature. External providers cost per signature (twice per inventory, twice per lease) and send the photos to a third party.

**Consequences:**
- Signing happens on a frozen version. After the first signature nothing changes; a correction creates a new version and both sign again.
- The PDF contains the signature, name, time and the version hash, plus a short line explaining how to check it.
- Provider choice is a separate future decision.

## D-08 Two-step login (Sam, 6 Nov 2026)

**Decision:** TOTP two-step login, optional for both landlord and tenant. Signing always needs a recent sign-in, with or without two-step.

## D-09 Payment comes last

**Decision:** Pay per inventory, built last. Price hypothesis from the one-pager (€29 move-in, €19 move-out, €39 bundle) is untested.

**Consequences:**
- The data model has a place for payment status from the start, but no payment code until the payment task.
- Provider (Stripe or Mollie) and the moment of payment are open (O-05, O-06).

## D-10 Separate from WFSR

**Decision:** Own repository (`Gladharim14/plaatsbeschrijving`), own Supabase project (`plaatsbeschrijving`, ref `tmjmwkxpwqpxiqhnqmxy`, eu-central-1), own keys, own Vercel project, own email domain. Nothing shared with WFSR.

**Consequences:**
- WFSR code and patterns are copied and adapted, never imported or linked.
- No WFSR account, key, bucket, table or secret may be referenced.

## D-11 Own visual identity (Sam, 6 Nov 2026)

**Decision:** A new identity: calm, trustworthy, practical. Nothing that reads as WFSR (no green and gold, no Cinzel).

## D-12 Privacy from day one

**Decision:** Photos of a home are personal data. Private storage, EXIF stripped on every derivative, short-lived signed URLs, access only for the two parties of that inventory.

**Consequences:**
- Tenant belongings may appear in photos; they are excluded from the item list but the photo stays as taken.
- Retention period and deletion rules are open (O-02). Until decided, nothing is deleted automatically.
- Data processing terms with Anthropic, Supabase, Vercel and Resend are checked before the pilot.

## D-13 Landlord captures, tenant reviews by email link (Sam, 7 Nov 2026)

**Decision:** The landlord has an account and does the capture. The tenant has no account or password: an email link plus a one-time code opens the review, and a new code is asked again at signing.

**Consequences:**
- The tenant is identified by email address and code; the code check is a ledger event.
- Supersedes the tenant part of D-02. The landlord part of D-02 stands.

## D-14 72-hour review window (Sam, 7 Nov 2026; signing alone withdrawn by D-22)

**Decision:** The landlord shares the finished report with the tenant for a 72-hour review. Without a reaction, the tenant gets an automatic reminder and 72 more hours. After that, the landlord may sign alone; the PDF then states "not reviewed by the tenant".

## D-15 Comments block signing until resolved (Sam, 7 Nov 2026)

**Decision:** During review the tenant adds comments to items or rooms, with text and optionally live camera photos. The landlord accepts or rejects each comment. An open or rejected comment blocks signing for both. The app does not break a deadlock: the parties resolve it themselves, or the app suggests calling in an expert.

**Consequences:**
- An accepted comment changes the item (or adds one) and is shown in the PDF as agreed.
- A rejected comment can be withdrawn by the tenant or reopened by the landlord; while it stands, nobody signs.
- Known risk: an inventory can stay unsigned. D-22 sets what the app offers then.

## D-16 Three languages, all of Belgium (Sam, 7 Nov 2026)

**Decision:** App, AI descriptions, emails and PDF in Dutch, French and English. Scope is all of Belgium: Flanders, Brussels and Wallonia.

**Consequences:**
- Supersedes D-01 for region and languages. UI strings in `messages/{nl,fr,en}.json`.
- Each person picks their language; the report language is chosen per inventory, with each party able to view it in their own language.
- The region of the address decides the legal template and clauses (three regional rental laws). O-01 now covers all three regions.

## D-17 Close-up loop with manual override (Sam, 7 Nov 2026)

**Decision:** Per room the AI keeps asking close-ups until no item is uncertain. When a close-up is impossible, the landlord fills in the condition manually; the report marks it "filled in by the landlord, no close-up".

## D-18 Voice notes (Sam, 7 Nov 2026)

**Decision:** Notes can be dictated. The audio is transcribed, then deleted; only the text is kept, as a user-written note (erasable, hash in the ledger).

## D-19 Payment before signing (Sam, 7 Nov 2026)

**Decision:** Capture, AI, report and review are free. The landlord pays €15 per report (incl. VAT) before signing. Resolves O-06.

## D-20 Personal data is managed, minimally (Sam, 7 Nov 2026)

**Decision:** The product processes personal data by nature (names, emails, address, signatures, photos of a home, notes). We store it, but only what the report needs: no phone number, national register number or date of birth. Personal data sits in separate tables, never in ledger payloads, in the EU, and is erasable without breaking the proof.

## D-21 Report by email (Sam, 7 Nov 2026)

**Decision:** When the report is signed, the PDF is emailed to landlord and tenant. Both can download it again from the app or the email link.

## D-22 No inventory without consensus (Sam, 7 Nov 2026)

**Decision:** An inventory exists only when both parties signed the same version. Nobody signs alone, ever. Withdraws the "landlord may sign alone" part of D-14; the review window and reminders stay.

**Why:** A plaatsbeschrijving is by definition a joint record; that is why it is made. A record signed by one party is that party's version, the same weakness as today's templates.

**Consequences:**
- The app's job is to make agreement easy: clear photos with times, close-ups, comments with photos, accept or reject, new versions, reminders.
- When agreement fails (a rejected comment that stands, or no reaction from the tenant), the landlord or the tenant can download a "not agreed" file: the latest version, every comment with both positions, all photos with times, clearly marked as unsigned and not an inventory. It is meant for the legal route (expert appointed through the justice of the peace, to confirm in O-01).
- No "signed by landlord only" state, flag or PDF text anywhere.

## D-23 MVP is move-in only (Sam, 7 Nov 2026)

**Decision:** The MVP covers the move-in inventory only. Move-out with the per-item comparison is a later phase.

**Consequences:**
- The data model keeps room for it (inventory kind, link from a move-out to its move-in, item links), so nothing has to be rebuilt later; no move-out screens, AI comparison or tests in the MVP.
- Pricing, review and signing rules are written for move-in; move-out gets its own decisions when that phase starts.

---

## Open questions (proposed defaults, not binding)

| Id | Question | Proposed default |
|---|---|---|
| O-01 | Which format and fields does an inventory need to be registrable in Flanders, Brussels and Wallonia, and what may we claim about validity? | Check with a lawyer or rental expert before the pilot; PDF follows the common Flemish model until then |
| O-02 | How long do we keep photos and inventories after the lease ends? | Until the deposit is settled plus the legal limitation period; landlord can export and delete |
| O-03 | Which condition scale? | Nieuw, Goed, Gebruikt, Beschadigd, Defect, plus "niet te beoordelen" |
| O-04 | Can the tenant add items, photos and close-ups, or only remarks? | Resolved by D-15 |
| O-05 | Payment provider? | Mollie (Belgian payment methods, Bancontact) |
| O-06 | When does the landlord pay? | Resolved by D-19 |
| O-07 | What if the parties disagree on an item? | Resolved by D-15: blocks signing |
| O-08 | Final name and company (own or under WFSR's company)? | Plaatsbeschrijving.be as working name |
