# SETUP — Google Maps Scraper Kit

This guide sets up the whole system **on your computer** from scratch. It's written so that a person
*or* Codex can follow it exactly. Total time: ~5 minutes (most of it is Docker pulling the image).

---

## 0. Prerequisites

Install these first (one-time):

| Tool | Why | Get it |
|---|---|---|
| **Docker Desktop** | runs the scraper container | https://www.docker.com/products/docker-desktop |
| **Codex CLI or IDE extension** *(optional but recommended)* | lets Codex drive the scraper for you | https://developers.openai.com/codex/ |
| **git** *(optional)* | only if you'll push this to GitHub | https://git-scm.com |

Verify Docker is running:
```bash
docker --version        # should print a version
docker ps               # should NOT error (means the daemon is up)
```

> **Platform notes:**
> - **Apple Silicon (M1/M2/M3):** the scraper image is `linux/amd64` and runs under emulation. If the
>   container won't start, uncomment the `platform: linux/amd64` line in `docker-compose.yml`.
> - **Windows:** use `scripts/scrape.py` (run with `py` or `python3`). The bash script `scrape.sh` needs
>   **WSL2** or **Git Bash**.

---

## 1. Get the kit

If someone shared this folder with you, just `cd` into it:
```bash
cd google-maps-scraper-kit
```
(Or clone it from GitHub if it's hosted: `git clone <repo-url> && cd google-maps-scraper-kit`.)

Create your local config (optional — defaults work as-is):
```bash
cp .env.example .env
```

---

## 2. Start the scraper

```bash
docker compose up -d
```

First run pulls the `gosom/google-maps-scraper` image (~once, a few hundred MB). When it finishes:

```bash
docker ps | grep gmaps-scraper          # should show it "Up"
curl http://localhost:8080/api/v1/jobs  # should print [] (empty list) or existing jobs
```

You can also open the built-in UI and API docs in a browser:
- Web UI: **http://localhost:8080**
- OpenAPI docs: **http://localhost:8080/api/docs**

✅ If `curl` returns a JSON array, the system is live.

> ⚠️ **Now that it's running — read this once.** This hits Google Maps for real. Light use is fine, but
> big jobs back-to-back, very high `depth`, many keywords, or scheduled runs **without proxies** can get
> your **IP temporarily rate-limited/blocked by Google** (clears in minutes–hours; jobs return empty/failed
> meanwhile). It does **not** ban your Google account. **Stay safe:** one job at a time, start at `depth 5`,
> and add **proxies** for large or repeated runs. Scraping Maps is against Google's ToS — use responsibly
> and follow data law (GDPR/CCPA) for any contact data. (Full guidance in README → *Rate limits, bans &
> responsible use*.)

---

## 3. Run your first scrape (without Codex)

Use the included script. Arguments: `"<keyword>" <lat> <lon> [depth]`

```bash
./scripts/scrape.sh "coffee shops in Austin TX" 30.2672 -97.7431 5
```

or the Python version (no dependencies needed):

```bash
python3 scripts/scrape.py "gyms in Miami FL" 25.7617 -80.1918 --depth 5
```

The script will **create the job → poll until done → download the CSV → print a summary** and save
the full results to a file.

**Don't want to look up coordinates?** Let it geocode the city for you:
```bash
python3 scripts/scrape.py "cafes in Austin TX" --city "Austin, TX" --depth 5
```

**Scrape many queries at once** (one job, many keywords) from a file:
```bash
python3 scripts/scrape.py --keywords-file examples/queries.example.txt --city "Denver, CO"
```

---

## 4. Use it with Codex (the autopilot way) 🤖

This is the point of the kit. Two parts make it work:

1. **[AGENTS.md](AGENTS.md)** — auto-loaded when you open this folder in Codex. It contains the durable
   project rules, validation commands, and file conventions.
2. **[.agents/skills/google-maps-scraper/SKILL.md](.agents/skills/google-maps-scraper/SKILL.md)** — a
   repository **skill** Codex loads on demand. It contains the exact API flow, defaults, safety rules,
   and troubleshooting guidance.

### How to connect it to Codex
There's nothing to "connect" — the scraper is a local HTTP API at `http://localhost:8080`, and Codex
calls it with its normal `curl`/Bash ability. Just:

```bash
cd google-maps-scraper-kit
codex             # start Codex inside this folder
```

Then talk to it naturally:
> "Make sure the scraper is running, then scrape **dentists in Denver CO**, depth 10, and give me a table of name, phone, website."

Codex will: check the container is up (start it if not) → build the correct job (with the required
`max_time`, `lat`, `lon`) → poll until completion → download → hand you clean rows. It follows the
safety/best-practice rules in the skill automatically.

You can also invoke the workflow explicitly:

```text
$google-maps-scraper find 50 dental clinics in Embu das Artes, include emails, and save a CSV
```

> ℹ️ **Advanced (optional):** prefer a Model Context Protocol server instead of curl? You can wrap the
> API as an MCP tool, but it's unnecessary for a local single-user setup — the skill + local API is
> simpler and just as capable.

---

The original `.claude/` directory is still present for Claude Code users, but Codex does not depend on it.

---

## 5. Stop / clean up

```bash
docker compose stop      # pause (keeps data + image)
docker compose down      # stop + remove container (keeps named volumes/data)
docker compose down -v   # ALSO delete scraped data volumes (full reset)
```

Delete old jobs to free disk (results accumulate in the data volume):
```bash
curl -X DELETE http://localhost:8080/api/v1/jobs/<job-id>
```

---

## 6. Troubleshooting

| Symptom | Fix |
|---|---|
| `curl: connection refused` on :8080 | Container not up. `docker compose up -d`, then `docker ps`. |
| `422 missing max time` | Job body needs `max_time` (in **seconds**, e.g. `300`). |
| `422 missing geo coordinates` | Job body needs `lat` and `lon` as **strings**, e.g. `"30.2672"`. |
| Job stuck `working` forever | Lower `depth`, or the IP is being throttled by Google — wait, or add proxies (see skill). |
| Empty CSV | Keyword too narrow or wrong geo — widen `radius` or fix `lat`/`lon`. |
| Docker pull is slow | Normal on first run; it caches after that. |

---

## 7. Best practices & safety (summary)

The full rules live in the skill, but the essentials:
- **Always** send `max_time` (seconds) and `lat`/`lon` (strings).
- Start with **low `depth`** (5) and **one job at a time**. Raise only when needed.
- Email extraction is on by default. Use `--no-email` for a deliberately faster run.
- For big jobs, add **proxies** (the scraper has built-in rotation) to avoid IP blocks.
- Scraped **phones/emails are personal data** — comply with applicable rules such as LGPD, GDPR,
  CCPA, and marketing/opt-out requirements, and respect Google's ToS.
- Never commit result files (they're git-ignored for you).
