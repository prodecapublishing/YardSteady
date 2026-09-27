# Yard Steady — Website

Reliable weekly lawn care subscription service serving Pearland, Manvel, and League City, TX.  
Built as a single `index.html` file, deployable directly to GitHub Pages with zero build tools.

---

## 🚀 Going Live Checklist

### 1. Signup Flow (manual verify, then text a Stripe link)
Signups do **not** redirect straight to Stripe checkout. A customer fills out the signup form (standard plan or custom quote) and submits it; the form posts directly to Formspree, which emails you the full submission — name, email, phone, address, plan, preferred start date, and referral code. You review it, confirm it's a real request, and then text or email them a Stripe Payment Link yourself to finish signup — the same "just text us" simplicity as cancellation, but for onboarding.

**Formspree endpoint:** `https://formspree.io/f/xgavppqo` — set as `FORMSPREE_ENDPOINT` near the bottom of the `<script>` block. Log into [formspree.io](https://formspree.io) to see submissions, manage where they're delivered, or set up email notifications/forwarding. Each submission is tagged `request_type` (`Standard Signup` or `Custom Quote`) so you can tell them apart.

**Fallback:** if the Formspree request ever fails (network issue, endpoint down), the site automatically falls back to opening a pre-filled `mailto:hello@yardsteady.com` message in the customer's own email app instead, so a request is never silently lost — just make sure that inbox is actually monitored.

The two Payment Links you'll text/email back are still defined near the top of the `<script>` block for reference:

```js
const STRIPE_SMALL = 'https://buy.stripe.com/14A4gs9M93IL91z9SEbjW00';
const STRIPE_LARGE = 'https://buy.stripe.com/6oU3cof6tfrtcdL4ykbjW01';
```

- **STRIPE_SMALL** → send for the $45/week (Standard) plan
- **STRIPE_LARGE** → send for the $60/week (Large) plan

Review/manage these at [dashboard.stripe.com/payment-links](https://dashboard.stripe.com/payment-links).

---

### 2. Referral Program (manual — no third-party platform)
Referrals are tracked manually, not through an affiliate SaaS tool or a personalized link. Each referrer gets a free, self-service "referral code" — **first name + house number**, e.g. `Brad3314` for Brad at 3314 Whatever St. No database, no signup flow, no Tolt/Rewardful subscription.

1. Give existing customers their code (e.g. `Brad3314`) — they just tell it to a neighbor or text it to them. There's no personalized link; the neighbor types the code into the "Referral Code" field on the signup form.
2. The code comes through to you as part of their submitted signup info (see section 1 above) — no Stripe field needed since you're texting the payment link yourself after reviewing the request.
3. Log new signups + the referral code into `Yard_Steady_Referral_Tracker.xlsx` (in the Drive folder) — keep a lookup of code → referrer's actual name/address there so codes stay traceable to a real person even if two people share a first name (the house number disambiguates). It has tabs for the Referral Log, a Referrer Summary (auto-counts + totals per referrer), and Credits Applied.
4. Once a referred customer's payment has gone through, apply a credit to the referrer's account using Stripe's **Customer Balance** feature (Dashboard → Customer → Balance → add a negative balance). Suggested credit: one week free ($45 or $60, matching their own plan).
5. Cap referral credits at **$600/year per referrer** — above that the IRS requires a 1099, which isn't worth the paperwork at this stage.
6. As volume grows, this can be automated with a low-cost tool like Make.com (~$9–16/mo) or Zapier (~$20–30/mo) triggered off new Stripe subscriptions — evaluate only once manual tracking becomes a bottleneck.

---

### 3. Contact Email
Find all instances of:

```
hello@yardsteady.com
```

Make sure this inbox is actually set up before launch. Appears in:
- Footer (both home and referral page)
- Custom quote mailto link in pricing section
- Custom quote modal mailto fallback

---

### 4. Business Name / Branding
Business name is **Yard Steady** (BLambInvestments LLC d/b/a Yard Steady). If it changes again, search for `Yard Steady` and update all instances.

**Note:** `logo_smaller.png` in this folder still has the old "Gulf Coast Mow" wordmark baked into the image — it's no longer referenced in `index.html` (the nav now uses plain CSS text), but a proper Yard Steady logo still needs to be designed and dropped in.

---

## 📦 Deploying to GitHub Pages

1. Create a new GitHub repository (e.g. `yardsteady`)
2. Upload `index.html` to the root of the repo
3. Go to **Settings → Pages**
4. Set source to `main` branch → `/ (root)`
5. Click Save — your site is live at:  
   `https://yourusername.github.io/yardsteady`

### Custom Domain Setup
1. Buy your domain (`yardsteady.com`) from Namecheap, GoDaddy, etc. — this repo's `CNAME` file already assumes you own it, so don't push it live until you do.
2. In GitHub → Settings → Pages → Custom domain → enter your domain
3. At your domain registrar, add these DNS records:

```
Type    Name    Value
A       @       185.199.108.153
A       @       185.199.109.153
A       @       185.199.110.153
A       @       185.199.111.153
CNAME   www     yourusername.github.io
```

4. Check "Enforce HTTPS" in GitHub Pages settings
5. DNS propagation takes 10–30 minutes

---

## 💳 Stripe Setup Notes

### How customer data reaches you
When a customer completes the signup form (standard plan or custom quote) and submits it, the site posts the submission straight to Formspree (`https://formspree.io/f/xgavppqo`), which forwards you everything you need — something like:

```
request_type: Standard Signup
name: John Smith
email: john@example.com
phone: (832) 555-1234
address: 123 Maple St, Pearland TX
plan: Standard — $45/wk + tax
preferred_start_date: Sat, Oct 4
referral_code: Brad3314
```

(A custom quote submission comes through the same way, tagged `request_type: Custom Quote`, with `notes` in place of `plan`/`preferred_start_date`.)

Once you get that notification, verify it's a real request, log the referral code (if any) in the referral tracker per section 2 above, then text or email them the matching Stripe Payment Link (STRIPE_SMALL or STRIPE_LARGE) to complete their signup — or, for a custom quote, send them your quote directly.

If Formspree is ever unreachable, the site automatically falls back to opening a pre-filled `mailto:hello@yardsteady.com` message in the customer's own email app with the same information, so a submission is never silently lost.

### Recommended Stripe settings for your Payment Links
In the Stripe Payment Link editor, enable:
- **Collect phone numbers** — backup in case you need to reach them
- **Collect billing address** — for your records
- **Collect shipping address** — use as service address backup

---

## 📐 Pricing Tiers

| Plan | Yard Size | Weekly Rate | Monthly (approx) |
|------|-----------|-------------|------------------|
| Standard | Up to 10,000 sq ft | $45/week | ~$180/month |
| Large | 10,001 – 15,000 sq ft | $60/week | ~$240/month |
| Custom | Over 15,000 sq ft | Quote required | — |

---

## 🗂 File Structure

```
yardsteady/
├── index.html      ← entire site (home + referral page)
└── README.md       ← this file
```

### When to split into multiple files
Consider splitting when you add:
- A blog or city-specific landing pages
- A backend form handler (Node/Python)
- Google Analytics or tag management scripts
- A separate admin or customer portal

---

## 🛠 Tech Stack

- **HTML/CSS/JS** — no frameworks, no build tools
- **Fonts** — Barlow Condensed + DM Sans via Google Fonts
- **Images** — Unsplash (free commercial license)
- **Payments** — Stripe Payment Links (no backend required)
- **Referrals** — Manual tracking via spreadsheet + Stripe Customer Balance credits (see section 2)
- **Hosting** — GitHub Pages (free)

---

## 📍 Service Area

Currently serving:
- Pearland, TX
- Manvel, TX
- Friendswood, TX

Coming soon:
- League City, TX
- Alvin, TX
- Clear Lake, TX

To add a new city once it's actually live, update the hero badge, the `SERVICE_ZIPS`/`COMING_SOON_ZIPS` sets, the "Service Areas" section, and the JSON-LD `areaServed` list in `index.html`.

---

## 📞 Support & Business Info

| Item | Value |
|------|-------|
| Email | hello@yardsteady.com |
| Phone | (832) 798-4827 |
| Service area | Pearland · Manvel · Friendswood, TX (League City · Alvin · Clear Lake coming soon) |
| Plans | $45/wk (standard) · $60/wk (large) |
| Referral credit | One week free per referral, applied manually via Stripe Customer Balance — capped at $600/year per referrer (IRS 1099 threshold) |
| Guarantee | 7-day re-mow or refund |

---

## ⚠️ Known gap (not related to the rebrand)

The custom-quote flow (large yards, "request a quote") has no actual delivery mechanism to notify the business owner when someone submits one — it's client-side only right now. Worth wiring up (even just a mailto or a simple form backend) before relying on it for real leads.

---

*Last updated: 2026*
