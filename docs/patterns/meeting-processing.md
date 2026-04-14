# Meeting Processing Pattern

Combine calendar-aware AI summaries with verbatim transcription for comprehensive meeting notes.

## Why You'd Want This

Meetings generate valuable information that's hard to capture manually. AI summary tools give you structure (action items, decisions). Local transcription gives you verbatim accuracy (who said what). Combining both gives you the best of both worlds.

## Architecture

```
Calendar Event
      │
      ├──→ AI Summary Tool ──→ Summary (action items, decisions, attendees)
      │    (Granola, Otter,
      │     Fireflies, etc.)
      │
      └──→ Local Transcription ──→ Transcript (verbatim, speaker labels)
           (MacWhisper, Whisper,
            Buzz, etc.)
                  │
                  ▼
         ┌──────────────┐
         │   Combiner    │
         │               │
         │  Summary      │
         │  + Transcript │
         │  = Full Notes │
         └──────────────┘
                  │
                  ▼
         meetings/YYYY-MM-DD-title.md
```

### Source 1: AI Summary Tool

Calendar-aware meeting assistants that join calls and produce structured output:
- **What they capture**: Attendees, agenda, key topics, decisions, action items
- **Strengths**: Structure, actionability, quick to scan
- **Weaknesses**: May miss nuance, paraphrase inaccurately, miss offline discussions
- **Examples**: Granola, Otter.ai, Fireflies, Fathom

### Source 2: Local Transcription

Audio transcription with speaker diarization (who said what):
- **What they capture**: Every word spoken, attributed to speakers
- **Strengths**: Verbatim accuracy, speaker identification, works offline
- **Weaknesses**: No structure, no prioritization, large output
- **Examples**: MacWhisper, OpenAI Whisper (local), Buzz

### Combining Both

The combination is more valuable than either source alone:

| From Summary | From Transcript |
|-------------|----------------|
| Action items | Exact quotes supporting decisions |
| Decisions made | Context around why decisions were made |
| Attendee list | Who said what |
| Topic structure | Details that summaries miss |

## Processing Workflow

### Step 1: Detect Meeting
Monitor calendar for completed meetings. When one ends:
- Check if AI summary is available
- Check if local transcription is available

### Step 2: Gather Sources
```bash
# Example: sync AI summaries
./marvin.sh granola sync  # or your summary tool's CLI/API

# Example: find local transcript
./marvin.sh whisper find "meeting title"
```

### Step 3: Map Speakers
Local transcription labels speakers generically ("Speaker 1", "Speaker 2"). Map them:
- "Microphone" or "Speaker 0" = you (the meeting host)
- Cross-reference attendee list from calendar/summary
- Map by content context if needed

### Step 4: Combine
Create a processed note that merges both sources:

```markdown
# Meeting: {Title}
Date: {YYYY-MM-DD}
Attendees: {from summary}

## Summary
{AI-generated summary}

## Decisions
{from summary, enriched with transcript quotes}

## Action Items
{from summary}
- [ ] {item} -- {owner}

## Key Discussion
{selected transcript excerpts organized by topic}

## Full Transcript
{link to raw transcript file}
```

### Step 5: Save
```
meetings/
├── YYYY-MM-DD-meeting-title.md    # Processed notes
└── transcripts/
    ├── granola/                    # Raw AI summaries
    └── whisper/                    # Raw transcriptions
```

## Speaker Mapping

| Transcript Label | Mapping Strategy |
|-----------------|-----------------|
| "Microphone" / "Speaker 0" | Always the host (you) |
| "Speaker 1", "Speaker 2" | Match from attendee list + context |
| Duplicate speakers | Same person detected as multiple; merge based on content |

## Key Decisions

| Decision | What we learned |
|----------|----------------|
| Two sources > one | Summaries miss detail; transcripts miss structure |
| Local transcription | Privacy: audio stays on your machine |
| Raw files preserved | Always keep originals; processed notes can be regenerated |
| Speaker mapping is imperfect | Best effort is fine; don't block on perfect attribution |
| Calendar-aware detection | Meetings are calendar events; use that for automatic processing |

## Building a /meeting Command

Create a `/meeting` command that automates the workflow:
1. List recent meetings (from calendar or summary tool)
2. User picks which one to process
3. Pull summary and transcript
4. Map speakers
5. Combine into processed notes
6. Save to `meetings/`

## Getting Started

1. Choose your tools (one AI summarizer + one transcriber)
2. Test both on a real meeting
3. Manually combine the outputs once to understand the format
4. Create the `meetings/` directory structure
5. Build a `/meeting` command to automate the combination
6. Add meeting detection to your `/start` briefing (any unprocessed meetings?)
