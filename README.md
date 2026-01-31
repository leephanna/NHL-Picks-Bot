# NHL Picks Bot (Email + SMS)

This bot fetches **current NHL moneyline odds**, computes an estimated **no‑vig win probability** for each game, ranks the **Top 6 most likely winners**, and sends them to:
- Email (via **Resend**)
- SMS (via **Twilio**)

It’s built as a **Next.js** app designed for **Vercel Cron Jobs**.

---

## What you get

- Daily (or on-demand) ranked list: **Top 6 favorites**, most likely first  
- Puck drop times in **America/Chicago**
- Uses The Odds API (`icehockey_nhl`, `h2h` moneyline market)
- Email dedupe using Resend **Idempotency-Key** (24h)  
- SMS dedupe by checking Twilio message history for today

---

## Start here (fast setup)

### 1) Create accounts & keys

- **The Odds API**: get an API key
- **Resend**: get an API key, and set a From address you can send from (domain verification recommended)
- **Twilio**: get Account SID + Auth Token + a sending phone number

### 2) Put secrets in Vercel

In Vercel → Project → Settings → Environment Variables, add everything from `.env.example`.

**CRON_SECRET is important**: Vercel will automatically include it as an `Authorization: Bearer <CRON_SECRET>` header when invoking the cron path, and your route validates it.

### 3) Deploy

- Push this repo to GitHub
- Import into Vercel
- Deploy to Production

### 4) Test instantly

Locally:

```bash
cp .env.example .env.local
npm i
npm run dev
```

In another terminal:

```bash
curl -H "Authorization: Bearer $CRON_SECRET" "http://localhost:3000/api/cron/nhl-picks?dryRun=1"
```

Production:

```bash
curl -H "Authorization: Bearer $CRON_SECRET" "https://YOUR-VERCEL-DOMAIN/api/cron/nhl-picks?dryRun=1"
```

`dryRun=1` builds the message but **does not send**.

---

## Scheduling

`vercel.json` schedules the route **every 5 minutes**:

- The bot only **sends at 09:05 America/Chicago**
- This avoids DST headaches (Vercel cron timezone is UTC)

If you want a different time, edit:
- `lib/time.ts` (the `shouldSendNow()` logic)

---

## How ranking works (simple + robust)

For each game:
- For each bookmaker that provides both sides:
  - Convert American odds to implied probability
  - Remove vig by normalizing (pA / (pA+pB))
- Average the no‑vig probability across bookmakers
- Pick the higher probability side as “most likely winner”

Then the bot sorts games by that best-side probability and returns the Top N.

---

## Notes

- If Resend rejects the From address, you need to verify a domain and use a From address on that domain.
- Twilio trial accounts require you to verify the destination number.

