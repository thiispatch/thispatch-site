# Hechi Web — Marketing Site & Operator Dashboard

Two **self-contained**, **PlayStation-styled** (see root [`DESIGN.md`](../DESIGN.md)),
**bilingual (KO/EN)** HTML pages. No build step, no external resources
(CSP-safe, work fully offline).

## Files
- `index.html` — public marketing / project showcase (hybrid: story chapters + an embedded interactive detection preview).
- `hechi-dashboard.html` — operator console (Overview hub + Threats + Policies & Rules + Allowlists & Settings).

## Run / preview
From the repo root:
```bash
python3 -m http.server 8787 --directory web
#   http://localhost:8787/index.html
#   http://localhost:8787/hechi-dashboard.html
```
Or open a file directly (WSL: `explorer.exe web/index.html`). Each file also opens standalone by double-click.

## `index.html` (marketing)
Chapter-rhythm landing (dark → white → … → blue), KO/EN toggle (persists via `localStorage`), reveal-on-scroll. Sections: nav · hero · principles (hot/cold path, kernel≠user rules, append-first, allowlist carve-out) · 3-stage pipeline (kernel eBPF → userspace Go → GNN+LLM) · attack anatomy (npm kill-chain) · **interactive detection console preview** (3 sessions) · GNN · team/roles · footer.

## `hechi-dashboard.html` (operator console)
Dark sidebar + light workspace. **Hash-routed** (`#overview` / `#threats` / `#policies` / `#settings`, bookmarkable). KO/EN toggle. Overview is a **hub**: clickable KPI cards and recent-detection rows deep-link into the relevant view.
- **Overview** — route-distribution donut, sensor health (BPF objects, ring buffer, drops, throughput sparkline), recent detections.
- **Threats** — search + route filters, queue, full detail (route / action / severity, GNN score ring, Tier-0/1/2 findings, blast radius, session timeline, provenance graph, mock triage actions).
- **Policies & Rules** — Monitor-mode (dry-run) state, kernel-policy table, userspace rules (R7 etc.), GNN-threshold slider, **live YAML preview**, protected policy save + loader status polling.
- **Allowlists & Settings** — CIDR sets (add/remove), `agent.yaml` fields.

## Design basis & deliberate deviations (from `DESIGN.md`)
PlayStation tokens: blue `#0070d1`, fully-rounded pill CTAs, 8px cards, weight-300 display type, three-canvas rhythm, flat-on-canvas.
- **Mono** used only inside the console + inline technical tokens (DESIGN.md says "no monospace" for marketing chrome — a justified domain deviation for a security tool).
- **Blue kept scarce** (primary CTAs + footer + small accents only).
- **Severity / route accents stay in palette**: red `#c81b3a` = alert/critical, blue = gnn_gated, gray = context (no invented green/amber).
- **Fonts**: system stack approximating PlayStation SST at weight 300 (SST is proprietary; no web fonts → CSP-safe/offline; Korean falls back to Noto Sans KR / Apple SD Gothic / Malgun).
- **Default language: Korean** (one-line change to English in the `lang` init).

## Live data and policy writes
`cmd/hechi-web` serves the dashboard and exposes live JSON APIs for threats,
overview counts, userspace rules, allowlists, runtime config, reports, and kernel
policy files. If the backend is not reachable, `hechi-dashboard.html` keeps using
its built-in seed data so the page still opens standalone.

Kernel policy update is opt-in:

```bash
TP_POLICY=policies TP_POLICY_STATUS_PATH=data/policy_reload_status.json ./thispatch

HECHI_WEB_POLICY_WRITE=1 \
HECHI_WEB_POLICY_TOKEN='change-me' \
HECHI_WEB_POLICY_ORIGIN='http://localhost:8787' \
go run ./cmd/hechi-web -repo . -data data -web web
```

The dashboard writes only `policies/*.yaml` / `policies/*.yml`. It does not touch
BPF maps directly. Saving a policy performs a protected HTTP write to
`hechi-web`; the final YAML rename is then observed by the loader's inotify
watcher, which reloads without restart. The dashboard polls
`/api/policy/status` to distinguish "file saved" from "loader applied".

`monitor:true` is global across the assembled enabled policy set: if any enabled
policy file requests monitor mode, the whole kernel policy load is downgraded to
observe-only LOG behavior. Disabled files (`enabled:false`) do not contribute
their monitor flag.

UI ↔ engine vocabulary (already modeled in the mock data):
- kernel action: `LOG` / `DENY` / `KILL`
- route: `alert` / `gnn_gated` / `context`
- findings: Tier-0 `kernel_verdict`, Tier-1 rules (R7 allowlist carve-out, sensitive/secret read…), Tier-2 `correlator`
- identity: `exec_id = (pid, start_time)`, `followed` lineage

## i18n pattern (if you extend the UI)
`T = { 'key': {ko, en}, … }`; static text via `data-i18n="key"` (or `data-i18n-html` when the value contains markup); JS reads `t('key')`. The KO/EN toggle calls `setLang()` → re-applies static text + re-renders dynamic views. Dynamic data (alerts, rules) carries its own `{ko, en}` fields. Choice persists in `localStorage` under `hechi-lang`.

## Next steps / TODO
- Real team member names + GitHub / Docs links (currently placeholders `#`).
- Optional: default to English; add an event-log/replay view and a process-tree explorer.
- If the UI grows, consider migrating to a framework (React/Vite); the i18n setup and data shapes port cleanly.

## Session work log (handoff)
1. **Architecture sweep** of the kernel (eBPF), userspace (Go cold path), and GNN layers → full context in [`../docs/00_Hechi_Project_Context.md`](../docs/00_Hechi_Project_Context.md) (also mirrored in the sibling `thispatch/documents/` folder). Read it for the complete engine model.
2. **Marketing page** (`index.html`) — hybrid landing + embedded console preview.
3. **Operator dashboard** (`hechi-dashboard.html`) — the four-view console described above.
4. Moved both into this `web/` folder on the `feat/web-frontend` branch.

## Related
- Root [`DESIGN.md`](../DESIGN.md) — the PlayStation design system these pages follow.
- [`../docs/00_Hechi_Project_Context.md`](../docs/00_Hechi_Project_Context.md) — full kernel / userspace / GNN architecture context (also mirrored in `thispatch/documents/`).
- [`../docs/README.md`](../docs/README.md) — engine documentation index.
