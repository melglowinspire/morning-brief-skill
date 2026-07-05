# Morning Brief Skill for Claude Code

A Claude Code skill pair that automates your daily morning briefing — collecting news, reminders, and email todos, then creating a Notion page. After reading, discuss the most important article with Claude and save the analysis back to Notion.

---

## What It Does

```
Scheduled CronJob (e.g. 06:30)
  → /morning-brief
  → Reads Gmail Google Alerts (your subscribed topics)
  → Scrapes a government/industry news website
  → Checks Notion work journal for follow-up items
  → Checks Notion databases for overdue reminders & habits
  → Reads Gmail for unread emails needing action (last 3 days)
  → Creates a Notion page with the full briefing
     + blank deep-analysis template at the bottom

You (when you're ready)
  → /news
  → Claude shows today's briefing in your terminal
  → You pick the most important article to discuss
  → Claude guides the discussion
  → After discussion, Claude fills in:
       Notion page properties (title, category, importance, etc.)
       Background / Perspectives / Further Reading sections
  → "My Take" section — you fill this in yourself
```

---

## Prerequisites

| Requirement | Notes |
|---|---|
| **Claude Code** | Paid subscription required |
| **Gmail MCP** | Configured in Claude Code (`mcp__claude_ai_Gmail__*`) |
| **Notion MCP** | Configured in Claude Code (`mcp__notion__*`) |
| **Firecrawl MCP** | For scraping news websites |
| **Google Alerts** | Set up at alerts.google.com for your topics |
| **Notion databases** | See setup guide below |

---

## Notion Database Setup

You need four Notion databases. Create them (or use existing ones) and note their IDs.

**How to find a Database ID:**
Open the database in Notion → click Share → Copy link → the ID is the 32-character string in the URL.

### 1. News Database (required)

This is where morning-brief creates the daily page and /news writes the analysis.

**Required properties:**

| Property name | Type | Notes |
|---|---|---|
| Title | title | Article headline or date |
| Date | date | Article/briefing date |
| Category | multi_select | e.g. Industry Trends, Policy, AI |
| Source URL | url | Link to original article |
| Importance | select | Options: High, Medium, Reference |
| Fact | rich_text | Core fact in ≤30 words |
| Why/Reason | rich_text | Driving force behind the story |
| Relevance | rich_text | Impact on your role/organization |

**Required page sections (as headings in your page template):**
Background / Perspectives / Further Reading / My Take

### 2. Work Journal (optional but recommended)

Used to surface follow-up items in the morning brief.
Needs a "follow-up" or "next steps" field and a "recap date" date field.

### 3. Knowledge Base (optional)

Used to surface overdue review reminders.
Needs a "review date" date field and a "review completed" checkbox.

### 4. Task Tracker (optional)

Used to surface overdue tasks and daily/weekly/monthly habits.
Needs: task name (title), deadline (date), done (checkbox), recurrence (select: daily/weekly/monthly).

---

## Installation

### 1. Copy the skills to your Claude Code skills folder

```bash
# Default Claude Code skills location
cp -r skills/morning-brief ~/.claude/skills/
cp -r skills/news ~/.claude/skills/
```

If you use a custom skills path, copy there instead.

### 2. Fill in your configuration

Edit both `SKILL.md` files and replace the placeholders:

| Placeholder | Replace with |
|---|---|
| `YOUR_NEWS_DATABASE_ID` | Your news Notion database ID |
| `YOUR_JOURNAL_DATABASE_ID` | Your work journal database ID |
| `YOUR_KNOWLEDGE_DATABASE_ID` | Your knowledge base database ID |
| `YOUR_TASKS_DATABASE_ID` | Your task tracker database ID |
| `YOUR_NEWS_SOURCE_URL` | URL of a news page to scrape (e.g. a government environmental news page) |
| `YOUR_EMAIL` | Your email address (for future notification features) |

Also update the Notion property names in `news/SKILL.md` Step 5 to match your actual database property names.

### 3. Test manually

In Claude Code, type:
```
/morning-brief
```

Check that it creates a Notion page correctly. Then type:
```
/news
```

And walk through a discussion.

### 4. Set up the CronJob (optional)

Once manual testing works, schedule it to run automatically:
```
/schedule
```

Follow the prompts to set up a daily 06:30 run of `/morning-brief`.

---

## Google Alerts Setup

Go to [google.com/alerts](https://google.com/alerts) and create alerts for topics relevant to your work. Suggestions for sustainability/environment professionals:

- `[your organization name]`
- `waste management AI`
- `net zero industry`
- `ESG regulation [year]`
- `carbon tax policy [your country]`
- `AI workflow automation`
- `circular economy [your country]`
- `food waste management` (bridges food & environmental sectors)

Alerts are delivered to your Gmail — morning-brief reads them automatically.

---

## Customizing News Scoring

The importance scoring in `morning-brief/SKILL.md` Step 5 is designed around an environmental/sustainability role. Edit the scoring criteria and priority order to match your industry and role.

---

## How /news Works After Discussion

After discussing an article, Claude will:
1. Suggest renaming the page from "YYYY-MM-DD Morning Brief" to the article headline
2. Fill in all Notion properties (category, importance, source URL, fact, reason, relevance)
3. Write Background and Perspectives sections based on your discussion
4. Add related article links to Further Reading
5. Leave "My Take" blank — this is intentionally left for you

---

## Project Structure

```
morning-brief-skill/
├── README.md
└── skills/
    ├── morning-brief/
    │   └── SKILL.md    ← CronJob skill: data collection + Notion page creation
    └── news/
        └── SKILL.md    ← Interactive skill: display briefing + discussion + Notion update
```

---

## Contributing

Feel free to fork and adapt for your industry. If you improve the news scoring logic, Notion schema, or add new data sources, PRs are welcome.

---

## License

MIT
