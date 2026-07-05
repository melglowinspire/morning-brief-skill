---
name: morning-brief
description: Daily morning briefing automation. Reads Gmail Google Alerts, government news websites, Notion work journal, and Gmail emails to compile a daily briefing and create a Notion page. Triggered by CronJob at a scheduled time or manually with /morning-brief.
---

# Morning Brief

Daily morning briefing automation. Collects news, reminders, and email todos, then creates a Notion page with the full briefing.

## When to Use

- Triggered by CronJob at scheduled time (e.g. 06:30)
- Manual run: type `/morning-brief`

---

## ⚙️ Configuration Required

Before using this skill, fill in the values in the section below.

```
YOUR_NEWS_DATABASE_ID       → Notion database for news articles
YOUR_JOURNAL_DATABASE_ID    → Notion database for work journal
YOUR_KNOWLEDGE_DATABASE_ID  → Notion database for knowledge base
YOUR_TASKS_DATABASE_ID      → Notion database for task tracking
YOUR_NEWS_SOURCE_URL        → URL of a government/industry news page to scrape
YOUR_EMAIL                  → Your email address for notifications
```

See `README.md` for Notion setup instructions.

---

## Execution Steps

### Step 1: Read Today's Google Alerts (Gmail)

Use `mcp__claude_ai_Gmail__search_threads`:
- query: `from:googlealerts-noreply@google.com newer_than:1d`
- Read each thread with `mcp__claude_ai_Gmail__get_thread`
- Extract: headline, source, URL, summary

### Step 2: Scrape Industry News Source

Use Firecrawl MCP to scrape:
- URL: `YOUR_NEWS_SOURCE_URL`
- Extract the latest 5 articles: title, date, link
- Fallback to Playwright if Firecrawl fails

### Step 3: Read Work Journal Follow-ups

Use `mcp__notion__API-query-data-source` on `YOUR_JOURNAL_DATABASE_ID`.
Get the most recent entry and extract the "follow-up" field.
Skip if empty.

### Step 4: Check Notion Reminders

Query three databases for overdue or due-today items:

**A. Knowledge Base** (`YOUR_KNOWLEDGE_DATABASE_ID`)
- Filter: review date ≤ today AND review completed = false
- Format: `[Topic] — Review date: YYYY-MM-DD (N days overdue)`

**B. Work Journal** (`YOUR_JOURNAL_DATABASE_ID`)
- Filter: recap date ≤ today AND status = pending recap
- Format: `[Entry title] — Recap date: YYYY-MM-DD`

**C. Task Tracker** (`YOUR_TASKS_DATABASE_ID`)
- Overdue tasks: deadline ≤ today AND done = false (no recurrence set)
- Daily habits: recurrence = daily → show every day regardless of done status
- Weekly habits: recurrence = weekly AND done = false
- Monthly habits: recurrence = monthly AND done = false
- Format overdue: `⚠️ [Task name] — Deadline: YYYY-MM-DD (N days overdue)`
- Format habits: `🔁 [Task name] (daily/weekly/monthly)`

### Step 4b: Read Gmail Todos (Last 3 Days)

Use `mcp__claude_ai_Gmail__search_threads`:
- query: `newer_than:3d -from:googlealerts-noreply@google.com -category:promotions -category:social -category:updates is:unread`
- Identify emails needing action (questions, deadlines, requests)
- Max 5 items, format: `Subject → What needs to be done`

### Step 5: Prioritize and Score News

Score all collected news (Google Alerts + scraped source) using this matrix:

| Distance to work | Will it change things? | Importance |
|---|---|---|
| Close (directly affects your org/role) | Yes (new law/policy/major event) | 🔴 High |
| Close | No (trend continuation) | 🟡 Medium |
| Far (international reference) | Yes | 🟡 Medium |
| Far | No | ⚪ Reference only |

**Selection rules (max 6 articles, in priority order):**
1. Your organization directly mentioned
2. Core industry topics (waste management, circular economy, etc.)
3. AI workflow / automation
4. Carbon tax / ESG regulation
5. Net zero / sustainability
6. Government announcements from scraped source

For each article, summarize:
- Headline (one sentence, ≤30 words)
- Why it's happening (regulatory change? market pressure? tech breakthrough?)
- Relevance to your role (1 sentence, in blue text)

### Step 6: Create Notion Page

Use `mcp__notion__API-post-page` to create a page in `YOUR_NEWS_DATABASE_ID`.

**Properties:**
```json
{
  "標題": {"title": [{"text": {"content": "YYYY-MM-DD Morning Brief"}}]},
  "日期": {"date": {"start": "YYYY-MM-DD"}}
}
```

**Page content structure:** See section below.

---

## Page Structure

### Block 1: Briefing Header

```json
{"type": "callout", "callout": {
  "rich_text": [{"type": "text", "text": {"content": "Morning Brief YYYY-MM-DD"}}],
  "icon": {"type": "emoji", "emoji": "📰"},
  "color": "blue_background"
}}
```

### Block 2: News Items (max 6, sorted by importance)

For each article:
```json
{"type": "heading_3", "heading_3": {
  "rich_text": [{"type": "text", "text": {"content": "1. [Headline]"}}]
}}
{"type": "bulleted_list_item", "bulleted_list_item": {
  "rich_text": [{"type": "text", "text": {"content": "Source: [Media name]"}}]
}}
{"type": "bulleted_list_item", "bulleted_list_item": {
  "rich_text": [{"type": "text", "text": {"content": "Importance: 🔴 High / 🟡 Medium / ⚪ Reference"}}]
}}
{"type": "bulleted_list_item", "bulleted_list_item": {
  "rich_text": [{"type": "text", "text": {"content": "Summary: [2-3 sentences]"}}]
}}
{"type": "bulleted_list_item", "bulleted_list_item": {
  "rich_text": [
    {"type": "text", "text": {"content": "Relevance: [1 sentence]"}, "annotations": {"color": "blue"}}
  ]
}}
{"type": "bulleted_list_item", "bulleted_list_item": {
  "rich_text": [
    {"type": "text", "text": {"content": "Source: "}},
    {"type": "text", "text": {"content": "[URL]", "link": {"url": "[URL]"}}}
  ]
}}
{"type": "divider", "divider": {}}
```

### Block 3: Work Journal Follow-ups

```json
{"type": "heading_2", "heading_2": {
  "rich_text": [{"type": "text", "text": {"content": "📋 Work Journal Follow-ups"}}]
}}
{"type": "paragraph", "paragraph": {
  "rich_text": [{"type": "text", "text": {"content": "[Follow-up items, or '(No follow-ups today)']"}}]
}}
```

### Block 4: Notion Reminders

```json
{"type": "heading_2", "heading_2": {
  "rich_text": [{"type": "text", "text": {"content": "📅 Reminders"}}]
}}
```
One `bulleted_list_item` per reminder. If none: `(No reminders today)`

### Block 5: Gmail Todos

```json
{"type": "heading_2", "heading_2": {
  "rich_text": [{"type": "text", "text": {"content": "📬 Gmail Todos"}}]
}}
```
One `bulleted_list_item` per todo item.

### Block 6: Deep Analysis Template (blank, filled in after /news discussion)

```json
{"type": "divider", "divider": {}}
{"type": "callout", "callout": {
  "rich_text": [{"type": "text", "text": {"content": "Deep analysis section — filled in after /news discussion"}}],
  "icon": {"type": "emoji", "emoji": "💬"},
  "color": "gray_background"
}}
{"type": "heading_2", "heading_2": {
  "rich_text": [{"type": "text", "text": {"content": "Background"}}]
}}
{"type": "paragraph", "paragraph": {
  "rich_text": [{"type": "text", "text": {"content": "(Where does this story come from? Broader context beyond this article)"}, "annotations": {"color": "gray"}}]
}}
{"type": "heading_2", "heading_2": {
  "rich_text": [{"type": "text", "text": {"content": "Perspectives"}}]
}}
{"type": "paragraph", "paragraph": {
  "rich_text": [{"type": "text", "text": {"content": "(Government / Industry / NGO perspectives)"}, "annotations": {"color": "gray"}}]
}}
{"type": "heading_2", "heading_2": {
  "rich_text": [{"type": "text", "text": {"content": "Further Reading"}}]
}}
{"type": "paragraph", "paragraph": {
  "rich_text": [{"type": "text", "text": {"content": "(Related articles and links)"}, "annotations": {"color": "gray"}}]
}}
{"type": "heading_2", "heading_2": {
  "rich_text": [{"type": "text", "text": {"content": "My Take"}}]
}}
{"type": "paragraph", "paragraph": {
  "rich_text": [{"type": "text", "text": {"content": "(Your own perspective — fill this in yourself)"}, "annotations": {"color": "gray"}}]
}}
```

---

## Completion Report

```
✅ Morning brief created
📄 Notion page: [URL]
📰 News: N articles (High: X / Medium: Y / Reference: Z)
📬 Gmail todos: N items
📋 Work follow-ups: Yes / None
```

---

## Notes

- This skill reads Gmail and Notion data. Ensure you are comfortable with this data being processed by the AI model you are using.
- If a morning brief page already exists for today (title contains today's date), skip creation and report the existing page URL.
- Adapt the news scoring criteria to match your industry and role.
