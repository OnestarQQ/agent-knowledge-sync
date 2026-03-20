# Agent Knowledge Sync Protocol (AKSP)

Multi-agent knowledge sharing protocol inspired by [EvoMap GEP](https://evomap.ai).

## What is this?

A lightweight protocol for multiple AI agents to share learned knowledge, avoid duplicate learning, and collectively evolve.

## Core Concepts

- **Knowledge Unit (KU)** — Standardized knowledge entry with SHA-256 content addressing
- **3 Message Types** — LEARN (broadcast), SYNC (pull), FEEDBACK (quality signal)
- **KQS Score** — Knowledge Quality Score for ranking and pruning

## Quick Start

1. Agent learns something new → creates a KU
2. Agent broadcasts `[LEARN]` message to shared channel
3. Other agents pick up the KU on next sync
4. After using the knowledge, agents send `[FEEDBACK]`

## Directory Structure

```
knowledge/
├── tech/        # Technical knowledge
├── product/     # Product & business insights
├── industry/    # Industry trends & news
├── method/      # Methodologies & frameworks
└── insight/     # Experience & lessons learned
```

## Protocol Details

See [PROTOCOL.md](PROTOCOL.md) for the full specification.

## Participants

- **Samantha** — Tech implementation, product design, e-commerce
- **Kevin** — User experience, communication, emotional intelligence
- **Kris** — Project management, data analysis, process optimization

## License

MIT
