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
```

---

## Main Flow

### Step 1: Find Today's Briefing Page

Use `mcp__notion__API-post-search` to find today's page:
- query: `YYYY-MM-DD Morning Brief` (today's date)
- filter: `{"value": "page", "property": "object"}`

**Found:** proceed to Step 2.

**Not found:** ask the user:
> "Today's morning brief hasn't been created yet. Would you like to run /morning-brief now?"

### Step 2: Display the Briefing

Read the page content with `mcp__notion__API-get-block-children`, then display clearly in Claude Code:

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
- [Item 2]

📅 Reminders
- [Item 1]
```

Then ask:
> "Which article interests you most, or feels most relevant to your work today?"

---

### Step 3: Deep Discussion

Guide a discussion on the selected article(s). You may discuss 2-3 related articles together.

**Discussion questions (ask one at a time, don't front-load all):**
1. What's your first reaction to this?
2. What do you think is driving this — regulation? market pressure? technology?
3. If you were at [your organization], how would this affect your work?
4. What direction do you see this pushing the industry?

**If the user mentions multiple related articles:**
- Help map the connections between them
- Ask: "Do you want to combine these into one analysis page, or pick one as the main focus with the others in Further Reading?"

**While discussing, gradually build up:**
- Background (where this story comes from)
- Perspectives (government / industry / NGO angles)
- Further Reading links (from related articles)

When discussion winds down, prompt:
> "Ready to write this up? Which article should be the main focus for this Notion page?"

---

### Step 4: Confirm Before Writing

Summarize what you'll write and ask for confirmation:

> Here's what I'm planning to fill in — does this look right?
>
> **Page title:** [Article headline]
>
> **Properties:**
> - Category: [suggestion]
> - Importance: 🔴 High / 🟡 Medium / ⚪ Reference
> - Source URL: [link]
> - Fact: [core fact in ≤30 words]
> - Why: [driving force behind the story]
> - Relevance to me: [impact on your role/organization]
>
> **Page content:**
> - Background: [summary]
> - Perspectives: [summary]
> - Further Reading: [list of related article links]
>
> **My Take** — left blank for you to fill in.
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
    "標題": {"title": [{"text": {"content": "[Article headline]"}}]},
    "分類": {"multi_select": [{"name": "[Category]"}]},
    "重要性": {"select": {"name": "[High/Medium/Reference]"}},
    "來源網址": {"url": "[Source URL]"},
    "事實": {"rich_text": [{"text": {"content": "[Core fact ≤30 words]"}}]},
    "原因": {"rich_text": [{"text": {"content": "[Driving force]"}}]},
    "對我的意義（與大豐的關係）": {"rich_text": [{"text": {"content": "[Relevance to role]"}}]},
    "日期": {"date": {"start": "YYYY-MM-DD"}}
  }
}
```

> **Note:** Rename the property keys to match your Notion database's actual property names.

**Update page content** (find block IDs, then update):
1. `mcp__notion__API-get-block-children` — get all blocks on the page
2. Find the heading blocks for Background, Perspectives, Further Reading
3. Update the paragraph blocks below each heading with `mcp__notion__API-update-a-block`

**Further Reading format** — one `bulleted_list_item` per link:
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
🔗 [Notion page link]

💬 "My Take" is left blank — fill that in when you're ready.
```

---

## Database Spec

**Database ID:** `YOUR_NEWS_DATABASE_ID`

**Property types to match:**
- Title field (title type)
- Date field (date type)
- Category field (multi_select type)
- Source URL field (url type)
- Importance field (select type) — options: High / Medium / Reference
- Fact field (rich_text type)
- Why/Reason field (rich_text type)
- Relevance field (rich_text type)

**Page content sections:**
Background / Perspectives / Further Reading / My Take

---

## Notion API Notes

- Adding a new multi_select option that doesn't exist yet: just pass the new name — Notion creates it automatically.
- To update block content: get the block ID from `get-block-children`, then use `update-a-block` on the specific paragraph block below each heading.
