---
name: apify-threads-research
description: This skill should be used when the user asks to "scrape Threads", "get posts from a Threads account", "search Threads for a keyword or hashtag", "monitor brand mentions on Threads", "get replies under a Threads post", "find Threads accounts about a topic", "get follower counts / bio / emails of Threads profiles", or wants Meta Threads posts, replies, engagement (likes, replies, reposts, views) or profile data as rows. It picks the right Apify Actor for accounts, keyword search, replies or profiles, builds a valid input, keeps the run cheap and works without a Threads login. Do not use it for Instagram, X/Twitter, Bluesky or Facebook.
license: MIT
metadata:
  version: "1.0"
  category: social-media
  keywords: "threads, meta threads, threads scraper, social listening, brand monitoring, hashtag, influencer research, threads profiles, engagement, apify"
---

# Research Meta Threads with Apify: posts, search, replies, profiles

Read public Threads data without logging in: an account's posts, keyword and hashtag search, the replies under a post, and profile facts (followers, bio links, emails and phones from the bio).

Disclosure: the Actors this skill routes to are paid (pay per result) and are built and maintained by the author of this skill. Links carry no affiliate parameters.

## Example prompts

Prompts this skill handles:

- "Get the last 50 Threads posts from @natgeo with likes and replies."
- "What are people saying on Threads about 'AI agents' this week? Top 30 posts."
- "Pull the replies under https://www.threads.net/@zuck/post/DfihFPxkaZ1."
- "Follower counts and bio emails for these Threads accounts: nasa, natgeo, vogue."

Out of scope (the boundary):

- "Get this account's Instagram posts" or "search X/Twitter" — different networks; use an Instagram or X skill.
- Private accounts, DMs, posting or liking — this skill only reads what a logged-out visitor sees.

## Prerequisites

- Apify account ([sign up](https://apify.com))
- Authentication via one of:
  - `apify login` (OAuth, if using the Apify CLI)
  - `APIFY_TOKEN` environment variable
  - Token from [Apify Console → Settings → Integrations](https://console.apify.com/settings/integrations)

## Workflow

1. **Classify the request.** Account names → posts of those accounts. A topic, phrase or hashtag → keyword search. A post link → replies (the "thread"). "Who are the accounts / followers / contacts" → profiles.
2. **Fetch the live schema** of the chosen Actor before the first call (fields can change).
3. **Start small.** 10–50 posts per account or keyword answers most questions; `maxPosts` is per account or keyword, not per run. Add `maxItems` as a hard ceiling for the whole run.
4. **Filter in the input, not afterwards.** `postedAfter`, `minLikes`, `languages`, `filterKeywords` drop rows before they are billed.
5. **Report.** A table: author, date, text (shortened), likes, replies, reposts, views where present, link. Say how many rows came back and what the run cost.

## Actor routing

| User need | Actor ID | Tier | Best for |
|-----------|----------|------|----------|
| Posts of accounts; also search, replies, profiles in one Actor via `mode` | `lergassy/threads-scraper` | community | Default choice: `usernames`, `keywords` or `postUrls`, date window, engagement filters, scheduled monitoring |
| Keyword or hashtag search only | `lergassy/threads-search-scraper` | community | Top + recent + tag result pages merged and de-duplicated |
| Replies under a post, or every reply an account wrote | `lergassy/threads-replies-scraper` | community | `mode: "thread"` with `postUrls`; `mode: "replies"` with `usernames` |
| Profile facts for a list of accounts | `lergassy/threads-profile-scraper` | community | Followers, bio, bio links, website, emails and phones in the bio, linked Instagram, latest post |

`Tier` = `apify` (Apify-maintained, prefer) or `community` (third-party).

## Inputs that work

All examples below were run on 20 Nov 2026 and returned rows in 5–10 seconds.

Fetch the schema first:

    apify actors info "lergassy/threads-scraper" --input --json \
      --user-agent lergassy-agent-skills/apify-threads-research 2>/dev/null

**Posts of an account** (10 rows in 6 s):

    apify actors call "lergassy/threads-scraper" \
      -i '{"usernames":["zuck"],"maxPosts":10,"includeViews":false}' \
      --json --user-agent lergassy-agent-skills/apify-threads-research 2>/dev/null

Usernames may be bare (`zuck`), with `@`, or profile URLs. Add `"postedAfter":"YYYY-MM-DD"` for a date window — paging stops at that date, so it also saves money.

**Keyword or hashtag search** (10 rows in 7 s):

    apify actors call "lergassy/threads-search-scraper" \
      -i '{"keywords":["coffee"],"maxPosts":10}' \
      --json --user-agent lergassy-agent-skills/apify-threads-research 2>/dev/null

Useful fields: `searchTypes` (`top`, `recent`, `tags`), `strictKeywordMatch: true` (drop posts that do not contain the words), `minLikes: 1` (drops zero-engagement filler), `languages` (`["en"]`).

**Replies under a post** (post + 10 replies in 10 s):

    apify actors call "lergassy/threads-replies-scraper" \
      -i '{"mode":"thread","postUrls":["https://www.threads.net/@zuck/post/DfihFPxkaZ1"],"maxRepliesPerPost":10}' \
      --json --user-agent lergassy-agent-skills/apify-threads-research 2>/dev/null

**Profiles** (2 rows in 5 s):

    apify actors call "lergassy/threads-profile-scraper" \
      -i '{"usernames":["natgeo","nasa"]}' \
      --json --user-agent lergassy-agent-skills/apify-threads-research 2>/dev/null

**Row fields.** Posts: `url`, `text`, `createdAt`, `username`, `fullName`, `isVerified`, `likeCount`, `replyCount`, `repostCount`, `quoteCount`, `viewCount`, `mediaType`, `images`, `videos`, `hashtags`, `mentions`, `isReply`, `replyToUsername`, `language`. Profiles: `username`, `fullName`, `biography`, `bioLinks`, `websiteUrl`, `emails`, `phones`, `followerCount`, `isVerified`, `instagramUrl`, `latestPostAt`, `latestPostUrl`.

**Read the rows.** The call returns `defaultDatasetId`:

    apify datasets get-items DATASET_ID --format json \
      --user-agent lergassy-agent-skills/apify-threads-research 2>/dev/null

Rows with `type` of `notice`, `error` or `summary` are explanations and run totals, not posts — read them to the user instead of counting them.

## Calling Actors — choose your interface

The commands above use the Apify CLI. The same Actors and inputs work through the Apify MCP connector at <https://mcp.apify.com> (`call-actor` with the same input JSON; `https://mcp.apify.com/?tools=lergassy/threads-scraper` pins the Actor as a tool) or an MCP client such as [mcpc](https://github.com/apify/mcpc).

## Troubleshooting

- **Search returns only ~20–25 posts per keyword.** Threads shows a logged-out visitor one page per query and result type. Keep `expandQueries: true` (default) and all three `searchTypes`; for more volume add related keywords rather than raising `maxPosts`.
- **Only ~25 replies under a big post.** That is what Threads shows a logged-out visitor; the Actor cannot see more.
- **`viewCount` is empty.** Threads publishes views for few posts. `includeViews` reads each post page and is slower — set it to `false` when views are not needed.
- **`profile not found or private` error row.** The handle does not exist on Threads (many brands use a different handle than on Instagram) or the account is private. Error rows are free; ask the user for the profile URL.
- **Rate-limit error rows.** Rerun with `"proxyConfiguration":{"useApifyProxy":true,"apifyProxyGroups":["RESIDENTIAL"]}`.
- **Search has no date filter at the source.** `postedAfter` is applied to rows after they are read; for "this week" use `searchTypes: ["recent"]`.
- For costs and monitoring see [references/gotchas.md](references/gotchas.md).
