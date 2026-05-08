# Chartai Agent Skills

Agent-installable Chartai skill pack for chart context for trading agents.
It teaches an agent how to scan, inspect, confirm visuals, watch symbols, read
feed events, and check usage.

Default flow: call `scan_contexts`, choose a context, then call
`inspect_chart_context`. If the runtime can see the chart, read the visible VC
code and call `confirm_chart_visual_inspection`. Text-only runtimes must report
`visual_unverified`.

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
