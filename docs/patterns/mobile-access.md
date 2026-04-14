# Mobile Access Pattern

Run MARVIN from your phone with full context and capabilities.

## Why You'd Want This

Your best ideas don't happen at your desk. Being able to share a link, brainstorm, or check your schedule from your phone keeps MARVIN useful throughout the day. But mobile access has unique constraints: smaller screens, intermittent connectivity, touch input, and the need for shorter responses.

## Approaches

### Approach 1: WebSocket Relay

A relay server bridges your phone to your desktop MARVIN instance.

```
Phone App ←──WebSocket──→ Relay Server ←──WebSocket──→ Desktop Daemon
```

**How it works:**
1. A relay server runs in the cloud (Railway, Fly.io, or similar)
2. Your desktop daemon maintains a persistent WebSocket connection
3. Your phone connects to the same relay
4. Messages are forwarded bidirectionally

**Pros:**
- Full MARVIN capabilities (desktop has all tools and context)
- No separate AI subscription for mobile
- Conversations persist locally

**Cons:**
- Requires always-on daemon at home
- Latency depends on relay + desktop being up
- If daemon is down, mobile doesn't work

### Approach 2: Claude Code SDK Session

Run a Claude Code SDK session that your phone connects to directly.

```
Phone App ←──API──→ Claude Code SDK Session (cloud or local)
```

**How it works:**
1. A persistent Claude Code SDK session runs on your server
2. The session has your MARVIN context (CLAUDE.md, state files)
3. Phone sends messages via API, receives responses
4. Session maintains conversation history

**Pros:**
- Works independently of your desktop
- Can run in the cloud (always available)
- Lower latency than relay

**Cons:**
- Needs its own compute (cloud instance or home server)
- Context sync between mobile and desktop sessions
- API costs for the SDK session

### Approach 3: Native Mobile AI App

Use a mobile AI app (Claude mobile, ChatGPT, etc.) with exported context.

**How it works:**
1. Export your MARVIN context as a system prompt
2. Load it into a mobile AI app
3. Use the app directly

**Pros:**
- Simplest setup
- Works anywhere with internet
- No infrastructure to maintain

**Cons:**
- Limited capabilities (no file access, no tools)
- Context gets stale (manual re-export needed)
- No conversation persistence back to desktop

## Mobile UX Considerations

Mobile is different from desktop. Optimize for:

| Desktop | Mobile |
|---------|--------|
| Long, detailed responses | Short, scannable responses |
| Multi-step workflows | Quick actions |
| File editing | Link sharing and note capture |
| Deep work sessions | Quick check-ins |

### Common Mobile Use Cases

1. **Link sharing** (most common): Share a URL, MARVIN processes it (extract transcript, summarize article, save for research)
2. **Quick lookups**: "What's on my calendar today?" "Any P1 emails?"
3. **Brainstorming**: Capture ideas on the go, MARVIN saves to research/inbox/
4. **Social posts**: Draft a quick LinkedIn post from your phone
5. **Talk prep**: Review speaker notes before going on stage

### Conversation Persistence

Mobile conversations should be saved for desktop continuity:
```
sessions/mobile/
├── YYYY-MM-DD.md    # Daily mobile conversation log
```

During desktop `/start`, scan recent mobile logs for:
- Shared links that need processing
- Unresolved threads
- Ideas captured on the go

Retention: keep 2 weeks of mobile logs, archive older ones.

## Key Decisions

| Decision | What we learned |
|----------|----------------|
| Relay approach for full power | Desktop has all tools; mobile gets full capability via relay |
| Short responses by default | Mobile users want answers, not essays |
| Link sharing is the killer feature | 90% of mobile use is "process this link" |
| Persist conversations | Desktop needs to see what happened on mobile |
| 2-week rolling retention | Mobile logs are ephemeral context, not permanent records |

## Getting Started

1. **Start with Approach 3** (native app with exported context) to validate you'll use mobile access
2. If you find yourself wanting tool access, move to **Approach 2** (SDK session)
3. If you need full desktop parity, build **Approach 1** (relay)
4. Don't over-engineer mobile. Most people just need link sharing and quick lookups.
