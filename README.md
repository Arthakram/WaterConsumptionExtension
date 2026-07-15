# AquaAI — ChatGPT Water Footprint (Chrome Extension, MV3)

Visualises the **estimated** water consumption associated with each ChatGPT
query, to raise awareness of AI's environmental impact.

> ⚠️ All figures are **estimates**, not measurements. They do not represent
> exact water usage. See *Water model* below.

## What it does

1. **Detects** interactions on `chatgpt.com` / `chat.openai.com`.
2. **Estimates** water per query from response length and a research-based model.
3. **Displays** a small 💧 icon docked on the prompt bar (click to expand
   Last / Today / All-time details).
4. **Tracks** cumulative usage (per-day + all-time) in `chrome.storage.local`,
   with a toolbar badge and a popup dashboard.
5. **Backfills history**: on load it tallies the messages already in the open
   conversation into your all-time total, de-duplicated by message id so
   reloads never double-count. Each conversation is added the first time you
   open it.

> **Scope of "total":** ChatGPT only loads the *currently open* conversation
> into the page, so the extension can only count conversations you actually
> open — there is no public API to read your entire account history at once.
> Backfilled history counts toward **All-time**, not **Today** (its real date
> isn't reliably available in the DOM).

## Architecture

| File | World | Role |
|------|-------|------|
| `manifest.json` | — | MV3 manifest, permissions, content-script wiring |
| `src/estimator.js` | content + popup | Pure water model (`AquaAIEstimator`) |
| `src/injected.js` | page main world | Hooks `[CHATGPT_API_ENDPOINT]` for an in-flight signal |
| `src/content.js` | content isolated world | **Source of truth**: DOM observer, estimation, storage, overlay |
| `src/background.js` | service worker | Defaults on install + toolbar badge |
| `src/popup.{html,css,js}` | popup | Dashboard + tunable model settings + reset |
| `styles/overlay.css` | content | Floating overlay styling (light/dark) |
| `icons/` | — | Placeholder icons (regenerate via `tools/make_icons.py`) |

### Detection strategy

The **DOM `MutationObserver`** is the single source of truth: it watches for
completed assistant messages (`[data-message-author-role="assistant"]`),
de-duplicates by `data-message-id`, and finalises a message after
`STREAM_IDLE_MS` of no changes (a "finished streaming" heuristic). This is
robust to API changes and never double-counts.

`injected.js` additionally hooks the network layer
(`[CHATGPT_API_ENDPOINT]` = `/backend-api/conversation`) purely to show a
"calculating…" state early — it does **not** count, so the two paths can't
conflict.

## Water model  `[WATER_CONSUMPTION_MODEL_DETAILS]`

```
water_mL = BASE_ML_PER_QUERY + (responseTokens / 1000) * ML_PER_1K_TOKENS
responseTokens ≈ responseChars / CHARS_PER_TOKEN
```

Defaults (`[WATER_CONSUMPTION_FACTOR]`, all user-tunable in the popup):

| Constant | Default | Meaning |
|----------|---------|---------|
| `BASE_ML_PER_QUERY` | `5` mL | fixed cooling/overhead per request |
| `ML_PER_1K_TOKENS` | `30` mL | marginal water per 1,000 generated tokens |
| `CHARS_PER_TOKEN` | `4` | tokenizer approximation |
| `ASSUMED_TOKENS_WHEN_UNKNOWN` | `300` | used when length can't be read |

**Basis:** Li, P., Yang, J., Islam, M. A., & Ren, S. (2023), *"Making AI Less
Thirsty"* (arXiv:2304.03271) — a short GPT-3-class session of ~20–50 medium
responses consumes on the order of ~500 mL of freshwater (on-site cooling +
off-site power generation), i.e. roughly ~10–25 mL/response. The defaults land
a typical ~300-token answer at ~14 mL. Real usage varies widely by model, data
centre WUE, and grid — hence everything is an editable assumption.

## Install (unpacked)

1. `chrome://extensions` → enable **Developer mode**.
2. **Load unpacked** → select this folder.
3. Open ChatGPT and send a message; the pill updates bottom-right.

Regenerate icons: `python3 tools/make_icons.py`.
```
# WaterConsumptionExtensioon
