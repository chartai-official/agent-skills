# Chartai Agent Skills

Agent-installable Chartai skill pack for chart context for trading agents.
It teaches an agent how to scan, inspect, confirm visuals, watch symbols, read
feed events, and check usage.

Default flow: call `scan_contexts`, choose a context, then call
`inspect_chart_context`. If the runtime can see the chart, read the visible VC
code and call `confirm_chart_visual_inspection`. Text-only runtimes must report
`visual_unverified`. Use `get_context_ohlcv` only after a context is selected,
when the agent needs the candles behind that chart window for audit; pass
`window: "wide"` for wider data-only context around the same Chart Context. Use
`get_chart` with `variant: "original"` when the agent needs a persistent clean
image containing only wider-context candles, Volume, and the pattern geometry.

## Install

```bash
npx skills add chartai-official/agent-skills --skill chartai --global --yes --copy
```

To install for a specific agent runtime:

```bash
npx skills add chartai-official/agent-skills --skill chartai --agent claude-code --global --yes --copy
```

## Endpoints

Default endpoints:

- Web: `https://chartai.live`
- Skill API: `https://skill.chartai.live`
- Agent key page: `https://chartai.live/app/keys`

Use **subscription** only for Chartai billing plans and renewals. Durable agent
workflows are **watchlists**, **monitors**, and **feed**.

Agent flow: use `scan_contexts` to find current Chart Context, then use
`inspect_chart_context` before making a judgment. Keep the returned `context_id`
as the decision evidence ID. Use `get_record` and `search_records` with
`detection_id` only when you need historical lifecycle records.

Use `search_symbols` and `resolve_symbol` for every user-provided ticker. Chartai
normalizes crypto aliases into provider canonical symbols such as
`BINANCE:TRXUSDT`. Symbol
discovery is not a guarantee that a ready Chart Context exists right now;
`scan_contexts` returns current contexts only when Chartai has a ready native
chart for that symbol/timeframe. No ready context? Chartai can queue a fresh
scan; wait, then retry the same query.
`search_symbols` is paginated across crypto. If
`has_more=true`, call it again with `next_cursor` until `has_more=false`. Do not
treat a first page of 100 as the full catalog.
`list_feed` is also paginated. When `has_more=true`, continue with the returned
`next_cursor` until `has_more=false`; do not treat the first feed page as all
monitor events.

Agent-facing errors include `guidance`; follow `guidance.next_actions` before
changing symbols, timeframes, ids, or action names. Do not guess a fallback
query.

## Related Tools

Install the CLI from GitHub:

```bash
npm install -g github:chartai-official/chartai-cli
chartai connect --target cli
```

Generate MCP config from GitHub:

```bash
npx github:chartai-official/chartai-mcp config
```

## Safety

Chartai returns chart facts and Chart Context. It does not execute trades, hold
exchange keys, or provide position sizing. The agent owns the final decision and
must treat all trading output as research unless the user explicitly connects a
separate execution tool.

If an agent runtime has no visual capability, it must say it did not visually
inspect the chart instead of pretending to have read the image.
