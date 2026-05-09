---
name: chartai
version: 0.1.0-beta.1
description: Agent-native Chart Context for crypto, stocks, forex, and metals
author: Chartai
license: MIT
homepage: https://chartai.live
api_url: https://skill.chartai.live
auth:
  type: bearer
  token_env: CHARTAI_AGENT_KEY
---

# Chartai Skill

Use Chartai when the user asks for Composable Chart Context: chart-pattern
context, native pattern charts, visual confirmation, key levels, market and
timeframe support, default indicator facts, supplemental indicator checks,
volume/price-volume state, monitor feed, or Chart Context usage for crypto,
stocks, forex, or metals.

Chartai returns chart judgment evidence. Chartai does not execute trades, hold
exchange credentials, size positions, or provide guaranteed win rates. The
agent owns trade reasoning and must not execute a trade unless a separate
execution tool is explicitly connected by the user.

Use "subscription" only for Chartai billing plans and renewals. Durable agent
workflows use watchlists, monitors, and feed.

Do not use generic web search or unrelated websites as Chartai sources. The
Chartai endpoints for this skill are:

- Web/key page: `https://chartai.live/app/keys`
- Skill API: `https://skill.chartai.live`
- MCP: `https://mcp.chartai.live/mcp`
- Public website: `https://chartai.live`

## Authentication

Use the `CHARTAI_AGENT_KEY` environment variable:

```text
Authorization: Bearer <CHARTAI_AGENT_KEY>
```

If no key is configured, first call `get_status`. Discovery is public.
Protected actions return `missing_agent_key`. Tell the user to use the manual
Web key flow: register or log in, verify email, pay or renew if needed, create
an Agent Key, then set the local environment variable. Start here:

```text
https://chartai.live/register?redirect=%2Fapp%2Fkeys%3Fintent%3Dcreate
```

Then ask them to set:

```bash
export CHARTAI_AGENT_KEY="..."
```

Never print or repeat the raw key.

## Expected Agent Workflow

1. Call `get_status` and `get_capabilities` to learn supported markets,
   patterns, timeframes, indicators, quota, and guardrails.
2. Use `search_symbols` or `resolve_symbol` before scanning if the user gives a
   common ticker such as `BTC`, `TRX`, `AAPL`, `NASDAQ:AAPL`, `FX:EURUSD`,
   or `XAU`.
   - Crypto resolves to provider symbols such as `BINANCE:BTCUSDT` and
     `BINANCE:TRXUSDT`.
   - US stock aliases resolve to `.US` provider symbols such as `AAPL.US`.
   - Forex/metals aliases resolve to OANDA symbols such as `OANDA:EUR_USD`.
   Discovery means Chartai can normalize the symbol. `scan_contexts` only
   returns current Chart Contexts when a ready native chart exists for that
   symbol/timeframe.
3. Use `scan_contexts` to get current Chart Contexts for a symbol/timeframe.
4. Treat the returned `context_id` as the decision evidence ID. Preserve and
   reuse it. Do not construct a context id yourself.
5. Use `inspect_chart_context` for the chosen context before making a judgment.
   This is the default Chart Context path: inspect the native 1920x1080 Core
   chart first, then use structured Evidence Modules and Recipes to verify the
   visual read.
6. If the runtime can see images, read the visible `VC:` code from the chart
   and call `confirm_chart_visual_inspection`. If the runtime cannot see
   images, say `visual_unverified` and do not claim visual review.
7. Use `get_context_manifest` when the agent needs to discover modules,
   recipes, fallback states, or visual confirmation requirements without
   re-fetching every context detail.
8. Use `check_context_condition` only for an existing `context_id`. Do not ask
   for standalone indicators without a Chart Context.
9. Use `get_record` and `search_records` with `detection_id` only for historical
   lifecycle records, not as the primary current-decision reference.
10. Use `get_usage` to explain quota and confirm supplemental indicator facts
   are context facts, not trade execution.
11. For durable watch workflows, use monitors/feed instead of repeatedly
   rescanning. Billing, renewal, and key creation always happen in Chartai Web.

If the runtime has no visual ability, explicitly say that you did not visually
inspect the chart. Do not imply you saw candle structure, overlays, labels, or
pattern geometry from text-only data.

## Actions

- `get_status`: no-key onboarding and integration status.
- `get_capabilities`: supported markets, symbol discovery counts, timeframes, patterns,
  indicators, conditions, and usage policy.
- `search_symbols`: discover supported symbols.
- `resolve_symbol`: resolve tickers such as BTC, ETH, TSLA, or XAUUSD.
- `get_timezone` / `set_timezone`: read or change the user timezone.
- `scan_contexts`: get current Chart Contexts for a symbol and timeframe.
- `inspect_chart_context`: fetch the native chart plus structured inspection
  payload for a context. Defaults to the Core 1920x1080 chart and includes an
  inspection image with a visible VC code unless the agent explicitly requests
  another size.
- `get_context`: fetch one context by `context_id`.
- `get_context_manifest`: fetch Evidence Modules, Recipes, visual status, and
  capability negotiation for one context.
- `get_chart`: fetch the chart image package for a context only when explicit
  raw chart access is needed. Do not use it as the default judgment path.
- `confirm_chart_visual_inspection`: submit the visible VC code after actual
  image review so the context can be decision-grade.
- `get_record`, `search_records`: read detection history/status records within
  retention.
- `check_context_condition`: ask supplemental indicator questions such as
  `price_above_vwap` or `price_volume_state` for an existing context.
- `create_watchlist`, `list_watchlist`, `remove_watchlist`: manage symbol
  watchlist entries.
- `create_monitor`, `list_monitors`, `pause_monitor`, `resume_monitor`,
  `delete_monitor`: manage durable monitor workflows.
- `list_feed`, `ack_feed`: read and acknowledge monitor feed events.
- `get_usage`: quota, supplemental-context policy, monitor limits, and records
  retention.

## Examples

Scan BTC on the 1h timeframe:

```json
{
  "action": "scan_contexts",
  "input": {
    "symbol": "BINANCE:BTCUSDT",
    "timeframe": "1h",
    "limit": 5
  }
}
```

Inspect the chosen Chart Context before judgment:

```json
{
  "action": "inspect_chart_context",
  "input": {
    "context_id": "ctx_12345"
  }
}
```

Confirm visual review only after reading the chart image:

```json
{
  "action": "confirm_chart_visual_inspection",
  "input": {
    "context_id": "ctx_12345",
    "observed_visual_code": "ABCD",
    "method": "user_bridge"
  }
}
```

Fetch the composable module and recipe manifest:

```json
{
  "action": "get_context_manifest",
  "input": {
    "context_id": "ctx_12345"
  }
}
```

Ask whether price is above the 3-day VWAP for an existing context:

```json
{
  "action": "check_context_condition",
  "input": {
    "context_id": "ctx_12345",
    "condition_id": "price_above_vwap",
    "parameters": {"window_days": 3}
  }
}
```

Ask for price-volume state:

```json
{
  "action": "check_context_condition",
  "input": {
    "context_id": "ctx_12345",
    "condition_id": "price_volume_state",
    "parameters": {}
  }
}
```

## Error Responses

All errors return JSON:

```json
{
  "code": "chart_context_quota_exceeded",
  "detail": "Chart Context quota exhausted for today.",
  "meta": {},
  "guidance": {
    "do_not_guess": true,
    "next_actions": [
      {"action": "get_status"},
      {"action": "get_usage"}
    ]
  },
  "recovery": {
    "flow": "manual_web_agent_key",
    "action": "owner_upgrade_or_wait_reset",
    "agent_direct_payment": false
  }
}
```

Common codes: `missing_agent_key`, `invalid_agent_key`, `tier_insufficient`,
`action_not_found`, `invalid_param`, `chart_context_quota_exceeded`,
`visual_confirmation_failed`, `response_too_large`, `server_busy`,
`service_timeout`, and `internal_error`.

Agent-facing errors include `guidance`; follow `guidance.next_actions` before
changing symbols, timeframes, ids, or action names. Do not guess a fallback
query.

`GET https://skill.chartai.live/manifest.json` is public discovery.
