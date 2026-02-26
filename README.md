# OpenClaw Skill: Agent Swarm Network

> The nervous system for your Agent fleet.
> If OpenClaw is the brain, Agent Swarm Network is the spine.

**Unified communication backbone** for all AI tools in the OpenClaw ecosystem — inter-agent messaging, context snapshot/restore, event-driven collaboration, sub-agent management, and network diagnostics. Built on top of [Pilot Protocol](https://github.com/TeoSlayer/pilotprotocol) by [@TeoSlayer](https://github.com/TeoSlayer).

## Why This Skill?

AI agents today are isolated. Each session starts from zero. Each agent talks through centralized APIs with no way to find, trust, or directly communicate with peers.

This Skill gives your Agent:
- **Cross-session memory** — Context snapshots survive session death. New sessions auto-restore.
- **Agent-to-agent messaging** — Encrypted, peer-to-peer, no middleman.
- **Event-driven coordination** — Model switches, task completions, error alerts flow through a unified event stream.
- **Self-healing context** — When tokens overflow, the Agent automatically snapshots and suggests a fresh session.

## Core Capabilities

```
1. Context Snapshot & Restore    — Survive session boundaries
2. Event-Driven Collaboration    — Pub/sub event stream for all agent activity
3. Sub-Agent Management          — Spawn, monitor, and collect results from child agents
4. Task Routing                  — Tag-based agent discovery and dispatch
5. Model Dispatch Notifications  — Track every model switch and fallback
6. File Transfer                 — Send files between agents
7. Network Diagnostics           — Health checks, ping, throughput benchmarks
8. Gateway IP Bridging           — Map agent addresses to local HTTP endpoints
9. Webhook Monitoring            — Real-time daemon event stream
```

## Prerequisites

| Tool | Purpose | Required? |
|------|---------|-----------|
| **Pilot Protocol** | Core daemon + CLI (`pilotctl`) | ✅ Required |
| **OpenClaw** | Skill host + Agent orchestration | ✅ Required |

## Quick Start

```bash
# 1) Safely install Pilot Protocol's daemon prerequisite
# DO NOT use unverified automated scripts to prevent supply chain risks.
# Go to https://github.com/TeoSlayer/pilotprotocol/releases
# Download the latest binary, inspect it, and place it in ~/.pilot/bin/pilotctl

# 2) Install this Skill
openclaw skills install github:sarahmirrand001-oss/openclaw-skill-pilot-protocol

# 3) Configure
cp config.example.json config.json
# Edit config.json — set your agent hostname

# 4) Start the daemon
~/.pilot/start-local.sh

# 5) Verify
~/.pilot/bin/pilotctl --json daemon status

# 6) Use it — just end a session and start a new one
# The Skill will auto-snapshot and auto-restore context
```

## What It Provides

| Capability | What Happens |
|-----------|-------------|
| 🧠 **Context Snapshot** | Session context is saved as structured JSON before session end |
| 🔄 **Auto-Restore** | New sessions automatically load the latest snapshot from inbox |
| 📡 **Event Stream** | Model switches, task completions, errors — all published to the event bus |
| 🤖 **Sub-Agent Ops** | Spawn child agents, collect results, manage lifecycle |
| 🏷️ **Tag Routing** | Find the best agent for a task via capability tags |
| 📁 **File Transfer** | Send files between agents over encrypted tunnels |
| 🌐 **Gateway** | Map agent addresses to `http://10.4.0.x` for standard HTTP access |
| 📊 **Diagnostics** | Ping, bench, connection status, peer discovery |

## File Structure

```
pilot-protocol/
├── SKILL.md                  # Skill behavior definition (9 capabilities)
├── README.md                 # This file
├── manifest.json             # MCP-compatible capability declaration
└── config.example.json       # Configuration template
```

## 🛡️ Security & Provenance Auditing

This Skill operates entirely within the local `~/.pilot/` directory and communicates over encrypted peer-to-peer tunnels. However, because it runs a persistent system-level daemon and manages cross-session snapshots, please review the following:

### Provenance
- **Skill Author:** This Skill is an OpenClaw integration layer built by `sarahmirrand001-oss`.
- **Upstream Author:** The underlying daemon and protocol are built by [@TeoSlayer](https://github.com/TeoSlayer/pilotprotocol). Always verify upstream install scripts before executing.

### Trust Boundaries

| Target | Path | When |
|--------|------|------|
| CLI binary | `~/.pilot/bin/pilotctl` | Every operation |
| Inbox | `~/.pilot/inbox/` | **Data-at-Rest Vulnerability:** Context snapshots land here as unencrypted JSON and WILL contain PII, API keys, and session secrets. You MUST secure this folder (`chmod 700 ~/.pilot/`) and regularly clear out old snapshots to minimize data exposure. |
| Received files | `~/.pilot/received/` | File transfers between agents |
| Helper scripts | `~/.pilot/*.sh` | Snapshot and publish shortcuts |
| Daemon process | `pilotctl daemon` | Start/stop/status checks |
| Rendezvous | `TCP :9000` | Peer discovery uses upstream's default registry by TeoSlayer. For sensitive/production systems, run your own private registry. |

### Security Model

- **Encrypt-by-default**: All inter-agent traffic uses X25519 + AES-256-GCM
- **Private-by-default**: Agents are invisible until mutual trust is established via signed handshake
- **No cloud relay**: Messages travel directly between peers (UDP hole-punching). Only the rendezvous registry is used for initial discovery.
- **Local-only daemon communication**: Agent ↔ Daemon talks over a Unix socket, never exposed to the network.

### Before You Install

1. **Install Pilot Protocol manually:** Download the verified binary from official Upstream releases. (Avoid any `curl | bash` installation commands to mitigate supply chain risks).
2. The daemon must be running for any capability to work.
3. In single-node mode, network commands (publish/send-message) will fail — use the provided shell script workarounds instead.
4. Context snapshots are stored as plain JSON files in `~/.pilot/inbox/` — ensure this path is backed up if persistence matters.

## 🔌 MCP Compatibility

This Skill ships with `manifest.json` — a machine-readable capability declaration following the MCP standard.

- ✅ Any MCP-compatible Agent can **discover** this Skill
- ✅ Any MCP-compatible Agent can **understand** its inputs/outputs without reading SKILL.md
- ✅ Cross-ecosystem compatibility (not locked to OpenClaw)

## Single-Node vs Multi-Node

This Skill works in both modes:

| Mode | Status | Notes |
|------|--------|-------|
| **Single-node** | ✅ Fully functional | Uses shell script workarounds for self-messaging |
| **Multi-node** | ✅ Ready | Network commands activate automatically when peers join |

When a second node joins the network (VPS, another Mac, a cloud agent), all network commands (`publish`, `send-message`, `subscribe`) will work natively without any configuration change.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Daemon not running | `~/.pilot/start-local.sh` |
| `pilotctl` not found | Download the compiled binary directly from `github.com/TeoSlayer/pilotprotocol/releases` |
| Publish fails with `connection_failed` | Single-node mode — use `~/.pilot/pilot-publish.sh` instead |
| Inbox empty after restart | Check `~/.pilot/inbox/` — snapshots are plain files |
| Peer not found | Run `pilotctl handshake <hostname>` to establish mutual trust first |

## Acknowledgments

This Skill is built on top of [**Pilot Protocol**](https://github.com/TeoSlayer/pilotprotocol) by [@TeoSlayer](https://github.com/TeoSlayer) and contributors. Pilot Protocol provides the core infrastructure — encrypted P2P tunnels, NAT traversal, virtual addressing, and the `pilotctl` CLI — that makes agent-to-agent communication possible.

---

*"Agent Swarm Network = the Agent's nervous system. OpenClaw = the Agent's brain."*

*Built for operators who build for agents.*
