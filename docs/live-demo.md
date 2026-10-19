# Live WebShop demo

The live demo is one web page. On it, an agent learns a shopping task once,
saves the recipe in MongoDB, and races a memoryless browsing agent.

## One-time setup

1. Install dependencies: `uv sync`, then `uv run playwright install chromium`.
2. Set up WebShop next to this repo: `scripts/setup_webshop.sh`.
3. Sign in to Codex once: `codex login`. The model proxy uses this login.
4. Put `MONGODB_URI` (and optionally `MONGODB_DB`) in `.env`.
5. Optional: set `DEMO_BROWSER` in `.env` to the path of another
   Chromium binary for the visible agent windows. Without it, the windows
   use Playwright's "Google Chrome for Testing", which is the recommended
   choice. Regular Google Chrome breaks the Toolshop browsing agent: its
   pages never reach network-idle, so every page load times out.

## Run

```bash
scripts/start_demo.sh
```

The script starts each service that is not already running:

| Service | Address | Log |
|---|---|---|
| WebShop | `http://127.0.0.1:3000` | `.recordings/logs/webshop.log` |
| Codex model proxy | `http://127.0.0.1:18080` | `.recordings/logs/proxy.log` |
| Live demo page | `http://127.0.0.1:8790` | `.recordings/logs/demo.log` |

It then prints a status line and opens the demo page. All three values must
be `True`:

```text
Status: {'webshop': True, 'proxy': True, 'mongodb': True}
```

WebShop takes a few seconds to load its search index on first start. If it
shows `False`, wait and check `http://127.0.0.1:8790/api/status` again.

## Use the page

1. Pick a site (WebShop or Toolshop) and a task.
2. Keep **Show in browser windows** checked to watch the agents work.
3. Click **Teach** to record one run and save the learned recipe to MongoDB.
4. Click **Race on this task**. The recipe agent replays the recipe from
   MongoDB. The browsing agent clicks through the site with no memory.
5. Click **Forget recipes** to retire the saved recipes and start over.
   MongoDB keeps their history.

## Restart after code changes

The script skips services that already run. Stop the demo page first:

```bash
pkill -f demo/webshop_live.py
scripts/start_demo.sh
```

## Demo browser windows do not appear or stay blank

macOS can open the automation browser behind other windows or in another
Space. Check the Dock for "Google Chrome for Testing". Do not pass
`--force-device-scale-factor` to the browser: Chrome then paints a blank
window. `demo/visible.py` zooms pages with CSS instead.

## Toolshop browsing agent makes 0 model calls

Its page load timed out before the first model call. Remove `DEMO_BROWSER`
from `.env` if it points at regular Google Chrome, then restart the demo page.
