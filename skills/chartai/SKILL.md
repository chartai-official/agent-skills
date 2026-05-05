---
name: chartai
version: 0.1.0-beta.1
description: Agent-native Chart Context for crypto, stocks, forex, and commodities
author: Chartai
license: MIT
homepage: https://test.chartai.live
api_url: https://skill-staging.chartai.live
auth:
  type: bearer
  token_env: CHARTAI_AGENT_KEY
---

# Chartai Skill

Use Chartai when the user asks for chart-pattern context, pattern status,
pattern charts, market/timeframe support, indicator facts, volume/price-volume
state, monitor feed, or Chart Context usage for crypto, stocks, forex, or
commodities.

Chartai returns factual chart context. Chartai does not execute trades, hold
exchange credentials, size positions, or provide guaranteed win rates. The
agent owns trade reasoning and must not execute a trade unless a separate
execution tool is explicitly connected by the user.

Do not use `chartai.trade` as a Chartai source. The beta endpoints for this
skill are:

- Web/key page: `https://test.chartai.live/app/keys`
- Skill API: `https://skill-staging.chartai.live`
- MCP: `https://mcp-staging.chartai.live/mcp`
- Public website: `https://test.chartai.live`

## Authentication

Use the `CHARTAI_AGENT_KEY` environment variable:

```text
Authorization: Bearer <CHARTAI_AGENT_KEY>
```

If no key is configured, first call `get_status`. If the action returns
`missing_agent_key`, tell the user to open:

```text
https://test.chartai.live/app/keys
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
   common ticker such as `BTC`, `ETH`, `TSLA`, or `XAU`.
3. Use `scan_contexts` to get current Chart Contexts for a symbol/timeframe.
4. Use `get_context` for the chosen context before making a judgment.
5. Use `check_context_condition` only for an existing `context_id`. Do not ask
   for standalone indicators without a Chart Context.
6. Use `get_chart` if the user or agent needs the chart artifact.
7. Use `get_usage` to explain quota and confirm supplemental indicator facts
   are context facts, not trade execution.
8. For subscriptions, use monitors/feed instead of repeatedly rescanning.

## Actions

- `get_status`: no-key onboarding and integration status.
- `get_capabilities`: supported markets, symbols, timeframes, patterns,
  indicators, conditions, and usage policy.
- `search_symbols`: discover supported symbols.
- `resolve_symbol`: resolve tickers such as BTC, ETH, TSLA, or XAUUSD.
- `get_timezone` / `set_timezone`: read or change the user timezone.
- `scan_contexts`: get current Chart Contexts for a symbol and timeframe.
- `get_context`: fetch one context by `context_id`.
- `get_chart`: fetch the chart image package for a context.
- `get_record`, `search_records`: read detection history/status records within
  retention.
- `check_context_condition`: ask supplemental indicator questions such as
  `price_above_vwap` or `price_volume_state` for an existing context.
- `create_watchlist`, `list_watchlist`, `remove_watchlist`: manage symbol
  watchlist entries.
- `create_monitor`, `list_monitors`, `pause_monitor`, `resume_monitor`,
  `delete_monitor`: manage monitor subscriptions.
- `list_feed`, `ack_feed`: read and acknowledge monitor feed events.
- `get_usage`: quota, supplemental-context policy, monitor limits, and records
  retention.

Legacy aliases remain available for older agents: `scan`, `list_symbols`,
`list_patterns`, `get_quota`, and `add_watchlist`.

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
  "meta": {}
}
```

Common codes: `missing_agent_key`, `invalid_agent_key`, `tier_insufficient`,
`action_not_found`, `invalid_param`, `chart_context_quota_exceeded`,
`response_too_large`, `server_busy`, `service_timeout`, and `internal_error`.

`GET https://skill-staging.chartai.live/manifest.json` is public discovery.

