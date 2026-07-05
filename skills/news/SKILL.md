---
name: news
description: Display today's morning briefing and guide a deep-dive discussion. After discussion, organize notes into your Notion news database. Triggered by /news or phrases like "discuss today's news" or "show me today's briefing".
---

# News — Daily Briefing Discussion

Retrieve today's morning brief, guide a discussion on the most important article(s), then write up the analysis into Notion.

## When to Use

- Type `/news`
- Say "discuss today's news" or "show me today's briefing"

---

## ⚙️ Configuration Required

```
YOUR_NEWS_DATABASE_ID  → same database used in morning-brief
YOUR_ORG_NAME          → your organization name (used in discussion prompts)
YOUR_ROLE              → your role (used to frame relevance questions)
```

Also update the Notion property names in Step 5 to match your actual database column names.

---

## Main Flow

### Step 1: Find Today's Briefing Page

Use `mcp__notion__API-post-search`:
- query: `YYYY-MM-DD` (today's date — matches the page title created by morning-brief)
- filter: `{"value": "page", "property": "object"}`

**Found:** proceed to Step 2.

**Not found:** ask the user:
> "Today's morning brief hasn't been created yet. Would you like to run /morning-brief now?"

### Step 2: Display the Briefing

Read page content with `mcp__notion__API-get-block-children`, then display in Claude Code:

```
📰 Morning Brief YYYY-MM-DD
══════════════════════════

1. [Headline] 🔴 High
   [Summary]
   💼 Relevance: [...]
   🔗 [Source link]

2. [Headline] 🟡 Medium
   ...

📋 Work Follow-ups
[Items]

📬 Gmail Todos
- [Item 1]

📅 Reminders
- [Item 1]
```

Then ask:
> "Which article interests you most, or feels most relevant to your work today?"

---

### Step 3: Deep Discussion

Guide a discussion on the selected article(s). You may discuss 2-3 related articles together.

**Discussion questions (ask one at a time):**
1. What's your first reaction to this?
2. What do you think is driving this — regulation? market pressure? technology?
3. How would this affect your work at [YOUR_ORG_NAME]?
4. What direction do you see this pushing your industry?

**If multiple related articles come up:**
- Help map the connections between them
- Ask: "Do you want to combine these into one page, or pick one as the main focus with others in Further Reading?"

**While discussing, gradually build up:**
- Background (where this story comes from, beyond just this article)
- Perspectives (government / industry / NGO angles)
- Further Reading links

When discussion winds down:
> "Ready to write this up? Which article should be the main focus for this Notion page?"

---

### Step 4: Confirm Before Writing

Summarize what you'll fill in and ask for confirmation:

> Here's what I'm planning — does this look right?
>
> **Page title:** [Article headline]
>
> **Properties:**
> - Category: [suggestion]
> - Importance: 🔴 High / 🟡 Medium / ⚪ Reference
> - Source URL: [link]
> - Fact: [core fact in ≤30 words]
> - Why: [driving force behind the story]
> - Relevance to me: [impact on your role / organization]
>
> **Page content:**
> - Background: [summary from discussion]
> - Perspectives: [summary from discussion]
> - Further Reading: [list of related article links]
>
> **"My Take"** — left blank for you to fill in yourself.
>
> Any changes?

After confirmation, proceed to Step 5.

---

### Step 5: Update the Notion Page

**Update properties** with `mcp__notion__API-patch-page`:

```json
{
  "page_id": "[today's brief page ID]",
  "properties": {
    "YOUR_TITLE_PROPERTY": {"title": [{"text": {"content": "[Article headline]"}}]},
    "YOUR_CATEGORY_PROPERTY": {"multi_select": [{"name": "[Category]"}]},
    "YOUR_IMPORTANCE_PROPERTY": {"select": {"name": "[High/Medium/Reference]"}},
    "YOUR_SOURCE_URL_PROPERTY": {"url": "[Source URL]"},
    "YOUR_FACT_PROPERTY": {"rich_text": [{"text": {"content": "[Core fact ≤30 words]"}}]},
    "YOUR_REASON_PROPERTY": {"rich_text": [{"text": {"content": "[Driving force]"}}]},
    "YOUR_RELEVANCE_PROPERTY": {"rich_text": [{"text": {"content": "[Relevance to role]"}}]},
    "YOUR_DATE_PROPERTY": {"date": {"start": "YYYY-MM-DD"}}
  }
}
```

**Update page content blocks:**
1. `mcp__notion__API-get-block-children` — get all blocks on the page
2. Find the heading_2 blocks for Background, Perspectives, Further Reading
3. Update the paragraph block below each heading using `mcp__notion__API-update-a-block`

**Further Reading format** (one bulleted_list_item per link):
```json
{
  "type": "bulleted_list_item",
  "bulleted_list_item": {
    "rich_text": [
      {"type": "text", "text": {"content": "[Article title] — "}},
      {"type": "text", "text": {"content": "[Media name]", "link": {"url": "[URL]"}}}
    ]
  }
}
```

**Leave "My Take" block unchanged** — the user fills this in manually.

---

### Step 6: Done

```
✅ Notion updated
📄 [Page title]
🔗 https://www.notion.so/{page-id-without-hyphens}

💬 "My Take" is left blank — fill that in when you're ready.
```

---

## Database Spec

**Database ID:** `YOUR_NEWS_DATABASE_ID`

**Typical property types:**
- Title (title type) — article headline
- Date (date type) — article date
- Category (multi_select type) — topic tags
- Source URL (url type) — original article link
- Importance (select type) — High / Medium / Reference
- Fact (rich_text type) — core fact in ≤30 words
- Why/Reason (rich_text type) — driving force behind the story
- Relevance (rich_text type) — impact on your role/organization

**Page content sections (heading_2 blocks):**
Background / Perspectives / Further Reading / My Take

---

## Notion API Notes

- Adding new multi_select options: just pass the new name — Notion creates it automatically.
- To update block content: get block ID from `get-block-children`, then use `update-a-block` on the paragraph below each heading.
- Page URL format: always use `https://www.notion.so/{32-char-id}` (the `app.notion.com` format returned by the API may not open correctly in all browsers).
