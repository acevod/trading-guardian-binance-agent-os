# Trading Guardian Binance Agent OS

A risk-aware Claude Skill that sits between a user's trading request and Binance Agent OS MCP execution.

## What it does

Trading Guardian evaluates the complete available risk state before classifying a trade as Simple, Light Guardian, or Full Guardian. It is designed to challenge risky trades without preventing a user from making their own decision.

### Safety flow

```text
Bind intent
  ↓
Gather complete risk state
  ↓
Evaluate all applicable thresholds
  ↓
Classify
  ├─ Simple ───────────────┐
  └─ Light / Full → Confirm│
                           ↓
                   Final revalidation
                           ↓
                        Execute
                           ↓
                         Verify
```

A Simple trade is **not** exempt from market, portfolio, concentration, or final-state checks. It only means the complete evaluation found no tier-producing threshold requiring Guardian confirmation.

## Files

```text
trading-guardian-binance-agent-os/
├── SKILL.md
├── README.md
├── LICENSE
└── references/
    ├── thresholds.md
    ├── execution-safety.md
    └── output-template.md
```

## Requirements

- Claude with Skills support
- Binance Agent OS MCP connected and authorized for the requested trading operation

This repository does not implement the Binance API, database, authentication service, frontend, or execution backend itself. Those components remain outside this Skill's codebase and must be secured independently.

## Safety properties

- Fail closed when required equity, market, or portfolio data is unavailable or not fresh enough.
- Evaluate all applicable risk inputs before assigning the Simple tier.
- Deterministic concentration rules; model judgment is not the sole basis for security-critical classification.
- Exact trade-intent binding across classification, confirmation, revalidation, and execution.
- Mandatory final pre-execution state refresh for every tier.
- No blind retry after ambiguous execution results.
- Post-execution verification of order/position state.
- MCP tool output is treated as untrusted data, never as instructions.

## Thresholds

See [`references/thresholds.md`](references/thresholds.md) for the current numeric policy.

See [`references/execution-safety.md`](references/execution-safety.md) for freshness, revalidation, and ambiguous-execution handling.

## Important scope note

This Skill is a policy/workflow layer. It cannot guarantee the security of the connected MCP or Binance account. In particular, API authentication/authorization, exchange-side permissions, transport security, and the implementation of the connected Agent OS remain outside this repository.
