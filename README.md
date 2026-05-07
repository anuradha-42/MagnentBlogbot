# MagnentBlogBot

Automated overnight blog writing pipeline for Magnent (AEO/SEO agency).

## What it does

1. **6pm IST daily** — Slack DM to Anuradha asking for tonight's blog topic
2. **11:30pm IST daily** — Full pipeline runs autonomously:
   - Reads topic from Notion Blog Queue
   - Loads client context (Layer 2 + Layer 3A) from Notion
   - Researches via Tavily API (3 searches × 3 results)
   - Writes a 1400–1800 word AEO/SEO-optimised article
   - Posts draft to Notion Blog Posts database
   - Sends Slack DM with review link

## Infrastructure

| Component | Detail |
|---|---|
| Runtime | Claude Code Remote (CCR) — Anthropic cloud, no local machine needed |
| Model | claude-sonnet-4-6 |
| MCP connectors | Notion, Slack |
| Web research | Tavily API (WebSearch fallback) |

## Notion databases

| Database | URL |
|---|---|
| Blog Queue | https://www.notion.so/d6ca4cb0d13740aea712c5cacd3b0c00 |
| Blog Posts | https://www.notion.so/2fbf3d223ec0467e8fe0af5905293159 |
| Magnent Clients | https://app.notion.com/p/8cfbdd7fbba94adcb73c36306fcd6666 |

### Blog Queue schema

Add a row here before 11:30pm each night:

| Field | Required | Notes |
|---|---|---|
| Topic | Yes | The blog subject |
| Client | Yes | e.g. Precisa, Finezza, Recepto |
| Angle / Notes | Optional | Specific angle or keyword to target |
| Status | Yes | Set to **Pending** |
| Layer 2 URL | Optional | Direct link speeds up the run |
| Layer 3A URL | Optional | Direct link speeds up the run |

## Routines

Managed via [claude.ai/code/routines](https://claude.ai/code/routines).

| Routine | ID | Cron (UTC) | IST |
|---|---|---|---|
| Evening Prompt | trig_015sPswrDPMUR4qjJkp3nSeX | `30 12 * * *` | 6:00pm daily |
| Overnight Writer | trig_01QCZGXmC7odr6eyLhSZPvoz | `0 18 * * *` | 11:30pm daily |

## Client context structure (Notion)

Each client has layered context pages:

- **Layer 1** — Brand Foundation (voice, audience, differentiators)
- **Layer 2** — AEO Strategy (content audit, confirmed pillars, keywords)
- **Layer 3A** — Content Calendar (planned topics, target keywords, Step 3 gaps)
- **Layer 3B** — Execution notes (optional)

The bot reads Layer 2 + Layer 3A by default. Layer 1 is only loaded if Persona is blank.

## Prompts

- [`prompts/overnight-writer.md`](prompts/overnight-writer.md) — Full 8-step pipeline prompt
- [`prompts/evening-prompt.md`](prompts/evening-prompt.md) — Slack nudge prompt

## Writing rules (baked into overnight writer)

- AEO structure: answer-first paragraphs, H1/H2/H3, FAQ at end
- Brand name in paragraph 1 (non-negotiable)
- Min 2 internal links + min 4–5 external authoritative links, embedded in body
- WordPress-compatible: no HTML, no emoji, minimal bold, simple tables
- Never criticise competitors by name
- No unproven superlatives

## Updating a prompt

1. Edit the relevant file in `prompts/`
2. Push to this repo
3. Update the live routine via `claude.ai/code/routines` or ask Claude Code to update it
