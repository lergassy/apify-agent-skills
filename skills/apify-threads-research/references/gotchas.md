# Cost guardrails and gotchas

## What a run costs (pay per result, list prices on 20 Nov 2026)

| Event | Price per row (Free plan → higher Apify plans) |
|-------|-----------------------------------------------|
| Post or reply | $0.002 → $0.0014 |
| Profile (with details) | $0.004 → $0.0028 |
| Account found by keyword (`mode: "users"`) | $0.002 → $0.0014 |
| Scheduled monitoring run (`threads-scraper`, from 23 Nov 2026) | $0.005 per run |

- 50 posts cost about 10 cents on the Free plan. The run-summary row, notices and errors are free.
- `maxPosts` is per username or keyword: 5 accounts × 100 = 500 billed posts. Set `maxItems` to cap the run.
- `includeProfile: true` adds one profile row per account.

## Monitoring

`threads-scraper` with `monitoringStoreName` (e.g. `"brand-watch"`) returns only posts it has not returned before under that name. Use it for a daily schedule instead of re-reading the same posts.

## Things that surprise people

- Threads URLs exist on two domains (`threads.net` and `threads.com`); both work as input.
- Share links from the app are accepted in `postUrls`.
- Keyword search mixes languages; add `languages: ["en"]` if the user wants one language.
