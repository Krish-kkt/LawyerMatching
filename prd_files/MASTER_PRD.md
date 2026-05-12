# Master PRD — Lawyer Connect (v0.2)

> Working title. v0.2 reframes around a Q&A board + lawyer directory, no booking/scheduling. Bangalore-only at launch.

---

## 1. Problem

- People in Bangalore with legal problems don't have a personal lawyer and don't know how to find one they can trust.
- Lawyers — especially solo practitioners with bandwidth — want more clients but lack consistent inbound.
- Existing platforms are either passive phone directories or operate in BCI's red zone with paid rankings and reviews.

## 2. Product (one line)

A mobile app where a person in Bangalore can post a legal problem (or browse a lawyer directory), chat with interested lawyers without revealing contact details, and pay through the platform to unlock contact info and take the engagement offline.

## 3. Scope

**v1 / MVP**. Two entry surfaces converging into one loop:

```
Post question  ─┐
                ├─→  Chat (PII-masked) → Quote → Pay → PII unlocked → Take offline
Browse directory─┘
```

After payment, the platform's job is done. Refunds, scheduling, and engagement tracking are between the user and lawyer.

Geography: **Bangalore only**.

## 4. Users

| User    | Description                                                                 |
|---------|-----------------------------------------------------------------------------|
| Client  | A person in Bangalore with a legal problem.                                 |
| Lawyer  | A practicing advocate enrolled with the Karnataka State Bar Council, Bangalore-based. |
| Admin   | Internal — handles verification fallback and abuse flags. Not user-facing.  |

## 5. Core surfaces

### Surface A — Question board

- A client posts a problem tagged with **one AOP**.
- All verified lawyers in that AOP (Bangalore) see the post in their in-app feed.
- Any lawyer can tap "I can help" → this initiates a chat thread with the client.
- The client sees a list of responding lawyers and can chat with one or more in parallel.

### Surface B — Lawyer directory

- The client browses verified Bangalore lawyers, filtered by AOP.
- Card shows: name, photo, AOPs, years of practice, languages, short bio. **No ratings, no fees, no rankings.** Default sort: randomized per session.
- Tapping a lawyer opens a "Start chat" flow.

Both surfaces use the **same chat → quote → pay → unlock** flow.

## 6. Core flows

### Lawyer

1. **Sign up**: phone OTP → name, email → enter Karnataka SBC enrolment number → automated verification (see §7) → upload Sanad as fallback → enter AOPs, languages, short bio, photo → live on directory once verified.
2. **See feed** of new questions in their AOP(s).
3. **Tap "I can help"** on a question → chat thread opens with that client.
4. **Receive chat requests** from the directory (same chat model).
5. **Chat with client** under PII firewall.
6. **Send Quote** — a structured message with rupee amount.
7. After client pays: receive payout (gross minus 15%), PII firewall lifts for that chat.

### Client

1. **Sign up**: phone OTP → name → done.
2. Pick a path:
   - **Path A — Post question**: choose AOP → write problem (text only, max 1000 chars) → submit. Wait for lawyer responses (push + in-app notification).
   - **Path B — Browse directory**: filter by AOP → tap a lawyer → "Start chat".
3. **Chat** with one or more lawyers, PII masked both ways.
4. **Receive Quote** from a lawyer → review → tap Pay → Razorpay flow.
5. After payment: PII firewall lifts for that chat. Free to share contacts, schedule, continue offline.

## 7. Lawyer verification (automated where possible)

Priority order:

1. **Karnataka SBC enrolment number lookup** — scrape or API-check against the State Bar Council's public roll. If match found and name matches → auto-verified.
2. **Sanad certificate OCR + check** — extract enrolment number from uploaded Sanad and cross-check with step 1.
3. **Admin manual review** — fallback if both above fail. Goes into an admin queue.

Re-verification at 12-month intervals or on report/abuse flag.

## 8. PII firewall (chat)

Before payment, the following content is **blocked, redacted, and the sender warned**:

- Phone numbers (regex, including spelled-out digits)
- Email addresses (regex)
- URLs and social handles (Instagram, LinkedIn, X, WhatsApp links, Telegram)
- Full street addresses
- WhatsApp / Telegram / Signal usernames

**Implementation**: regex-based first-pass for fast detection, plus an LLM-assisted second-pass (Claude Haiku or similar) for ambiguous patterns ("my number is one two three…"). Both sender and recipient see a "This message was redacted" banner.

After payment is completed for a given chat, the firewall **lifts only for that chat**.

## 9. Quote & payment

- **Quote** is a structured message type only the lawyer can send. Fields: amount (₹), short description.
- Client sees Quote in chat with a Pay button.
- Razorpay flow; on success:
  - 15% platform commission held
  - 85% queued for lawyer payout (Razorpay Route, released after 24-hour dispute window)
  - PII firewall lifts for that chat
- **No escrow tied to service delivery** — once paid, platform doesn't track whether the legal work happened. That's between user and lawyer (per your decision on Q2/Q7).

## 10. Modules (build order)

1. **Auth & onboarding** — both sides, OTP-based
2. **Lawyer profile + automated verification** — SBC lookup + Sanad OCR + admin fallback
3. **Question posting + AOP feed** — client posts, lawyer feed
4. **Lawyer directory** — list, filters, profile view
5. **Messaging** — chat, PII firewall, structured Quote message type
6. **Payments** — Razorpay + Route, payout, commission
7. **Admin panel** — verification fallback queue, abuse reports

Each module gets its own Module PRD.

## 11. Out of scope for v1

- Slots, scheduling, booking
- Voice / video calls
- Ratings, reviews, testimonials
- Public Q&A (questions are visible only to matched lawyers, not browsable by other users or search engines)
- Document upload in chat
- Multi-AOP per question
- Cities beyond Bangalore
- Multi-language UI
- AI-assisted legal answers or auto-drafting
- Subscriptions or retainers
- Web app (mobile app only per your call on Q10)
- In-platform tracking of post-payment engagement (refunds, cancellations, disputes — all between user and lawyer)

## 12. Monetization

- **15% commission** on every successful payment.
- Lawyer sets quote amount per case in chat — no published fees.
- Platform takes 15% at payout; lawyer receives 85% via Razorpay Route after 24-hour hold.

No subscriptions, no paid placement, no premium tiers in v1.

## 13. Tech stack

- **Frontend**: React Native (mobile app only per Q10) — Expo for faster iteration
- **Backend / DB / auth**: Supabase
- **ORM**: Drizzle
- **Payments**: Razorpay + Razorpay Route for split payouts
- **Messaging**: Supabase Realtime for chat
- **PII detection**: regex layer + Claude Haiku (or GPT-4o-mini) for ambiguous cases
- **OTP**: MSG91
- **Push notifications**: Expo Push / Firebase
- **Email**: Resend
- **Background jobs** (payout release, verification jobs): Inngest
- **File storage** (Sanad uploads, profile photos): Cloudflare R2

## 14. Success metrics (3 months post-Bangalore launch)

| Metric                              | Target  |
|-------------------------------------|---------|
| Verified lawyers (Bangalore)         | 50      |
| Registered clients                   | 1,000   |
| Questions posted                     | 500     |
| Questions with ≥1 lawyer response    | 70%     |
| Chats initiated (across both surfaces)| 300    |
| Chats reaching a paid Quote          | 10% (~30 paid txns) |
| Average transaction value            | ₹1,500  |
| Platform commission per txn          | ~₹225   |
| Total platform revenue (3 months)    | ~₹10,000|
| Lawyer 30-day retention              | 50%     |
| Client repeat rate (90-day)          | 10%     |

v1 is about proving the loop, not scale. Revenue target is intentionally modest.

## 15. Risks

### Regulatory (BCI Rule 36) — still gray, mitigated by design

- Questions are private to relevant verified lawyers, not a public solicitation board
- No ratings, no reviews, no testimonials
- Directory sort is randomized; no paid placement, no "top lawyer" labels
- Lawyers do not publish fees on profile — quotes are case-by-case in chat
- Platform branding: "directory + chat" rather than "matching" or "referral"
- Plain disclaimer at signup: platform does not endorse advice given by lawyers; engagement after payment is between client and lawyer

### Pivot paths if BCI pressure increases

- Drop the 15% commission, switch to flat lawyer subscription (₹X/mo for directory listing)
- Make platform passive directory only, payments removed

### Other risks

- **Cold start (lawyer side)**: need ~30 verified Bangalore lawyers on day 1 or questions go unanswered. Plan a 1-month outbound sourcing sprint before client-side launch.
- **PII leakage** despite firewall: regex has gaps; the LLM second pass adds latency but reduces leakage. Need clear ToS that platform isn't liable for circumvention.
- **DPDP Act 2023**: legal problem descriptions are sensitive. Need consent flow at posting, data minimization, deletion-on-request.
- **Verification fraud**: fake Sanad uploads. Auto-verification + occasional spot-check on the SBC roll mitigates.
- **Lawyer no-quote / abandoned chats**: chat sits open forever with no action. Auto-archive after 7 days of inactivity (see Open Q).
- **User abuse**: spam questions, lawyer-spam clients. Rate limits on posting and chat-initiation per user per day.

## 16. Open questions (must close before architecture doc)

1. **Question visibility**: confirm — visible to all verified lawyers in matching AOP+Bangalore only, no public exposure, no search-engine indexing. Yes/no?
2. **Parallel chats per question**: can client chat with 2–3 responding lawyers in parallel, or only one at a time?
3. **Quote initiation**: lawyer-only, or can client also request a quote?
4. **Payout window**: 24 hours after payment, or instant?
5. **Question timeout**: auto-close question after X days with no lawyer response (X=?), or keep open indefinitely?
6. **Chat timeout**: auto-archive chat after Y days of inactivity (Y=?)
7. **Lawyer notification frequency**: push per new question, daily digest, or both?
8. **Max open chats per lawyer**: cap (for quality control), or unlimited?
9. **Anti-abuse**: questions per client per day cap? Chats per lawyer per day cap?
10. **AOP taxonomy v1**: which AOPs to support at launch? Suggested: Criminal, Family/Matrimonial, Property/Real Estate, Consumer, Cheque Bounce (S.138), Employment/Labour, Cyber, Civil/Recovery. Confirm or trim.
11. **Refund handling at platform layer**: confirmed zero refunds from platform — client must resolve with lawyer directly. Surface a clear disclaimer at pay time. Yes/no?

## 17. Documentation hierarchy

```
Master PRD (this doc, v0.2)          ← direction
   ↓
Architecture Document                ← DB schema, API contracts, auth, payments, messaging, PII firewall
   ↓
Module PRDs (in order from §10)      ← execution
```

Module PRD build order:
1. Auth & onboarding
2. Lawyer profile + automated verification
3. Question posting + AOP feed
4. Lawyer directory
5. Messaging (with PII firewall + Quote)
6. Payments
7. Admin panel

---

**Status**: Draft v0.2. Locks: §1–§7, §9–§13, §15. Open: §16.