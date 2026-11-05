# Shopping Agent Starter — OpenAI Agents SDK with a pre-payment conformity check

A ready-to-copy **shopping agent template** for the OpenAI Agents
SDK: an agent that buys on a customer's behalf and checks its
built cart against the customer's instructions **before** payment
— the step most agent payment examples skip.

**Use this template** (button above) or clone it, add your keys,
and you have a working checkout agent whose payment tool refuses
to fire until the check says MATCH.

## What is wired in, from the first commit

| Piece | Where | What it does |
|---|---|---|
| Conformity check tool | `shopping_agent.py` → `check_cart_before_payment` | Compares the built cart with the customer's signed instructions via [ProveCart](https://provecart.com); returns MATCH / REVIEW / BLOCK with a machine-readable correction payload (Agent Correction Format) |
| Payment gate | `capture_payment` | Refuses unless the latest check in the run returned MATCH; keeps the check's `record_id` with the order as evidence |
| Correction loop | agent instructions + tool contract | On BLOCK: fix only what stays inside the customer's authorization, re-check with `previous_record_id`. On REVIEW: stop and ask the customer |
| Certification CI | `.github/workflows/certify.yml` | Runs the free [ProveCart Verified Agent exam](https://provecart.com/agents) for this agent on every release, so a prompt or model change that makes it buy carelessly fails CI |
| Coding-agent rules | `AGENTS.md` / `CLAUDE.md` | Tell Codex, Claude Code, Cursor and friends to keep the check step when they edit this repo |

## Run it

```bash
pip install -r requirements.txt
python shopping_agent.py            # offline demo loop, no keys
python shopping_agent.py --sdk      # full Agents SDK loop
```

Offline by default: the check routes to a local mock with the
real response shape. For the live check, set:

```bash
export LIVE_MODE=true
export PROVECART_API_KEY=...        # integrator key (merchant side)
export OPENAI_API_KEY=...
```

## Why the check exists

Agents build orders from instructions and get details wrong —
wrong variant, wrong date, a total over the customer's ceiling —
and the mistake is discovered at the return desk. A 200 ms
conformity check before capture turns that into a correction the
agent makes itself, or a question to the customer, while there is
still time. ProveCart never holds or moves money; it only
verdicts. The check is free to practice and to certify against:
https://provecart.com/agents.md

## Make it yours

- Replace the demo merchant/catalog with your store or your
  customer's.
- Replace `capture_payment`'s body with your payment provider —
  keep the MATCH gate in front of it.
- Carry `previous_record_id` and the final `record_id` in your
  run state (the starter uses a module-level dict so the seam is
  easy to see).

MIT licensed sample code. The correction format the check speaks
is published and royalty-free to implement:
https://provecart.com/standards/correction-format
