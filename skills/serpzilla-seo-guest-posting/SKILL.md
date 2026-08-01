---
name: serpzilla-seo-guest-posting
description: Buy SEO guest posts and link insertions on trusted donor sites via the Serpzilla platform — create projects, search and filter donors by DR/traffic/price/language, upload articles or URLs with anchor texts, purchase placements, and manage their lifecycle. Use whenever the user wants to build backlinks, find donor sites for guest posting, buy or approve link placements, or do anything with Serpzilla, even if they don't mention it by name.
---

# Serpzilla SEO Guest Posting

## Overview

This skill guides an agent through SEO link-building on the [Serpzilla](https://serpzilla.com) platform: creating projects for promoted websites, searching donor sites, uploading articles or URLs with anchor texts, buying placements (guest posts and link insertions), and managing the placement lifecycle. Interaction with the platform happens entirely through the **Serpzilla MCP server**, which the user connects themselves.

## Prerequisites

- A Serpzilla **advertiser** account. Get an API token in the [Passport → Security](https://passport.serpzilla.com/security/token/) section.
- The **Serpzilla MCP server** connected to the agent at `https://mcp.serpzilla.com/mcp`. Authentication is handled by the server through a built-in OAuth flow — the user only provides the URL when connecting. No Docker, Node.js, or CLI tools are required.

## How tool names appear

The MCP server exposes tools by their bare names (e.g. `get_user_info`, `create_project`). Different host agents may add a prefix — for example, Claude surfaces them as `mcp__serpzilla__get_user_info`. This skill refers to tools by their **bare names**; use whatever form your agent expects.

The `authorize` tool exists on the server but runs automatically — you do not call it yourself. The server refreshes tokens on its own.

## Verify the connection

Before doing anything else, call `get_user_info`. If it returns a `balance` (and `login`/`email`), the MCP is connected and authenticated. The top-level `balance` field is the available account balance in the user's currency (`userWmPriorityCurrency`). To top up the balance, point the user to https://passport.serpzilla.com/deposit/ — do not attempt to add funds programmatically.

## Domain model

Understanding these entities makes the rest of the workflow make sense:

- **Project** — a container for everything below. Created for a specific domain, though other domains can also be promoted inside it.
- **URL** — the promoted page that placements link back to. Linked to a project.
- **Article** — a full HTML article uploaded by the advertiser for the `news` format. Goes through moderation, so it has a status. Linked to a project.
- **Text** — the anchor text used in the link. When an Article is uploaded, the Text and URL are derived automatically. URLs and Texts can also be added manually via `add_texts` — this is how you supply additional promoted pages and anchor texts for formats where the webmaster writes the content (`review`, `link`, `archive`).
- **Content** — a general term covering both Article and Text, each with its own status transitions.
- **Placement** — the purchased guest post / link insertion. Moves through a status workflow. Linked to a Text and a URL.

## Placement formats

| Format    | What it is                          | Content provided by        | Required to buy        | Notes                                                                                  |
|-----------|-------------------------------------|----------------------------|------------------------|----------------------------------------------------------------------------------------|
| `news`    | Advertiser's Article (Guest Post)   | Advertiser (the client)    | `article_id`, `url_id` | `text_id` not needed — the article is fully supplied by the client.                    |
| `review`  | Publisher's Article (Guest Post)    | Publisher (the webmaster)  | `url_id`, `text_id`    | The webmaster writes a review article. **More expensive.**                             |
| `link`    | In The News (Link Insertion)        | Publisher (the webmaster)  | `url_id`, `text_id`    | The webmaster embeds the link naturally into a news piece.                             |
| `archive` | In The Archive (Link Insertion)     | N/A (article already exists) | `url_id`, `text_id` | The link is placed into an already-published article.                                  |

## Workflow

### 1. Create a project for the promoted website

```
create_project(domain="example.com")
```

Pass the domain **without** `https://`. Save the returned project **id** (`PROJECT_ID`) — you'll reuse it everywhere. Creating a project also auto-generates one **URL** (the domain itself) and one default **Text**.

To see existing projects instead of creating one:

```
list_projects()
```

This returns `projectsInGroupList[]`, where each entry has `id` (the `PROJECT_ID`), `domain`, `name`, and `firstUrlId` (the id of the auto-created URL — handy when you need a `url_id` quickly).

All arguments are optional. Use them for pagination, sorting, and filtering:

```
# Page 2, 50 projects per page, newest first
list_projects(page=2, limit=50, order_by="byCreatedAt", order_type="desc")

# Only active projects, sorted by name
list_projects(project_filter="isActive", order_by="byProjectName")

# Fetch specific projects by ID
list_projects(projects_ids=[123, 456])
```

| Argument | Values | Description |
|----------|--------|-------------|
| `page` | `1+` | Page number (starts at 1) |
| `limit` | `20`, `50`, `100`, `250` | Projects per page |
| `order_by` | `byProjectName`, `byLinksCount`, `byUrlsCount`, `byAutobuyersCount`, `byMonthlyBudget`, `byCreatedAt` | Sort field |
| `order_type` | `asc`, `desc` | Sort direction |
| `project_filter` | `all`, `isActive`, `isArchived`, `withErrors` | Status filter |
| `projects_ids` | `[id, …]` | Filter by specific project IDs |

> **Searching by name?** The API does not support substring search. Sort by `order_by="byProjectName"` and filter the results client-side, or use `projects_ids` if you know the IDs.

### 2. Prepare content

For the `news` format you must upload an article **before** buying. The article is reviewed by Serpzilla moderators; only `approved` articles can be purchased.

```
add_article(
  project_id=PROJECT_ID,
  url="https://example.com/landing-page",
  title="Internal article title",
  meta_title="Meta title for the publisher",
  body="<p>HTML article with your <a href='https://example.com'>backlink</a> already inserted.</p>"
)
```

- `url` — the full promotion URL, including `https://`.
- `title` — internal only (visible in the Serpzilla UI).
- `meta_title` — used by the publisher when posting.
- `body` — HTML content, with backlink(s) already inserted (must contain at least one link). Max 64 000 characters.

The call returns `articleId` and `urlId`. Moderation typically takes 1–2 business days. If rejected, read the reason and re-upload a corrected version.

Check content status:

```
get_project_content_list(project_id=PROJECT_ID)
```

For `review`, `link`, and `archive` formats, **no article is needed** — the purchase uses `url_id` and `text_id`, and the webmaster writes the surrounding content.

#### Adding URLs with anchor texts (`add_texts`)

For formats where the webmaster writes the content, you provide the promoted URL and anchor text via `add_texts`. This is how you add pages beyond the auto-generated domain URL, and anchor texts beyond the default one.

```
add_texts(
  project_id=PROJECT_ID,
  urls_texts=[
    {
      url: "https://example.com/blog/seo-guide",
      texts: [
        {text: "See our #a#complete SEO guide#/a# for details"},
        {text: "Learn more about #a#link building#/a# strategies"}
      ]
    },
    {
      url: "https://example.com/services/",
      texts: [{text: "Professional #a#SEO services#/a#"}]
    }
  ]
)
```

- `url` — the full promotion URL, including `https://`.
- `texts[].text` — the anchor text. Use `#a#...#/a#` markup to mark which portion becomes the link. One URL can have multiple anchor texts (each becomes a separate `text_id`).
- `keywords` (optional) — key phrases for the webmaster to target when writing content.

The call returns `texts[]` with `urlId` and `textsList` (array of text IDs) for each URL, plus any `urlErrors` or `textErrors` if a URL or anchor was rejected by the validator.

**When to use which:**

| Need | Tool |
|------|------|
| Advertiser's full article (`news` format) | `add_article` |
| Just URLs + anchor texts (`review` / `link` / `archive`) | `add_texts` |

### 3. Search for donor sites

```
search_sites(project_id=PROJECT_ID, link_type="link")
```

Valid `link_type` values: `news`, `review`, `link`, `archive`. The result is a list of matching donor sites with summary metrics.

Common filters you can add to narrow the search:

- `language` — site language (e.g. `"en"`)
- `price_from`, `price_to` — price range
- `ahrefs_dr_from`, `ahrefs_dr_to` — Ahrefs Domain Rating
- `moz_da_from` — Moz Domain Authority; `moz_spam_score_to` to cap spam
- `majestic_cf_from`, `majestic_tf_from` — Majestic Citation / Trust Flow
- `traffic_ahrefs_from`, `traffic_semrush_from` — traffic volume
- `pages_google_from` — minimum pages in Google index
- `domain_level` — `0` all, `2` second-level only, `3` third-level only
- `days_old_whois_from` — minimum domain age in days
- `external_links_to` — cap on external links on the donor page
- `avg_placement_time_from` / `avg_placement_time_to` — placement speed window
- `placement_probability_from` — minimum placement probability (%)
- `keywords` — comma-separated keywords
- `site_url` — restrict to a specific donor site

Get full details (price, DR, traffic, language, placement examples) for a candidate:

```
get_site_info(site_id=SITE_ID)
```

### 4. Buy a placement

**Stop. Read the financial guardrails first.** Never call `purchase_placement` without explicit user confirmation.

Once you have the pieces:

- `PROJECT_ID`
- `SITE_ID` (from search results)
- For `news`: an **approved** `article_id` and a `url_id`
- For `review` / `link` / `archive`: a `url_id` and a `text_id` (obtained from `add_texts` or the auto-generated defaults from project creation)
- `search_history_id` — always pass the **empty string** `""`

Examples per format:

```
# news — Advertiser's Article
purchase_placement(project_id=PROJECT_ID, link_type="news",
  site_id=SITE_ID, article_id=ARTICLE_ID, url_id=URL_ID, search_history_id="")

# review — Publisher's Article
purchase_placement(project_id=PROJECT_ID, link_type="review",
  site_id=SITE_ID, url_id=URL_ID, text_id=TEXT_ID, search_history_id="")

# link — In The News (Link Insertion)
purchase_placement(project_id=PROJECT_ID, link_type="link",
  site_id=SITE_ID, url_id=URL_ID, text_id=TEXT_ID, search_history_id="")

# archive — In The Archive (Link Insertion)
purchase_placement(project_id=PROJECT_ID, link_type="archive",
  site_id=SITE_ID, url_id=URL_ID, text_id=TEXT_ID, search_history_id="")
```

Optional for any format: `is_content_need_approval=true` makes the placement wait for the advertiser's approval before the publisher starts — useful when you want to review the final context.

View purchased placements:

```
get_project_placements(project_id=PROJECT_ID)
```

### 5. Manage placements

After purchase, each placement moves through a status workflow. You influence it with bulk actions:

```
perform_placement_action(placement_ids=[PLACEMENT_ID], action="approve_seo")
```

`placement_ids` is a **list** (you can act on several at once). `action` is one of the SEO actions described in **`references/placement_actions.md`**.

**Read `references/placement_actions.md` before taking any placement action.** It lists every action, the status it requires, the status it leads to, and important timeouts (for example, some cancellations are only allowed within 1–72 hours, or after 15 days). Acting without understanding these can forfeit money or remove live backlinks.

## Financial guardrails

These are not bureaucracy — every purchase and several placement actions move **real money**, and mistakes are hard or impossible to reverse. Treat them accordingly.

**Before `purchase_placement`**:

1. Call `get_user_info` and read the current `balance`.
2. Call `get_site_info(site_id=SITE_ID)` and read the price for the chosen donor.
3. Show the user: the site URL and price, the current balance, and the **remaining balance** after purchase (balance − price).
4. Ask for **explicit confirmation**. Buy only after they say yes.

**Before `perform_placement_action`**:

1. Explain what the action does and its financial impact. In particular:
   - `approve_seo` / `approve_content_seo` release funds to the publisher.
   - `cancel_seo` / `cancel_from_*` may trigger refunds or forfeiture.
   - `terminate_seo` permanently removes an already-live backlink.
   - `approve_from_arbitration_seo` resolves a dispute in the advertiser's favor.
2. Ask for **explicit confirmation**. Act only after they say yes.

If the user does not confirm, do not proceed. It's fine to wait.

## Pre-purchase checklist

Run through this before buying — it catches almost every common failure:

- [ ] `PROJECT_ID` known (created or found via `list_projects`)
- [ ] For `news`: an article in `approved` status (`get_project_content_list`)
- [ ] `SITE_ID` selected, with acceptable metrics (`get_site_info`)
- [ ] `url_id` (and `text_id` for `review`/`link`/`archive`) known — add more via `add_texts` if needed
- [ ] `search_history_id` set to `""`
- [ ] Balance covers the price, and the user has explicitly confirmed

## Troubleshooting

- **"My article isn't usable"** — it's likely still in moderation. Check its status with `get_project_content_list`; only `approved` articles can be purchased. Rejections usually include a reason — fix and re-upload.
- **"Not enough funds"** — confirm with `get_user_info` (`balance`). Send the user to https://passport.serpzilla.com/deposit/ to top up. Don't try to add funds through the API.
- **"My placement is stuck"** — match its current status against the table in `references/placement_actions.md`; that table tells you which actions are available from each status and any timeouts that apply.
- **MCP/auth errors or 401s** — the server refreshes tokens automatically. If errors persist, the usual fix is to reconnect the Serpzilla MCP in the host agent's settings.
- **"How many placements need my attention?"** — `get_user_info` returns `seoCounters`, e.g. `nofLinksStatusNeedApprove`, `nofPlacementCheck`, `nofPlacementsContentOnApproval`, `nofClarification`. Use these to surface what needs action.

## Additional resources

- [Serpzilla API documentation](https://serpzilla.com/api/)
- Serpzilla MCP endpoint: `https://mcp.serpzilla.com/mcp`
- [MCP server source (Docker/stdio variant)](https://github.com/stanislav-reshetnev/serpzilla-mcp-server)
- Agent Skills standard: [agentskills.io](https://agentskills.io)
