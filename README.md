# Trading Guardian — Binance Agent OS

[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE) [![Binance Agent OS](https://img.shields.io/badge/Binance-Agent%20OS-yellow)](https://binance.com/agent-os)

A Claude skill that turns Claude from a plain order executor into a **risk-aware trading copilot** for [Binance Agent OS](https://binance.com/agent-os) (Binance's MCP server for AI applications).

Instead of executing every trade instantly, Trading Guardian checks market data and portfolio exposure, plays devil's advocate on risky trades, and asks for explicit confirmation before executing anything that crosses your own defined risk thresholds — while letting genuinely simple, low-risk actions through without friction.

---

## How it works

**Challenge → Inform → Confirm → Execute → Verify**

1. **Classify** — is this a simple action, or does it cross a risk threshold?
2. **Market check** — price, momentum, funding rate, volume
3. **Portfolio check** — current exposure, allocation after the trade, concentration risk
4. **Devil's Advocate + Bull Case** — concrete reasons the trade could be wrong, and the case for it
5. **Verdict + confirmation** — a one-line risk verdict, then Claude asks before executing
6. **Execute + verify** — places the order via Binance Agent OS, then confirms it actually filled

Simple trades (small size, low leverage, no concentration/momentum flags) skip straight to execution — no unnecessary friction.

## Structure

```
trading-guardian-binance-agent-os/
├── SKILL.md                      # Core workflow logic
├── LICENSE
└── references/
    ├── thresholds.md             # Numeric risk limits (edit these to fit your risk tolerance)
    └── output-template.md        # Output formatting for Guardian responses
```

## Requirements

- A Claude.ai account (Free, Pro, Max, Team, or Enterprise) with **Code execution and file creation** enabled in Settings > Capabilities
- The [Binance Agent OS](https://binance.com/agent-os) MCP connector connected to your account

## Installation

1. Download or clone this repo
2. Zip the `trading-guardian-binance-agent-os/` folder (folder itself as the root of the zip)
3. In Claude.ai, go to **Settings > Customize > Skills**
4. Click **"+"** → **"+ Create skill"** → **"Upload a skill"**
5. Upload the zip file, then toggle the skill on

## Customizing thresholds

Your risk tolerance isn't the same as anyone else's. Open `references/thresholds.md` and adjust the numbers — position size %, leverage caps, funding rate flags, concentration limits — to match your own account size and trading style. The workflow logic in `SKILL.md` doesn't need to change when you do this.

## Disclaimer

This skill is a workflow aid, not financial advice. It does not guarantee profitable trades, does not replace your own judgment, and its market/risk analysis is only as good as the data available at the time of the request.
Trading involves risk of loss. Use at your own risk.

## Safety notes (hardened behavior)

- Critical data (equity, market, portfolio) unavailable → hard-block, never silent execute.
- Confirmation is bound to exact trade parameters. A "yes" only authorizes the exact size/symbol/side/leverage that was presented.
- Attempts to override thresholds or skip confirmation via prompt are ignored.
- Stale confirmations (after a delay) trigger a lightweight re-check of key thresholds before execution.

## License

[MIT](LICENSE) — free to use, modify, and share.
