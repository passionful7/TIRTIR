# TIRTIR Campaign Management

## Project Overview
This is a campaign management repository for **TIRTIR**, a client of **Madup Global**. It contains campaign assets, meeting documents, and related materials stored in Google Drive.

## Key Files
- `[Madup] TIRTIR .gsheet` — Main campaign tracking spreadsheet
- `[Madup]TirTir Meeting.gslides` — Meeting presentation slides

## Conventions
- File names should follow the `[Madup] TIRTIR <description>` naming pattern for client-facing documents
- Keep `.env` files and compressed archives out of version control (see `.gitignore`)
- Use Google Workspace file shortcuts (`.gsheet`, `.gslides`, etc.) — actual content lives in Google Drive

## Working with This Repo
- The repo is synced via Google Drive — avoid large binary files
- Campaign data and reports should reference the Google Sheets tracker as the source of truth

## Google documents (Google Slides, Google Docs, Google Sheets, etc.)
- Use GWS skills when you need to read, modify, or create Google documents

## Slack messaging rules

- **When sending Slack messages, call the API directly with the bot token instead of the MCP Slack tool** (avoids the “sent using claude” footer)
- Bot token: read from env var `SLACK_TOKEN` (Madup bot). Set it locally via `export SLACK_TOKEN=xoxb-...` (keep out of git)
- Channel read/search: MCP Slack tools are fine
- Message sending only: use the bot token API (`chat.postMessage`)

## TIRTIR US operations

- Work sheet: `1BLQfFg7TZaRuw6CYwe6rjGgyqThkrbjzbWRYiiyhIe4` (tab: Work)
- Internal Slack channel: `#mg_mkt_tirtir` (C0ALWLLQFU7)
- Client Slack channel: `#매드업x티르티르us` (C0AMP96KZHB)
- Before sharing work status: review every Slack thread for each Processing task (the sheet Result column is a snapshot — always reconcile with Slack as the source of truth)

### Slack work status — Processing task cross-check
- The sheet Result column is a **snapshot** — changes completed in Slack after the last update are not reflected
- Before sharing status, **check every Processing task's Slack thread** without exception
- `slack_read_channel` returns only parent messages → **use `slack_read_thread` to read replies**
- Determine status changes by matching keywords like "완료/확인/확정" (complete/confirmed/finalized)

### Slack thread deep-reading rules (CRITICAL)
Critical information is frequently buried in deep thread replies — not in parent messages. The following rules are **mandatory** when checking Slack:

1. **Read ALL threads with replies** — when `slack_read_channel` shows "Thread: N replies", ALWAYS call `slack_read_thread` for every thread with replies, regardless of how many replies it has. Threads with 20+ replies are the most likely to contain missed items.
2. **Read threads from BOTH channels** — always check `#매드업x티르티르us` (client) AND `#mg_mkt_tirtir` (internal). Convention changes and operational decisions often appear in only one channel.
3. **Scan for time/date mentions in replies** — thread replies often contain deadlines, schedule changes, and timezone decisions that are never restated in parent messages. Search for patterns: `PST`, `KST`, `EDT`, `시`, `종료`, `라이브`, `마감`.
4. **Watch the last 3 replies especially** — closing decisions, final confirmations, and schedule changes tend to appear at the very end of long threads (e.g., "4/5 PST 23시에 종료" in reply #24 of 24).
5. **Never assume parent messages are complete** — parent messages set the topic, but the actual decisions, modifications, and confirmations happen in replies. A parent saying "프로모션 준비 건" tells you nothing about timeline changes decided 20 replies later.

### Promotion lifecycle — always track all 3 phases
Every promotion has 3 phases. Each phase needs its own task row in the sheet:
1. **Setup** — creative, Shopify page/collection/discount, Klaviyo newsletter, banner
2. **Live** — go-live, QA, monitoring
3. **Close / Rollback** — disable banners, deactivate discount codes, hide promo pages, turn off ads

### Promotion sub-task breakdown (CRITICAL)
Every promotion setup MUST be broken into individual sub-tasks, not tracked as a single umbrella task. Follow the No Joke Sale pattern:
- **Shopify 세트/할인/GWP 세팅** (이동호)
- **Shopify 프로모션 페이지 + 메인배너/딜배너** (이동호)
- **Klaviyo 뉴스레터 세팅 및 발송** (이동호)
- **광고 소재 문구 작성** (황아인/김소형)
- **광고 캠페인 세팅 Meta/Google** (홍유니)
- **라이브 전 QA** (이동호)

When a TDL (To-Do List) or task breakdown is shared in a Slack thread with specific sub-tasks, deadlines, and assignees — each sub-task MUST become its own row in the sheet. Never collapse multiple distinct deliverables into one "프로모션 준비" umbrella row.

**Close/rollback tasks are frequently missed** because:
- They are mentioned in deep thread replies (e.g., reply #24 of a 24-reply thread), not parent messages
- They are embedded within operational updates alongside other info, not as standalone requests
- The sheet often only has setup/live rows — no one creates the closing row upfront

**When checking tasks, always ask:** "Is there a live promotion that has a scheduled end date? If yes, does a close/rollback task exist in the sheet?" If not, create one.

### Detecting new tasks from Slack threads
The cross-check process must not only **update existing tasks** but also **detect new tasks** that emerged in threads but were never added to the sheet. Watch for:
- Scheduled deadlines mentioned in replies (e.g., "4월 5일 PST 오후 11시에 종료 예정")
- Action items assigned to specific people (e.g., "동호님 크칙요청 예정")
- Requests from the client channel that imply new work not yet tracked
- Slack reminders set for future actions — these often signal a task that should be formalized in the sheet
- **Operational convention changes** (e.g., timezone standards, naming rules, communication protocols) — these are not tasks but affect all future tasks and must be reflected in CLAUDE.md immediately. Look for keywords like "기준", "변경", "앞으로", "이후부터" that signal a rule change rather than a one-time action

## Key resources

### Google Sheets
- Internal campaign sheet: `1BLQfFg7TZaRuw6CYwe6rjGgyqThkrbjzbWRYiiyhIe4`
  - Tabs: Work, UTM guide, Shopify ops guide, Shopify guide, Campaign board, Promotion summary

### Confluence
- UTM doc: Page ID `4701454355` (Space: MG)

### Shopify
- Store: `tirtir.global` (Shopify Plus)
- Theme: Broadcast
- Promotion setup: Page + Featured Collection
- Discounts: PowerX (per-product discount rates), BOGOS.io (GWP), Shopify Discounts
- Promotion rollback: manual (Launchpad not used)
- Markets: US market has separate custom settings — verify per country in the theme editor
- CDN cache: allow 5–15 minutes after theme changes to propagate. Use `?v=<random>` to bypass cache when checking

### Klaviyo
- Billing: paid directly by TIRTIR ($4,005/mo, email only)
- SMS: disabled
- Popups: use Sign-up Forms as event banners (remove email field)
- Send time: Optimal Sending Time (AI)
- Embedded forms: Shopify theme editor → Custom code section → `<div class="klaviyo-form-{ID}"></div>` + openForm script
- Email dark mode: set explicit background colors on all blocks (Transparent → inverts to black in dark mode)

### Ad accounts
- Meta Ad Account: `508479813992365`
- Meta Pixel: `879708003398074`
- Google Ads CID: `842-768-6345`
- GA4 Property: `244645253`
- Google MC: `648960094`

## Promotion timing convention (established 2026-04-03)

All promotion open/close times follow **PST 24-hour clock** (per TIRTIR 자연님 request):
- **Open:** PST 00:00 = **KST 16:00**
- **Close:** PST 23:00 = **KST 15:00**
- **Ad creatives OFF:** 1 hour before close (PST 22:00 = KST 14:00)
- Note: during daylight saving (Mar–Nov), actual timezone is PDT (UTC-7), but the team uses "PST" as the label regardless. The KST conversion stays the same.
- Always express times in **PST** when communicating externally, and include **KST equivalent** for internal coordination.
- When creating close/rollback tasks, set the due date/time to the PST close time converted to KST.

## Promotion setup process

1. Create collections (split by discount level)
2. In PowerX: set discount code + per-product rates (Cart qualifier: Item quantity > 0)
3. Create a Shopify page (connect collections via Featured Collection section)
4. In the theme editor, set main banner / deal banner (web + mobile)
5. Configure Klaviyo newsletter + `/discount/<code>?redirect=/pages/<landing>` URL for auto-apply
6. QA: web/mobile preview, discount tests, email test send
7. Go live
8. After the promo ends: manual rollback (discounts / banners / pages restored)

## Naming conventions

- Slack message status: `Complete` / `Processing` / `Not yet` / `Urgent`
- Due dates: `26.03.30` (YY.MM.DD)
- BB tasks: `[BB]` prefix

## Team

- 이동호 (CEO): Shopify/Klaviyo setup, UTM, contracts, data/tracking
- 홍유니: media migration, media mix, payment methods, permissions
- 황아인: creative planning/design comms, Klaviyo content, client communication
- 김소형: creative production

## Task prioritization guide

When deciding which tasks to surface or work on, apply the following logic in order:

### 1. Promotion launch deadlines come first
Promotions have hard live dates. A missed launch = missed revenue and broken client trust. Always check: **is there a promo going live within 3 days?** If yes, everything in its dependency chain is P0:
- Creative completion → Shopify page/collection/discount setup → Klaviyo newsletter → QA → live
- Any single delay in this chain pushes the entire launch. Surface blockers immediately.

### 2. Identify blocking dependencies
Some tasks gate others. Prioritize unblocking over doing independent work:
- No Meta page permission → can't create new ad sets
- No signed contract → business risk on all billable work
- No media mix → can't allocate budget or launch campaigns
- No creative assets → can't set up Shopify pages or Klaviyo emails
- No payment method switch → billing charged to wrong party

### 3. Overdue external deliverables need active follow-up
Tasks owned by external parties (BB, TIRTIR) don't resolve by waiting. If overdue:
- Check Slack for any partial delivery or update
- If silent, send a reminder through the appropriate channel (고객사 channel for TIRTIR, direct for BB)
- Escalate through 서우님/자연님 if no response after reminder

### 4. Revenue-impacting > operational improvement
- Campaign launches, ad account setup, creative delivery → directly tied to spend and revenue
- Reporting dashboards, SEO guides, app cleanup → important but won't lose money if delayed a few days

### 5. Time-based priority buckets
- **P0 (now):** Promo launching within 3 days, anything blocking ad spend, contract/legal deadlines
- **P1 (this week):** Tasks due within 7 days, overdue items from external parties, permissions/access gaps
- **P2 (next 2 weeks):** Infrastructure improvements, monitoring setup, optimization tasks
- **P3 (backlog):** Nice-to-haves with no hard deadline

### 6. Cross-check before reporting status
Before sharing task status with the team or client:
1. Read the 업무 sheet for current snapshot
2. Check Slack threads for every Processing task (sheet Result may be stale)
3. Flag any task where the due date has passed but status is still Processing or Not yet
4. Surface dependency risks: "Task X is blocked by Task Y which is overdue"

## Status judgment rules (prevent common mistakes)

### Complete vs. Processing vs. Not yet
Apply these rules strictly when setting task status:

1. **"Plan announced" ≠ Complete** — If someone says "예정입니다", "계획입니다", "구축할 예정", that is Processing or Not yet, NOT Complete. Only mark Complete when the **deliverable itself exists and is verified**.
   - ❌ "레버 엑스퍼트 연동 계획 수립" → Complete
   - ✅ "레버 엑스퍼트 연동 계획 수립" → Processing (plan exists, work not started)
   - ✅ "레버 엑스퍼트 대시보드 구축 완료, 리포트 발송 시작" → Complete

2. **Due date passed + person confirmed → Complete** — If the due date has passed AND the responsible person explicitly said they would do it (e.g., "종료처리하겠습니다"), mark as Complete unless there's evidence of failure. Don't leave as Processing just because you didn't personally verify.

3. **Shopify ops vs. Media ops are separate** — Never combine Shopify rollback (banners/pages/discounts, 이동호) and ad campaign termination (Meta/Google, 홍유니) into one task. They have different owners, different timelines, and different verification methods.

4. **Sheet-only items need Slack validation** — If a task exists in the sheet but has NO Slack discussion, flag it as "시트 기입 항목 — 슬랙 근거 미확인" before surfacing it as urgent. Don't assume sheet entries reflect active work.

5. **Duplicate detection** — Before creating a new task, scan the sheet for existing tasks covering the same scope. Consolidate rather than create duplicates.

### Processing task ownership check
Every Processing task must have a clear answer to: **"누가 지금 이걸 하고 있는가?"**
- If the answer is "nobody" → it's actually Not yet or blocked
- If the answer is "waiting for external party" → add who and since when to the Result column
- If the answer is "담당자 assigned but no activity" → flag as stale and verify via Slack

## Self-correction protocol

When a mistake is found in task status, Slack messages, or sheet updates:

1. **Identify the error** — What was wrong? (wrong status, wrong time, wrong scope, missing task)
2. **Find the root cause** — Which judgment rule was violated? (plan vs. complete, scope mixing, no Slack validation, etc.)
3. **Check for siblings** — Are there other tasks with the same type of error? Scan all non-Complete tasks.
4. **Fix immediately** — Update the sheet and any sent Slack messages.
5. **Update CLAUDE.md** — If the mistake reveals a new pattern not covered by existing rules, add a new rule. If an existing rule was insufficient, strengthen it.
6. **Never repeat** — The same root cause should not produce errors twice. If it does, the rule needs to be more specific.

## Slack status message rules

When sending task status updates to Slack:
- Always include a footer asking for corrections: "업무 현황에 수정/추가 사항이 있으시면 이 스레드에 코멘트 남겨주세요. 업무시트: [sheet link]"
- Both Slack AND the Google Sheet are sources of truth — check both, not just one
- Google Sheet items without Slack threads still need tracking and periodic verification
