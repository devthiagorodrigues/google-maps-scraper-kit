# Google Maps Scraper Kit

<p align="center">
  <img src="assets/banner.svg" alt="Google Maps Scraper Kit — run a free Google Maps scraper locally with Codex" width="100%">
</p>

A **plug-and-play kit** that runs a free, open-source Google Maps scraper **on your own computer**
and lets **Codex** drive it on autopilot — give Codex a city + a business type, get back a clean
list of businesses (name, address, phone, website, rating, reviews, lat/lng, emails).

Built as a thin wrapper around [**`gosom/google-maps-scraper`**](https://github.com/gosom/google-maps-scraper)
by **Georgios Komninos** (MIT). See [CREDITS.md](CREDITS.md). This kit adds: a one-command Docker setup,
ready-to-run scripts, and a **repository skill** so Codex knows exactly how to use the API the right way.
The original Claude Code files remain available for backward compatibility.

> ⚠️ **Before you scrape — over-use can get your IP temporarily blocked.**
> This runs a real scraper against Google Maps. Light use is fine, but big jobs back-to-back, very high
> `depth`, many keywords, or scheduled runs **without proxies** can make Google **temporarily rate-limit or
> block your IP** (usually clears in minutes–hours; jobs return empty/failed meanwhile). It does **not** ban
> your Google account. Stay safe: **one job at a time, start at `depth 5`, add proxies for large/repeated
> runs.** Scraping Maps is against Google's ToS — use responsibly and follow data law (GDPR/CCPA) for any
> contact data. Full guidance: [Rate limits, bans & responsible use](#️-rate-limits-bans--responsible-use).

---

## What you get

- 🐳 **One-command local setup** — `docker compose up -d` runs the scraper at `http://localhost:8080`.
- 🤖 **A Codex skill** — open this folder in Codex and just say *"scrape coffee shops in Austin"*. Codex does create → poll → download → clean results, following best practices automatically.
- 🛠️ **Standalone scripts** — `scripts/scrape.sh` (bash) and `scripts/scrape.py` (Python, no dependencies) if you'd rather not use Codex.
- 🔒 **Safe by default** — the scraper binds to `127.0.0.1` only, secrets are git-ignored, and the skill enforces rate-limit / legal / data-handling guardrails.

## What it scrapes (and what it doesn't)

- ✅ **Google Maps business listings** → a clean **lead list** by default: `name, phone, email, website, category, address, rating, review count`. The scraper captures ~34 raw fields, but the kit **strips the noise** (geo coordinates, IDs, hours, images, review blobs) so you only get data you can actually use for outreach. Want everything? `scrape.py --full`.
- ➕ **Optional socials** (`--socials`): also pulls each business's **Instagram / Facebook / LinkedIn** by scanning its website. **Token cost: 0** — it runs in code (HTTP + regex), no AI. It only adds ~40–50 tokens *per business* if you later load the rows into an AI chat (≈+2k for 50 leads; nothing if you keep the file on disk). It's slower (one fetch per site) and coverage is partial (only businesses that link socials on their site).
- ❌ **Not** a social-media scraper. It cannot touch Instagram / TikTok / YouTube.

## Why use this instead of asking an AI to "just scrape Google Maps"

Tested head-to-head: a chat AI fetching `maps.google.com` directly hits a **consent wall** and
**JavaScript-rendered** content, so it returns **~0 structured results**. This kit reliably returns
**dozens–hundreds of clean rows per query**. For Google Maps, it's the difference between *can't* and *can*.

---

## Quick start (3 steps)

```bash
# 1. Start the scraper (needs Docker Desktop)
docker compose up -d

# 2. Check it's alive — should print a JSON list (probably empty [])
curl http://localhost:8080/api/v1/jobs

# 3a. Scrape with the script…
./scripts/scrape.sh "coffee shops in Austin TX" 30.2672 -97.7431 5

# 3b. …or open this folder in Codex and say:
#     "Scrape gyms in Miami and give me phones + websites."
```

Full instructions: **[SETUP.md](SETUP.md)**. How Codex uses it:
**[AGENTS.md](AGENTS.md)** and
**[.agents/skills/google-maps-scraper/SKILL.md](.agents/skills/google-maps-scraper/SKILL.md)**.

---

## Using it with Codex

Launch Codex from the repository root so it discovers the project instructions and repository skill:

```bash
cd google-maps-scraper-kit
codex
```

Then ask naturally. The skill can be selected automatically or invoked explicitly as
`$google-maps-scraper`:

- "scrape coffee shops in Austin"
- "find all gyms in Miami with phone numbers"
- "build me a lead list of dentists in Denver, CO"
- "pull Google Maps listings for plumbers in Phoenix"
- "$google-maps-scraper find 50 dental clinics in Embu das Artes and save a CSV"

Codex reads `AGENTS.md` for repository-wide rules and loads the detailed skill only when this workflow is
relevant. It intentionally does **not** use the skill for direct Instagram/TikTok/YouTube scraping.

---

## ⚠️ Rate limits, bans & responsible use

The upstream project's guidance (and ours): *"Please use this scraper responsibly and in accordance with
applicable laws and regulations. Unauthorized scraping may violate terms of service."* There's **no
published ban threshold**, so be conservative.

**What "over-use" looks like** and the risk:
- Big jobs back-to-back, very high `depth`, many keywords at once, or scheduled runs **without proxies**.
- Risk: Google **temporarily rate-limits/blocks your IP** (minutes–hours, then clears). Your Google
  **account is not banned**. While blocked, jobs come back `failed` or with empty/short results.

**Block signals to watch:** jobs returning `failed`, suddenly empty results, or far fewer rows than an
identical earlier run. If you see these, **slow down** — pause, lower `depth`, or add proxies.

**Stay safe:**
- One job at a time; start at `depth 5` and raise only as needed.
- Email extraction is on by default because it is useful for lead generation. Use `--no-email` for a
  deliberately faster run.
- Treat results as **leads to verify**, not a redistributable dataset. Don't resell raw Google data.
- Scraped phones/emails are **personal data** → follow applicable rules such as LGPD/GDPR/CCPA and
  marketing/opt-out requirements if you store or contact them.

**When to add proxies** (*"For larger scraping jobs, proxies help avoid rate limiting"*): large jobs, many
keywords, repeated/scheduled runs, or after you hit block signals. Set the `proxies` array in the job body:
```json
"proxies": ["socks5://user:pass@host:port", "http://host2:port2"]
```
Types: `socks5`, `socks5h`, `http`, `https` (auth optional). The scraper rotates them automatically.
For reference, upstream measures ~120 places/min at `-c 8 -depth 1`.

> The Codex skill warns about temporary rate-limit risk on large jobs, recommends safer settings, and
> refuses clearly abusive uses such as surveillance, harassment, or spam.

---

## What's inside

```
google-maps-scraper-kit/
├── README.md            ← you are here
├── SETUP.md             ← detailed, copy-paste setup (start here)
├── AGENTS.md            ← auto-loaded repository instructions for Codex
├── CLAUDE.md            ← legacy Claude Code compatibility
├── CREDITS.md           ← attribution to the upstream author (gosom)
├── LICENSE              ← MIT (this kit)
├── docker-compose.yml   ← runs the scraper locally on :8080
├── .env.example         ← config template
├── .gitignore
├── scripts/
│   ├── scrape.sh        ← one-shot bash scraper (single keyword)
│   └── scrape.py        ← Python scraper: single, batch, + auto-geocoding (stdlib only)
├── examples/
│   ├── queries.example.json   ← reference job body + coordinate cheatsheet
│   └── queries.example.txt    ← batch keyword list (one per line)
├── .agents/
│   └── skills/
│       └── google-maps-scraper/
│           └── SKILL.md ← Codex workflow, API facts, guardrails, and troubleshooting
└── .claude/             ← original Claude commands and skill kept for compatibility
```

## Credits & license

This kit orchestrates **[google-maps-scraper](https://github.com/gosom/google-maps-scraper)** by
**Georgios Komninos**, used under the MIT License. The kit's own code is MIT licensed too. Full
attribution in **[CREDITS.md](CREDITS.md)**. Please respect Google's Terms of Service and applicable
data-protection law (including LGPD/GDPR/CCPA where relevant) when using scraped data — see the Safety
section in the skill.
