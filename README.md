# Pump.fun Replay Sprint — Paper Bot

[![make.fun · u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122](https://make.fun/badge/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122.svg)](https://make.fun/bounty/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122?utm_source=github&utm_medium=readme&utm_campaign=u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122)
[![rank](https://make.fun/badge/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122/MonyetAku/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122/rank.svg)](https://make.fun/bounty/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122)
[![reward](https://make.fun/badge/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122/MonyetAku/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122/reward.svg)](https://make.fun/bounty/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122?utm_source=github&utm_medium=readme&utm_campaign=u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122)

A paper-trading bot for the [Pump.fun Replay Sprint](https://make.fun/bounty/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122) on [make.fun](https://make.fun/?ref=vqmdgqxf) (referral: `vqmdgqxf`). It replays a 24h window of pump.fun launches and trades them with a single-file TypeScript bot (`main.ts`) that speaks the `agent-fun/replay/v2` protocol over stdin/stdout — no dependencies, no vendored SDK, fully deterministic.

## The idea

On the dev window every house baseline loses money: momentum-chasing buys the top (-9.39 SOL), dip-buying catches falling knives (-4.02 SOL), and even the launch sniper (-0.49 SOL) pays too much for entries that already pumped. The tape shows why: a coin's price is usually at its best within a few slots of creation, then fades — the median coin is flat-to-down 10 slots after its first trade, while a few run 2–10x. So the edge is not in holding or in pyramiding; it is in **buying only the cheapest, earliest launches and getting out fast**.

This bot does exactly that:

- **Buys early and cheap.** Only coins ≤ 3 slots old, whose price has run up at most 10% from creation, with 0.3–30 SOL of real reserves, a creator buy of at most 1 SOL, at most 1 non-creator buyer so far, and no creator sell. Stake 0.05 SOL, capped at 1% of real reserves, and never below 0.01 SOL (so every round trip counts toward the 20-trip guardrail and dust trades never pay a full tx fee).
- **Exits fast.** Take profit at +15%, stop loss at −12%, hard time exit after 10 slots, and an immediate exit if the creator sells after entry. No pyramiding, no partial take-profits, no trailing stops — on this data, adding to a "rising" coin and holding for the tail both lose money.
- **Stays small and safe.** At most 5 open positions, one order per buy/sell (no splitting, so no extra tx fees), wide-open slippage so a fill is never refused for the 2-slot latency move.

## Measured on the dev window (full 24h, all 3 dataset parts)

| bot | net PnL (SOL) | trips | wins | max drawdown |
| --- | --- | --- | --- | --- |
| momentum (house) | −9.39 | 80 | 5 | 0.95 |
| reversion (house) | −4.02 | 424 | 111 | 0.49 |
| sniper (house) | −0.49 | 349 | 45 | 0.05 |
| **this bot** | **−0.020** | **30** | **1** | **0.002** |
| hold cash | 0.00 | 0 | 0 | 0.00 |

No strategy is positive on the dev window — it is a losing market for buyers after fees and the 2-slot fill latency. This bot is the closest to breakeven with real exposure: 30 qualifying round trips (≥ 0.01 SOL each), a 0.2% drawdown, and the cheapest entries of any tested configuration, so it needs the smallest improvement in market conditions to turn positive on the unseen final window. Scores are means over four failed-transaction draws (11, 22, 33, 44); the worst draw is −0.024 SOL. The numbers above are the official `grade-local --driver process` dev grade (result_sha256 `4f6fb7bcf8c8214bc61f08f9e54f952516a173761b783142e6ce74ce1efe4284`, action_log_sha256 `6c8116cb8b7aaad15697435c09c053e90f62e6200592ed2e0bd365ea747fffaa`).

## Run it

```sh
node main.ts   # reads agent-fun/replay/v2 NDJSON on stdin, writes one reply per batch on stdout
```

## Files

- `main.ts` — the whole bot: a minimal protocol client plus the strategy above.
- `.makefun/owner.json` — payout wallet.
- `LICENSE` — MIT.
