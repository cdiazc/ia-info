---
name: ai-weekly-digest
description: Scrape AI news sources and generate a weekly digest organized into Claude Code, dev AI news, general AI news, and must-read/must-watch picks. Sources come from https://github.com/cdiazc/ia-info.
---

# /ai-weekly-digest - Weekly AI News Digest

Scrapes AI news sources from the curated list at https://github.com/cdiazc/ia-info and produces a concise weekly digest.

## Arguments

- **week** (optional): `last week`, `this week` (default), or a date like `2026-03-20`

## Instructions

### 1. Determine the Date Range

- Default: current week (Monday to today)
- `last week`: previous Monday to Sunday
- Specific date: Monday of that week to Sunday

### 2. Fetch the Source Index

First, fetch the curated source list to check for any new sources added since the skill was written:

```
WebFetch https://github.com/cdiazc/ia-info/blob/main/README.md
```

Extract all blog/newsletter/YouTube URLs. Compare against the hardcoded lists below and include any new sources found.

### 3. Scrape Sources in Parallel (5 subagents)

Launch **5 subagents in parallel** using the Agent tool. Each subagent uses `WebFetch` to scrape its assigned sources and returns a structured summary.

**IMPORTANT for all subagents:**
- Only include content published within the target week's date range
- For each item found, extract: **title**, **source name**, **date** (if available), **1-2 sentence summary**, and **URL**
- If a source fails to load or has no new content, skip it silently
- Keep output concise — max 5 items per source, prioritize by relevance/impact
- Return results as a structured list, not prose

---

#### Subagent 1: Claude & Anthropic

Scrape these sources for Claude Code updates, Anthropic announcements, and Claude-related content:

| Source | URL | What to look for |
|--------|-----|------------------|
| Claude Blog | https://claude.com/blog | New posts about Claude Code, features, updates |
| Anthropic News | https://www.anthropic.com/news | Company announcements, model releases, research |
| Simon Willison | https://simonwillison.net/ | Posts mentioning Claude, Claude Code, Anthropic |

**Prompt template for each WebFetch:**
> "List all blog posts or articles published in the last 7 days. For each, give me: title, date, URL, and a 1-sentence summary. If the page shows an archive/list, extract from that. If it shows a single post, summarize it. Only include items from {date_range}."

**Return format:**
```
## Claude & Anthropic
- [Title](url) — Source — Date — 1-sentence summary
- ...
```

---

#### Subagent 2: Other AI Providers (Dev-focused)

Scrape for developer-relevant news from other major AI providers:

| Source | URL | What to look for |
|--------|-----|------------------|
| OpenAI News | https://openai.com/news/ | API updates, model releases, developer tools |
| DeepMind Blog | https://deepmind.google/blog/ | Research breakthroughs, new models |
| Hugging Face Blog | https://huggingface.co/blog | Open-source models, libraries, tools |
| Google Developers Blog | https://developers.googleblog.com/ | AI/ML developer tools, Gemini updates |

**Prompt template for each WebFetch:**
> "List all blog posts or articles published in the last 7 days. For each: title, date, URL, 1-sentence summary. Only include AI/ML related content from {date_range}."

**Return format:**
```
## Dev AI News — Other Providers
- [Title](url) — Source — Date — 1-sentence summary
- ...
```

---

#### Subagent 3: AI Newsletters & Independent Blogs

Scrape curated newsletters and independent voices:

| Source | URL | What to look for |
|--------|-----|------------------|
| TLDR AI | https://tldr.tech/ai | Daily AI news roundup — top stories |
| Ben's Bites | https://www.bensbites.com/ | AI industry news, product launches |
| Sebastian Raschka | https://magazine.sebastianraschka.com/ | ML research, practical AI insights |
| Andrej Karpathy Blog | https://karpathy.bearblog.dev/ | Deep learning insights, tutorials |

**Prompt template for each WebFetch:**
> "List the most recent posts or newsletter editions from the last 7 days. For each: title, date, URL, 1-sentence summary. Only include items from {date_range}."

**Return format:**
```
## Newsletters & Independent Blogs
- [Title](url) — Source — Date — 1-sentence summary
- ...
```

---

#### Subagent 4: YouTube Channels (Video Content)

Check recent uploads from key AI YouTube channels:

| Channel | URL |
|---------|-----|
| Fireship | https://www.youtube.com/@Fireship/videos |
| AI Explained | https://www.youtube.com/@aiexplained-official/videos |
| Matthew Berman | https://www.youtube.com/@matthew_berman/videos |
| The AI Breakdown | https://www.youtube.com/@TheAIBreakdown-com/videos |
| Andrej Karpathy | https://www.youtube.com/@AndrejKarpathy/videos |
| Yannic Kilcher | https://www.youtube.com/@YannicKilcher/videos |
| Data Independent | https://www.youtube.com/@DataIndependent/videos |

**Prompt template for each WebFetch:**
> "List all videos uploaded in the last 7 days. For each: title, upload date, video URL, and 1-sentence description of the topic. Only include videos from {date_range}."

**Return format:**
```
## YouTube — Recent Videos
- [Title](url) — Channel — Date — 1-sentence description
- ...
```

---

#### Subagent 5: Additional Sources & Wildcard

Scrape remaining sources for anything notable:

| Source | URL | What to look for |
|--------|-----|------------------|
| DigitalOcean AI Articles | https://www.digitalocean.com/resources/articles/ai-blogs | Curated AI blog lists, new resources |
| Trelis Research (YouTube) | https://www.youtube.com/@TrelisResearch/videos | ML research explainers |
| Sentdex (YouTube) | https://www.youtube.com/@sentdex/videos | Python ML tutorials, news |
| Computerphile (YouTube) | https://www.youtube.com/@Computerphile/videos | CS/AI explainers |
| IBM Technology (YouTube) | https://www.youtube.com/@IBMTechnology/videos | Enterprise AI |
| Engineer Prompt (YouTube) | https://www.youtube.com/@engineerprompt/videos | AI tools & prompting |

**Also** do a broad WebSearch for: `"AI news this week" OR "LLM news" site:reddit.com OR site:news.ycombinator.com` to catch anything the dedicated sources missed.

**Return format:**
```
## Additional Sources
- [Title](url) — Source — Date — 1-sentence summary
- ...
```

---

### 4. Compile the Digest

Once all 5 subagents return, compile their results into the final digest. Apply these rules:

#### Classification Rules

**1. Claude Code** — Anything about Claude Code (the CLI/IDE tool), Claude models, Anthropic announcements, MCP servers/protocol, Claude API changes. Also include community blog posts that are specifically about using Claude Code or building with Claude.

**2. Dev AI News (Other Providers)** — Developer-relevant announcements from OpenAI, Google/DeepMind, Meta AI, Hugging Face, Mistral, Cohere, etc. Focus on: new models, API changes, developer tools, SDK updates, pricing changes. Skip pure research papers unless they have immediate practical impact.

**3. Other LLM & AI World News** — Broader industry news: regulation, funding rounds, acquisitions, AI ethics debates, product launches (non-dev), general trends, opinion pieces, research breakthroughs. Anything that doesn't fit cleanly into sections 1 or 2.

**4. Emerging Standards Across AI Agents (OPTIONAL)** — Only include this section when the week's news reveals tools, methodologies, protocols, or patterns that are converging as de facto standards across multiple AI coding agents (Claude Code, Cursor, Copilot, Windsurf, Kilo, Kiro, etc.). Examples: MCP adoption, spec-driven development, memory systems, context protocols, tool-use patterns, agentic workflows. Each item must cite at least 2 agents/tools adopting it. Skip this section entirely if nothing qualifies — do NOT force it.

**5. Must Read & Must Watch (5 total)** — Pick the 5 most impactful/interesting items across ALL sections. Mix of articles and videos. Prioritize:
   - Breaking news or major announcements
   - Hands-on tutorials or deep dives with lasting value
   - Contrarian or surprising takes
   - Items that a busy developer would regret missing

#### Formatting Rules

- Each section: **max 7 items**, each item is **1-2 lines max**
- Item format: `**[Title](url)** — Source — 1-sentence summary`
- If a section has no items for the week, write "Quiet week — nothing notable."
- The Must Read/Must Watch section should have a brief (1 line) note on *why* each pick made the list
- Total digest should be readable in **under 3 minutes**

### 5. Output — Markdown (in conversation)

Display the digest directly in the conversation:

```markdown
# AI Weekly Digest — {Mon DD} to {Mon DD, YYYY}

## 1. Claude Code & Anthropic

- **[Title](url)** — Source — Summary
- ...

## 2. Dev AI News (Other Providers)

- **[Title](url)** — Source — Summary
- ...

## 3. LLM & AI World News

- **[Title](url)** — Source — Summary
- ...

## 4. Emerging Standards Across AI Agents

- **[Standard/Pattern]** — Adopted by: Agent1, Agent2, Agent3 — What it is and why it's converging
- ...

_This section is optional. Omit entirely if no cross-agent convergence was spotted this week._

## 5. Must Read & Must Watch (Top 5)

1. **[Title](url)** — Source — Why this matters: ...
2. **[Title](url)** — Source — Why this matters: ...
3. **[Title](url)** — Source — Why this matters: ...
4. **[Title](url)** — Source — Why this matters: ...
5. **[Title](url)** — Source — Why this matters: ...

---
*Sources: [ia-info](https://github.com/cdiazc/ia-info) | Generated {today's date}*
```

### 6. Output — HTML File

After displaying the markdown, also generate a styled HTML file and save it to `/tmp/ai-weekly-digest-{YYYY-MM-DD}.html`.

Use the Write tool to create the file with this template — replace the content placeholders with the actual digest items compiled in step 4:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AI Weekly Digest — {date_range}</title>
  <style>
    :root {
      --bg: #0f0f0f;
      --surface: #1a1a1a;
      --surface2: #242424;
      --border: #333;
      --text: #e0e0e0;
      --text-muted: #888;
      --accent-claude: #d4a574;
      --accent-dev: #7aa2f7;
      --accent-world: #9ece6a;
      --accent-standards: #bb9af7;
      --accent-picks: #ff9e64;
      --link: #7dcfff;
    }
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Text', 'Segoe UI', system-ui, sans-serif;
      background: var(--bg);
      color: var(--text);
      line-height: 1.6;
      max-width: 720px;
      margin: 0 auto;
      padding: 2rem 1.5rem;
    }
    header {
      border-bottom: 1px solid var(--border);
      padding-bottom: 1.5rem;
      margin-bottom: 2rem;
    }
    header h1 {
      font-size: 1.5rem;
      font-weight: 600;
      letter-spacing: -0.02em;
    }
    header .date-range {
      color: var(--text-muted);
      font-size: 0.9rem;
      margin-top: 0.3rem;
    }
    section {
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: 12px;
      padding: 1.5rem;
      margin-bottom: 1.25rem;
    }
    section h2 {
      font-size: 1.1rem;
      font-weight: 600;
      margin-bottom: 1rem;
      display: flex;
      align-items: center;
      gap: 0.5rem;
    }
    .badge {
      display: inline-block;
      width: 8px;
      height: 8px;
      border-radius: 50%;
    }
    .badge-claude { background: var(--accent-claude); }
    .badge-dev { background: var(--accent-dev); }
    .badge-world { background: var(--accent-world); }
    .badge-standards { background: var(--accent-standards); }
    .badge-picks { background: var(--accent-picks); }
    ul { list-style: none; }
    ul li {
      padding: 0.6rem 0;
      border-bottom: 1px solid var(--surface2);
      font-size: 0.92rem;
    }
    ul li:last-child { border-bottom: none; }
    a { color: var(--link); text-decoration: none; font-weight: 500; }
    a:hover { text-decoration: underline; }
    .source { color: var(--text-muted); font-size: 0.82rem; }
    .why { color: var(--text-muted); font-size: 0.85rem; font-style: italic; display: block; margin-top: 0.2rem; }
    .adopted { color: var(--accent-standards); font-size: 0.82rem; }
    ol { list-style: none; counter-reset: picks; }
    ol li { counter-increment: picks; position: relative; padding-left: 1.8rem; }
    ol li::before {
      content: counter(picks);
      position: absolute;
      left: 0;
      top: 0.6rem;
      background: var(--accent-picks);
      color: var(--bg);
      width: 1.2rem;
      height: 1.2rem;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 0.75rem;
      font-weight: 700;
    }
    footer {
      text-align: center;
      color: var(--text-muted);
      font-size: 0.8rem;
      margin-top: 2rem;
      padding-top: 1rem;
      border-top: 1px solid var(--border);
    }
    footer a { color: var(--text-muted); }
    .quiet { color: var(--text-muted); font-style: italic; padding: 0.5rem 0; }
  </style>
</head>
<body>
  <header>
    <h1>AI Weekly Digest</h1>
    <div class="date-range">{Mon DD} — {Mon DD, YYYY}</div>
  </header>

  <section>
    <h2><span class="badge badge-claude"></span> Claude Code & Anthropic</h2>
    <ul>
      <!-- For each item: -->
      <li><a href="{url}">{Title}</a> <span class="source">— {Source}</span><br>{Summary}</li>
      <!-- If empty: <li class="quiet">Quiet week — nothing notable.</li> -->
    </ul>
  </section>

  <section>
    <h2><span class="badge badge-dev"></span> Dev AI News (Other Providers)</h2>
    <ul>
      <li><a href="{url}">{Title}</a> <span class="source">— {Source}</span><br>{Summary}</li>
    </ul>
  </section>

  <section>
    <h2><span class="badge badge-world"></span> LLM & AI World News</h2>
    <ul>
      <li><a href="{url}">{Title}</a> <span class="source">— {Source}</span><br>{Summary}</li>
    </ul>
  </section>

  <!-- Section 4: Emerging Standards — ONLY include this <section> block if there are items. Omit entirely if none. -->
  <section>
    <h2><span class="badge badge-standards"></span> Emerging Standards Across AI Agents</h2>
    <ul>
      <li><strong>{Standard/Pattern}</strong> <span class="adopted">— Adopted by: {Agent1, Agent2, Agent3}</span><br>{What it is and why it's converging}</li>
    </ul>
  </section>

  <section>
    <h2><span class="badge badge-picks"></span> Must Read & Must Watch</h2>
    <ol>
      <li><a href="{url}">{Title}</a> <span class="source">— {Source}</span><span class="why">Why: {reason}</span></li>
    </ol>
  </section>

  <footer>
    Sources: <a href="https://github.com/cdiazc/ia-info">ia-info</a> | Generated {today's date}
  </footer>
</body>
</html>
```

After writing the file, tell the user: **"HTML version saved to `/tmp/ai-weekly-digest-{YYYY-MM-DD}.html`"** and suggest they open it with `open /tmp/ai-weekly-digest-{YYYY-MM-DD}.html`.

### 7. Quality Checks (MANDATORY)

Before presenting the digest:

1. **Deduplication**: If the same news appears from multiple sources, keep only the best/most detailed version
2. **Date accuracy**: Drop any item you cannot confirm was published within the target week
3. **Link validity**: Only include URLs that were successfully fetched — no guessed or constructed URLs
4. **Balance**: Ensure sections 1-3 each have at least 2 items (if available). If a section is empty, note it explicitly
5. **No hallucinated content**: If a WebFetch returned unclear results, mark with [unverified] or skip entirely
6. **Section 4 rigor**: Only include an "Emerging Standards" item if you can name at least 2 specific agents/tools adopting the same pattern. Do not speculate — if evidence is thin, omit the section entirely

## Notes

- X/Twitter accounts from the source list are not scrapable — skip them
- Twitch streams (CodelyTV, MiduDev) are not scrapable — skip them
- YouTube scraping may be unreliable — if a channel page doesn't load, skip it
- The source list at https://github.com/cdiazc/ia-info may be updated over time. Always re-fetch it in step 2 to pick up new sources
- WebFetch may fail on some sites due to anti-scraping measures. This is expected — work with what you get
- Language: **English** — even if source content is in Spanish
