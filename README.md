# Chartai Agent Skills

Agent-installable Chartai skill pack for Chart Context research.

This repository is the public GitHub source used by agent runtimes and the
`skills` CLI. It teaches an agent how to use Chartai; it does not contain the
Chartai backend, vendor keys, or private API implementation.

Default agent flow is visual-first: after `scan_contexts`, agents must call
`inspect_chart_context` for the selected context, visually read the native Core
chart, then use structured Chart Context fields to verify the judgment.

## Install

```bash
npx skills add chartai-official/agent-skills --skill chartai --global --yes --copy
```

To install for a specific agent runtime:

```bash
npx skills add chartai-official/agent-skills --skill chartai --agent claude-code --global --yes --copy
```

## Runtime Endpoints

This beta skill points at Chartai staging:

- Web: `https://test.chartai.live`
- Skill API: `https://skill-staging.chartai.live`
- Agent key page: `https://test.chartai.live/app/keys`

Production endpoints will be enabled only after Chartai production launch is
approved.

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
