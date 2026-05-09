# Newsletter SaaS — Project Spec

**Status:** v0.1 draft
**Build context:** Solo founder, target ship date 4–6 weeks from start
**Goal:** A focused newsletter tool that beats Mailchimp/Substack on price predictability and beats them on speed-to-first-send for one specific niche.

---

## 1. One-liner

A no-bloat newsletter tool with flat per-send pricing for **[NICHE]**. Import your list, write a post, hit send. No automations, no funnels, no AI credits — just newsletters.

> **Decision needed before week 1:** lock the niche. Strong defaults below; pick one and don't change it for v1.

### Niche shortlist (pick one)

| Niche | Why it works | Where to find them |
|---|---|---|
| **Substack writers tired of the 10% cut** | Already paying for newsletters. Already proven they'll move (Beehiiv stole thousands). Pre-built audience, just need lower fee. | r/Substack, Twitter/X, Substack Notes |
| **Local restaurants & cafes** | Recurring need (weekly specials), low tech literacy means they hate Mailchimp, willing to pay $20–50/mo, sticky. | Local Chamber of Commerce, Instagram outreach |
| **Real estate agents** | High willingness to pay, send the same kind of email weekly, currently overpaying for Mailchimp. | LinkedIn, BiggerPockets, local MLS groups |
| **Coaches & course creators under $100k/yr** | Outgrew ConvertKit's free tier, can't justify $79/mo, want simple. | IndieHackers, Twitter, Circle communities |

**Recommended default if you're undecided: Substack refugees.** They're the highest-intent customers in software — a writer paying Substack 10% on $50k of revenue is paying $5k/yr to send emails. You can charge them $20/mo and save them thousands. Acquisition is straightforward (they post about leaving Substack publicly).

---

## 2. Core 20% — what's IN v1

The minimum a writer needs to send a newsletter to a list and not look unprofessional:

- Email + password auth (no SSO yet)
- Import a CSV list of subscribers (name, email)
- A public hosted signup page (one URL per account, customizable: logo, colors, headline, description)
- Embeddable signup form (one `<script>` tag)
- Single-step opt-in (skip double opt-in for v1; add later)
- Compose post: simple editor — Markdown OR a Tiptap/Lexical WYSIWYG with bold/italic/links/headings/images/quote. **No drag-drop blocks.**
- Send now OR schedule for later
- Required by law (do not skip): unsubscribe link in every email, physical sender address in footer, sender name + reply-to
- Subscriber list view: search, manual remove, see status (subscribed / unsubscribed / bounced)
- Per-post analytics: sends, delivered, opens, unique clicks, unsubscribes
- Custom sending domain (DKIM/SPF setup with copy-paste DNS instructions)
- Stripe-based subscription billing

That's it. If a feature isn't on this list, it's out of v1.

## 3. Explicitly OUT of v1 (resist the urge)

- Automations / drip sequences / autoresponders
- A/B testing
- Segmentation beyond a single tag
- AI subject line generation
- Paid subscriptions / monetizing the writer's own audience (huge scope, defer)
- Referral programs (SparkLoop-style)
- Mobile app
- Multi-user accounts / team seats
- API access
- Webhooks
- Migrate-from-X import wizards beyond CSV (manual export instructions are fine)
- RSS-to-email
- Web archive / public post pages (defer to v1.1 — not strictly needed for v1)

## 4. The wedge (what makes someone switch)

Three concrete promises, each tied to a specific competitor pain:

1. **Flat predictable pricing.** Mailchimp/ConvertKit charge per subscriber and the bill creeps up forever. We charge per send (or a flat tier), so a 50k list that sends weekly costs the same as a 50k list that sends monthly × 4. **Show the savings calculator on the landing page.**
2. **5 minutes from signup to first send.** Mailchimp onboarding is 20+ minutes of nag screens. Ours: signup → import CSV → write → send. No tour, no "verify your industry," no upsells.
3. **No subscriber-count hostage situation.** No paywall on the number of subscribers below the tier. No "your account is suspended, upgrade to send." Predictable.

Pick *one* of these three to be the headline on the landing page. The other two are supporting points.

---

## 5. Tech stack (chosen for solo + speed, not for resume)

| Layer | Choice | Why |
|---|---|---|
| Framework | Next.js (App Router) on Vercel | One deploy target, auth + UI + API in one codebase. |
| DB | Postgres on Neon or Supabase | Generous free tier, branching for dev. |
| ORM | Drizzle or Prisma | Either is fine; pick the one you've used. |
| Auth | Clerk or Lucia or NextAuth | Clerk if you want zero-effort, Lucia if you want control. |
| Email sending | **Resend** (primary recommendation) or Postmark | Resend has the best DX for shipping fast and supports custom domains, batching, webhooks for opens/clicks/bounces. Postmark has slightly better deliverability reputation but worse DX. **Do NOT build SMTP yourself.** |
| Editor | Tiptap (with StarterKit) | Most flexible, lots of examples. Keep the toolbar minimal. |
| Billing | Stripe Checkout + Customer Portal | Don't build a custom billing UI. |
| Background jobs | Inngest or Trigger.dev (free tiers) | For sending batches without timing out a serverless function. Critical — don't try to send 10k emails in a Vercel handler. |
| Analytics tracking | Resend's webhook events → write to Postgres | Don't roll your own pixel; Resend already does it. |
| Hosting | Vercel | Free until you have customers. |
| Domain / DNS | Cloudflare | Free, fast DNS lookups. |

### Critical infrastructure note

The single hardest problem in newsletters is **deliverability**. Resend handles IP reputation, bounce processing, and feedback loops for you. Use them. Charge enough that their bill ($20/mo for 50k emails on the Pro plan, then $0.40/1k after) is comfortable. Do not try to save money by going to Amazon SES on day one — the unwarmed-IP and bounce-handling work is its own multi-week project.

## 6. Pricing model (v1)

Two tiers. That's it.

- **Hobby — $0/mo.** Up to 500 subscribers, up to 2 sends per month, your branding in footer.
- **Pro — $19/mo flat.** Up to 25,000 subscribers, unlimited sends, custom sending domain, no branding.
- **Scale — $49/mo flat.** Up to 100,000 subscribers, everything in Pro, priority support email.

(For comparison: ConvertKit Creator at 25k subs is $179/mo. Mailchimp Standard at 25k is ~$259/mo. The $19 number is the headline weapon.)

Stripe Checkout. Annual toggle gives 2 months free. No usage-based billing in v1 — it's complex to build and complex to explain.

---

## 7. Build plan (4–6 week sprint)

### Week 1 — Foundation
- Project scaffolding, Postgres schema (users, lists, subscribers, posts, sends, events)
- Auth (signup, login, password reset)
- Empty dashboard shell
- Landing page (just enough to share — headline, pricing, signup)

### Week 2 — List + composer
- CSV upload + parse + insert subscribers (handle dupes, malformed emails)
- Subscriber list table with search and pagination
- Tiptap editor inside a "new post" page
- Save as draft

### Week 3 — Send pipeline
- Resend integration with custom domain setup wizard (show DNS records, verify)
- "Send test to myself" button (build this first — you'll use it constantly)
- Schedule post → enqueue Inngest job → batched send
- Unsubscribe link generation + handling page
- Required compliance footer

### Week 4 — Analytics + billing
- Resend webhook ingestion (delivered, opened, clicked, bounced, complained)
- Per-post analytics view
- Stripe Checkout for Pro/Scale tiers
- Customer Portal link
- Plan limits enforcement (block sends if over subscriber cap)

### Week 5 — Polish + launch prep
- Public hosted signup page
- Embeddable form snippet
- Onboarding flow: 3 steps max
- Error states, empty states
- Landing page final pass with savings calculator
- Terms of Service, Privacy Policy (use a template, link a generator)

### Week 6 — Launch
- Soft launch to your niche community (one post, one place — do not spam everywhere)
- Personal outreach to 20 specific people in the niche
- Fix the first 5 bugs people hit, ignore feature requests for 30 days

If you're at the end of week 4 and analytics aren't done — **ship without per-post analytics.** Send confirmation + "X subscribers received this" is enough for v1. Real analytics in v1.1.

---

## 8. Go-to-market (the part most spec docs skip)

- **Where to launch:** the one community where your niche lives. For Substack refugees: a Twitter thread tagging Beehiiv and asking "what's missing?", then a Show HN, then IndieHackers. For local restaurants: cold email + walk-ins. **Do not Product Hunt this in week 1** — wait until you have testimonials.
- **Pricing page must include a comparison table** vs Mailchimp at 5k, 25k, 50k subs. Numbers do the selling.
- **First 10 customers should be hand-held.** Personally migrate their list, personally help with DNS. The feedback is worth more than the MRR.
- **Build in public** if it fits your personality. If it doesn't, don't fake it.

## 9. Open decisions before you start

1. **Niche** (lock this — section 1)
2. **Markdown vs WYSIWYG editor** — recommend WYSIWYG, writers are not programmers
3. **Auth provider** — Clerk if you want speed, Lucia if you want zero monthly cost
4. **Send provider** — Resend recommended
5. **What's the one headline promise?** (flat pricing / 5-min onboarding / no subscriber paywall)

## 10. Risks & how to mitigate

- **Deliverability nightmare.** Mitigation: use Resend, do not buy email lists, require explicit opt-in, monitor bounce/complaint rates from week one.
- **A spam customer ruins your shared sending reputation.** Mitigation: ToS that lets you cancel, manual review of accounts that import >10k subscribers in week 1, complaint rate threshold that auto-pauses sends.
- **You build it and nobody comes.** Mitigation: pre-sell. Get 5 paying letters of intent before week 4. If you can't, the niche is wrong, not the product.
- **Scope creep from your own brain.** Mitigation: re-read section 3 every Monday morning.

---

## Appendix: data model sketch

```
users(id, email, password_hash, plan, stripe_customer_id, created_at)
lists(id, user_id, name, default_from_name, default_reply_to, sending_domain, dns_verified_at)
subscribers(id, list_id, email, name, status, source, subscribed_at, unsubscribed_at)
  index on (list_id, email) unique
posts(id, user_id, list_id, subject, preheader, body_html, body_md, status, scheduled_at, sent_at)
sends(id, post_id, subscriber_id, resend_id, status, sent_at)
  index on (post_id), (subscriber_id)
events(id, send_id, type, payload_json, occurred_at)
  type ∈ {delivered, opened, clicked, bounced, complained, unsubscribed}
```

Keep `events` append-only. Aggregate to `posts.stats_json` on a schedule for the dashboard view.
