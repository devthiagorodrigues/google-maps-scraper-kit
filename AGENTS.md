# AGENTS.md — Google Maps Scraper Kit

## Project purpose

This repository runs `gosom/google-maps-scraper` locally through Docker and provides small wrappers for
creating jobs, polling them, and exporting clean lead lists. The REST API is expected at
`http://localhost:8080` and must remain bound to localhost unless an authenticated reverse proxy is added.

## Repository workflow

- For Google Maps business searches, lead lists, job management, or scraper troubleshooting, load and
  follow `.agents/skills/google-maps-scraper/SKILL.md`.
- Prefer `python3 scripts/scrape.py` for normal single and batch searches. Use the raw REST API only when
  the script cannot express a requested option, such as proxies or a custom job body.
- Start with `depth 5`, one job at a time, and email extraction enabled unless the user requests a fast
  run without emails.
- Save result files to disk and show only a count plus a small sample in chat. Result files may contain
  personal data and are intentionally ignored by Git.
- Never expose the unauthenticated API publicly. Keep the Docker port mapping on `127.0.0.1`.
- Do not remove or alter `LICENSE` or `CREDITS.md`; this kit wraps the MIT-licensed upstream project.

## Useful commands

```bash
# Validate the Python CLI without running a scrape
python3 scripts/scrape.py --help

# Validate the Bash wrapper
bash -n scripts/scrape.sh

# Validate the Compose file
docker compose config

# Start and health-check the local API
docker compose up -d
curl --fail --silent http://localhost:8080/api/v1/jobs
```

## Editing expectations

- Keep the Python CLI dependency-free unless a new dependency is clearly justified.
- Preserve CSV as the default output and the concise lead-field set unless the user explicitly requests
  full data.
- Update `README.md`, `SETUP.md`, and the repository skill together when commands or behavior change.
- Run the relevant validation commands after editing scripts, Compose configuration, or the skill.
