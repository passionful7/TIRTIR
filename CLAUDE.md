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

### ⚠️ Default Slack scan scope (MANDATORY — NO EXCEPTIONS) ⚠️
**This rule has been violated multiple times. Treat it as a HARD requirement.**

**Default scope for every task audit: read ALL messages within the last 2 months from BOTH channels, including ALL thread replies.**

#### Required execution sequence
1. **Channel pagination — read until 2 months back**
   - Call `slack_read_channel` for `C0AMP96KZHB` (client) AND `C0ALWLLQFU7` (internal)
   - Each call returns `pagination_info.next_cursor` — if present, **call again with that cursor**
   - Continue until: (a) `next_cursor` is empty, OR (b) message timestamps are older than 2 months
   - Expected: minimum 5+ paginated calls per channel for active projects
   - **STOP CONDITION:** Only stop when you've reached the 2-month boundary. NEVER stop after the first page.

2. **Thread deep-read — every parent with replies**
   - For EVERY parent message that shows `Thread: N replies`, call `slack_read_thread`
   - This applies regardless of N (1 reply or 100 replies)
   - This applies regardless of whether you read the thread before — **threads can gain new replies**

3. **Old-parent + new-reply detection (CRITICAL)**
   - Some threads have parent messages OLDER than 2 months but RECENT replies within the 2-month window
   - To catch these, run these searches:
     - `mcp__claude_ai_Slack__slack_search_public_and_private` with query: `is:thread after:YYYY-MM-DD in:#매드업x티르티르us`
     - Same with `in:#mg_mkt_tirtir`
   - For every thread_ts returned, call `slack_read_thread` to read the full thread
   - This is the ONLY way to catch decisions made in long-running threads from earlier in the project

4. **Active-task thread re-read**
   - For every Processing/Not yet task in the sheet that has an associated thread, **always re-read that thread** in every audit cycle
   - New replies arrive constantly — yesterday's thread state is not today's

5. **Token limit handling**
   - If a single scan exceeds token limits, **launch parallel sub-agents** via the Agent tool, splitting by channel or date range
   - **NEVER** save time by skipping messages, truncating output, or guessing what's in unread threads
   - Sub-agents should report findings back; you consolidate

#### Forbidden shortcuts
- ❌ Reading only the first page of a channel
- ❌ Reading only parent messages without thread replies
- ❌ Skipping a thread because "I read it last time"
- ❌ Skipping a thread because "the topic doesn't sound new"
- ❌ Using a 1-week or 2-week window when the user asked for a thorough check
- ❌ Concluding a scan without verifying the 2-month boundary was reached
- ❌ Trusting summaries or recent memory instead of re-reading the source

#### Violation history (do not repeat)
- **2026-04-06:** Missed 23+ tasks because only first page of each channel was read
- **2026-04-07:** Missed BB(비욘드 보더스) 위임 task because existing thread `1775458171.961479` was not re-read after gaining 4 new replies on 4/7
- **2026-04-06:** Missed Lever Expert `us.team@madup.com` permission request because parent message announcement on 4/5 was on a later page of the internal channel
- **Pattern:** Every miss has been due to shallow scanning. The fix is always the same: scan deeper.

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

## Team & R&R (source of truth)

**The R&R tab in the work sheet is the source of truth for who does what and Slack tags.**
- Sheet: `1BLQfFg7TZaRuw6CYwe6rjGgyqThkrbjzbWRYiiyhIe4`, tab: `R&R`
- Columns: Name / Slack User ID / Slack Tag / Organization / Role / Responsibilities
- **Always read the R&R tab before assigning new tasks or drafting Slack messages.** Use the Responsibilities column to match each task to the correct owner. Use the Slack Tag column to format mentions in Slack messages.

### Current R&R quick reference (as of 2026-04-06)

Madup:
- 이동호 (`<@UCMKVTWP9>`) — CEO, Tech Lead. Shopify/Klaviyo/UTM/계약서/카탈로그 피드/데이터·트래킹/SEO/리포팅
- 홍유니 (`<@U02L20Z4MLP>`) — Media Lead. 매체 이관·믹스·운영·결제, 권한, 고객사 커뮤니케이션
- 황아인 (`<@U09PWQEJTMY>`) — Creative Planning. 소재 기획, 디자인팀 커뮤니케이션, Klaviyo 콘텐츠, 고객사 에셋 컨펌
- 김소형 (`<@U09FFQVH2FJ>`) — Creative Production. 소재 제작(이미지/영상/상세페이지)
- 하지현 (`<@U06ND4MHK5W>`) — Media Operations. Meta/Google 캠페인 세팅 지원

TIRTIR (Client):
- 권자연 (`<@U0AMPHRRLPP>`) — Marketing Manager. 프로모션 기획/컨펌/내부 조율
- 박서우 (`<@U09D7F2BVMW>`) — Marketing. 기획안 설명, 신규 요청(구독제/번들/GWP)
- 김현서 (`<@U0AMX4ACS68>`) — Media/Ad Ops. 광고 결제·계정 권한 부여

External:
- BB (Jake) — 이전 대행사 (Before BigBrothers). 인수인계 자료 미수령 5건 대기중

### Task assignment rule
When creating a new task, match the task's required skill to the R&R Responsibilities column. Example:
- "Shopify 할인 세팅" → 이동호 (Shopify owner)
- "Meta 캠페인 운영" → 홍유니 (Media Lead)
- "광고 소재 제작" → 김소형 (Creative Production)
- "고객사 프로모션 기획 컨펌" → 황아인 (Client Comms) or 홍유니 (Media Lead)

If a task spans multiple R&R areas (e.g., ABB 프로모션 전체), break it into sub-tasks per owner following the RC1 prevention rule.

### Slack message tagging rule (MANDATORY)
When sending any Slack message that references a person:
- **Always use the Slack Tag from the R&R sheet** (e.g., `<@UCMKVTWP9>` instead of "이동호")
- This triggers Slack notifications to the tagged person
- For team-wide mentions, use `<!subteam^S0ANECM32QY>` (tirtir usergroup)
- When listing task owners in a status message, tag every owner: `• Matcha Giveaway 크로스체크 (<@U02L20Z4MLP>, 4/7)`
- Exception: if the sheet already shows names in a table/list for readability, use the name + tag format: `홍유니 <@U02L20Z4MLP>`
- Before drafting any Slack status update, read the R&R tab to fetch the latest tags (handles for new team members may be added over time)

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

## Root cause registry (known mistake patterns)

Each time a mistake is found, log it here with root cause + prevention rule. Before any task audit, review this list to avoid repeating the same errors.

### RC1: Umbrella task collapse
**Symptom:** A large initiative (ABB promo, 레버 엑스퍼트 연동) tracked as one row instead of individual sub-tasks, causing specific deliverables/dependencies to be missed.
**Root cause:** Created one high-level task from a thread topic instead of extracting every concrete action item, deadline, and dependency mentioned in replies.
**Prevention:** Every initiative with multiple distinct deliverables → break into sub-task rows per owner. If a Slack message lists "1) X 2) Y 3) Z", each becomes a separate row. Apply the No Joke Sale sub-task pattern as the template.

### RC2: Shallow Slack scan (first page only)
**Symptom:** Missed critical messages buried beyond the first `slack_read_channel` page (e.g., 레버 엑스퍼트 4/5 announcement, promotion timezone rule change).
**Root cause:** Stopped reading after the first 15–20 messages without using `cursor` pagination.
**Prevention:** Default scan scope = 2 months, ALL messages, ALL thread replies, BOTH channels. Never stop at the first page. Use sub-agents in parallel if token limits are a concern.

### RC3: Thread-reply blindness
**Symptom:** Close/rollback times, convention changes, and action items buried in reply #20+ of long threads were missed (말차 PST 23:00 종료, 4/5 PDT→PST 전환 규칙). Also: existing threads that gained new replies after the previous audit were not re-read, causing client delegations (BB 구독제 문의 위임) to be missed.
**Root cause:** (1) Read only parent messages; assumed parent summarizes the thread. (2) When auditing a second time, only checked NEW parent messages and skipped re-reading EXISTING threads that may have had new replies.
**Prevention:**
- For every thread with ≥1 reply, call `slack_read_thread`.
- For threads with ≥20 replies, read from last to first — closing decisions usually appear at the end.
- **Re-read every existing tracked thread** during each audit cycle, even if you read it before — new replies arrive constantly. Compare reply timestamps against your last audit time; if any reply is newer, re-read the entire thread.
- **Threads tied to active sheet tasks** (especially Processing tasks) must be re-read in EVERY audit, no exceptions. The thread is the source of truth for status changes.

### RC4: Plan-vs-completion confusion
**Symptom:** Marked "레버 엑스퍼트 연동 계획 수립" as Complete when only the plan existed (no deliverable).
**Root cause:** Confused "announcement of intent" with "deliverable exists".
**Prevention:** Complete = verified deliverable exists. Plans, schedules, and announcements are Processing or Not yet, never Complete.

### RC5: Scope mixing (team boundaries ignored)
**Symptom:** Combined Shopify rollback (이동호) and ad campaign termination (홍유니) into one "말차 프로모션 종료" task with "광고 OFF" included, even though ads were on a separate timeline handled by a different owner.
**Root cause:** Used promotion name as the grouping key instead of owner + deliverable type.
**Prevention:** Before creating a task, ask "Is this one owner + one deliverable + one timeline?" If any of those are split, create separate rows. Shopify ops ≠ Media ops ≠ Creative ops.

### RC6: Sheet-only items surfaced without Slack validation
**Symptom:** "Google MC 피드 appeal" surfaced as this-week deadline even though no Slack discussion confirmed active work.
**Root cause:** Treated sheet Due date as authoritative without cross-checking Slack.
**Prevention:** Before surfacing any task as urgent/active, cross-reference with Slack. If no Slack evidence exists, label it "시트 기입 항목 — 슬랙 근거 미확인" and do NOT push it as a hard deadline.

### RC7: Client-dependency request chain broken
**Symptom:** 이동호 내부 공유: "Google Ads/GA4에 us.team@madup.com 권한 부여 필요" → 홍유니가 TIRTIR에 전달해야 하는데 태스크화되지 않아 전달 누락 위험.
**Root cause:** Internal messages that contain implicit requests for client action were not converted into standalone "고객사에 전달" tasks.
**Prevention:** When an internal message says "X를 해주시면/해주세요" AND X requires client action, create a task: "TIRTIR에 X 요청 전달" with owner = internal communicator. Track until TIRTIR confirms action.

### RC8: Operational convention changes missed
**Symptom:** "프로모션 시간대 PST 24시간제 기준으로 통일" 결정이 스레드 reply에서만 언급되어 CLAUDE.md에 반영되지 않았음.
**Root cause:** Treated convention changes as task updates instead of documentation updates.
**Prevention:** When a Slack message uses words like "기준", "변경", "앞으로", "이후부터", "통일" — the change must be reflected in CLAUDE.md immediately, not just in a task row. Operational rules govern all future tasks; missing them creates cascading errors.

### RC9: Row-number misalignment in sheet updates
**Symptom:** batchUpdate wrote values to row N+1 instead of row N because the array index ≠ sheet row number (header row offset).
**Root cause:** Confused 0-indexed array position with 1-indexed sheet row.
**Prevention:** When computing target rows from a read, always add 1 for the header row. After any batchUpdate to multiple rows, read back at least 2–3 of the updated rows to verify alignment before continuing.

### RC10: Incorrect Slack channel cursor usage
**Symptom:** Re-ran `slack_read_channel` without `cursor` and got the same first page again.
**Root cause:** Forgot to pass the `next_cursor` returned by the previous call.
**Prevention:** After every `slack_read_channel` call, inspect `pagination_info.next_cursor`. If present, the scan is NOT complete — continue paginating until `next_cursor` is empty or timestamps exceed the 2-month window.

### RC11: Silently rescheduling overdue tasks
**Symptom:** When a task's Due date had passed, silently moved the Due date forward (e.g., SEO 가이드 3/30 → 4/14) without explicit authorization, hiding the delay from the team.
**Root cause:** Treated Due date as a target that I can adjust, instead of a commitment that must be tracked.
**Prevention:**
- **NEVER silently reschedule a task.** Due dates can only be changed if (a) there is an explicit Slack message from the owner requesting the change, or (b) a human manually edits the sheet.
- When a task is overdue (Due date < today AND status ≠ Complete), **report it as overdue** in the Slack status message. Use phrases like "현재 일정보다 N일 지연", "Due 지남 (N일 초과)" — do NOT hide the delay.
- If a new target date is mentioned in Slack, log the evidence in the Result column (e.g., "이동호 4/6 재설정: 4/14로 연기 — 슬랙 ts X"), and only then update the Due date.
- If no Slack evidence exists, keep the original Due date and explicitly flag as overdue in reports.

### RC12: Bot message thread reply blindness (CRITICAL)
**Symptom:** Sent EOD status update via 매도비 bot. Team members (이동호) replied in the thread with status updates ("BOGOS 테스트 완료", "Google MC 4/30로 재설정", "오늘 신규 가이드 작성 예정"). Did NOT re-read the bot message thread in the next audit cycle, so missed all 4 status updates.
**Root cause:** Treated the bot message as "outbound only" — assumed no replies would come back since the bot author is myself. Failed to check threads on bot-authored parent messages.
**Prevention:**
- **Every audit cycle MUST re-read the most recent bot status update threads** in BOTH channels (`C0AMP96KZHB` and `C0ALWLLQFU7`).
- Bot messages are not "send and forget" — team members reply with status updates, corrections, and new task info.
- Search for `from:매도비` (bot user) or `from:U06AHN2ALJW` (bot user_id) in last 7 days, then `slack_read_thread` for each result.
- The thread of your own status messages is the **most likely place** to find direct human responses to specific tasks you listed.
- This rule supersedes RC3 — even self-authored threads must be re-read.

### RC13: "Permission request" tasks not checking for workarounds
**Symptom:** Created task "Shopify 카테고리 메뉴 권한 추가 요청" (R116) when 황아인 reported lack of menu permission. But 이동호's account already had the permission, so 황아인 added the product immediately via 이동호's URL. The task became unnecessary, but stayed in "Not yet" because the workaround wasn't tracked.
**Root cause:** When creating "request access to X" tasks, didn't track the workaround path. Assumed the formal permission grant was the only resolution.
**Prevention:**
- When creating any "권한 요청" / "access request" task, also check if a workaround exists (different account with permission, alternate path).
- If a workaround is used to bypass the permission requirement, **mark the original permission-request task as Complete** with note "우회 처리됨 — N의 계정으로 진행".
- Permission requests are often "nice-to-have" that get bypassed in practice. Always re-check 1-2 days after creation to see if the team found a workaround.
- Add to Result column: "권한 요청 vs. 우회 처리 결과 추적 필요"

### RC14: "Send/deliver" tasks staying in Processing after delivery
**Symptom:** R23 (4월 미디어 믹스 전달) was sent to TIRTIR on 4/2 23:46 by 홍유니 with the spreadsheet link and budget breakdown, but stayed in Processing because the Result column noted "ABB/Glow Green 잔여예산 배정 예정" (future allocation pending). The task's main deliverable (sending the media mix) was complete, but the "예정" phrase made me leave it as Processing.
**Root cause:** Confused "main deliverable complete" with "all related work complete". The task's deliverable was SENDING the media mix; future budget reallocation was a separate downstream activity.
**Prevention:**
- For tasks where the main deliverable is **send/deliver/share/transfer** (전달/발송/공유), Complete = the artifact was sent. Subsequent updates/iterations belong to separate tasks.
- When a Result column says "~예정" or "~진행 예정", check whether the original deliverable is already done. If yes, the task should be Complete and a separate task should track the future activity.
- Pattern: If task name contains "전달" / "발송" / "공유" / "delivery" / "send", verify the send happened, then mark Complete regardless of follow-up plans.
- After major project milestones (e.g., promo launches), re-audit all "전달" tasks to see if they should now be Complete.

### RC15: Partial-completion umbrella tasks not split
**Symptom:** R65 (말차 프로모션 종료/롤백) had 4 sub-deliverables: 메인배너 OFF / 딜배너 OFF / 할인코드 OFF / 프로모페이지 OFF / 랜딩페이지 정책 적용. 4 of 5 were complete on 4/6, but the task stayed in Processing because the 5th (랜딩페이지) was waiting for a separate decision (R83). Same pattern with R59 (매체 결제수단 변경) where Meta was complete and Google was blocked by an external system error.
**Root cause:** Kept umbrella tasks as Processing when 80%+ of sub-deliverables were done. The remaining residual was actually a separate, distinct deliverable that should have been tracked as its own row.
**Prevention:**
- When ≥70% of sub-deliverables in an umbrella task are complete and the residual is **a distinct dependency** (different blocker, different timeline, different decision needed), **split the residual into a new task** and mark the umbrella Complete.
- Examples of legitimate splits:
  - 말차 프로모션 종료 → 배너/할인 OFF (Complete) + 랜딩페이지 종료 정책 적용 (Not yet, blocked by policy decision)
  - 매체 결제수단 변경 → Meta 완료 (Complete) + Google 시스템 오류 follow-up (Not yet, blocked by external)
- Sub-task should reference the parent in Result column: "R65에서 분리"
- This prevents stale Processing tasks where the bulk of work is done but one residual makes the whole task look unfinished.
- Re-audit Processing tasks weekly: "Is this task ≥70% done with one distinct residual? If yes, split it."

### RC16: Audit checklist skipping (laziness)
**Symptom:** RC1–RC15 rules exist in CLAUDE.md, but were not actually applied step-by-step in each audit. Relied on memory/pattern recognition instead of forcing the checklist. Result: same patterns repeated across multiple audits (R20 광고 계약서 stayed "협상 교착" from 4/3 to 4/7 EOD even though 합의 was reached on 4/7 16:08).
**Root cause:** Treated CLAUDE.md rules as "background knowledge" rather than as a literal checklist that must be checked every time. Tried to short-cut by trusting Result column text from previous audit.
**Prevention:**
- **MANDATORY: Use `AUDIT_LOG.md` file** at the start of every audit. Add a new column for the current audit, then progress through each row, marking ✅ + KST timestamp + notes as you complete each step.
- AUDIT_LOG.md location: `/Users/ericklee/Library/CloudStorage/GoogleDrive-passionful7@gmail.com/My Drive/Madup/Global/Campaigns/TIRTIR/AUDIT_LOG.md`
- AUDIT_LOG.md is **single source of truth**: it contains the checklist (as rows) AND the execution history (as columns)
- **Do not skip any row** even if you "remember" the answer
- **Treat the Result column as STALE by default** — re-verify it against the latest Slack thread, not from memory
- After audit completion, if any RC violation is found, update CLAUDE.md (rule) and add the new row to AUDIT_LOG.md if needed
- File structure: CLAUDE.md = rules/reference, AUDIT_LOG.md = execution log + checklist (2 files only)

### RC17: Last-mile freshness check before send (CRITICAL)
**Symptom:** Audit started 4/7 ~11 AM. Slack message draft prepared. User asked to send at ~16:30 PM. In between, 4/7 16:08 (홍유니 v4 계약서 발송) and 4/7 16:23 (박서우 클린본 요청) arrived in Slack. The draft was sent without re-checking the most recent Slack messages, so it falsely reported R20 as "협상 교착".
**Root cause:** Treated the audit result as a static snapshot. Did not refresh between audit completion and send time.
**Prevention:**
- **Before sending ANY status update, ALWAYS run a last-mile check** of the past 30 minutes ~ 2 hours of Slack messages in BOTH channels.
- Specifically:
  1. Run `slack_read_channel` for both channels with `limit: 20` to get the latest activity
  2. For any message timestamp newer than the audit start time, deep-read the thread
  3. Check if any in-progress task in the draft has a status change in the new messages
  4. If yes, update the draft and re-verify before sending
- **The longer the gap between audit and send, the more critical this becomes.** A 5-hour gap means major status changes are likely.
- This rule applies even if user says "send now" — a 30-second freshness check is non-negotiable.
- If user is waiting, add a brief note: "발송 전 last-mile 체크 진행 중..."

### RC18: Week boundary mis-classification
**Symptom:** Today is 2026-04-07 (Tuesday). Slack draft labeled "이번주 마감 (4/8~4/9)" and "다음주 마감 (4/10~4/13)". But 4/10 (Fri) and 4/11 (Sat) are still THIS week, not next week. User had to point out the error.
**Root cause:** Did not compute the actual week boundaries based on today's date. Just split tasks arbitrarily by clusters of dates without checking what "이번주" / "다음주" mean relative to today.
**Prevention:**
- **Before grouping tasks by week, compute the explicit week boundary:**
  - Get today's day of week
  - **이번주 (this week)** = Monday of current week ~ Sunday of current week
  - **다음주 (next week)** = next Monday ~ next Sunday
- For 2026-04-07 (Tuesday): 이번주 = 4/6(월)~4/12(일), 다음주 = 4/13(월)~4/19(일)
- **Write out the boundaries explicitly** in the draft preparation step (e.g., "이번주 = 4/6~4/12, 다음주 = 4/13~4/19")
- After grouping, verify that every Due date in "이번주 마감" is between Mon-Sun of current week
- This applies to any time-bucketed grouping (이번주/다음주/이번달/다음달)

## Overdue task reporting rule

When drafting any status update (Slack message or internal summary):
1. Compute `days_overdue = today - Due` for every task with status ≠ Complete
2. If `days_overdue > 0`, the task is **overdue** and must be labeled accordingly:
   - `• 태스크명 (담당자, Due YY.MM.DD) ⚠️ N일 지연`
   - For critical overdue items (>3 days), surface them in a dedicated "⚠️ 지연 태스크" section
3. Do NOT rewrite the Due date to today or a future date. The original Due date is the audit trail.
4. Rescheduling authority:
   - ✅ Explicit Slack message from owner: "4/14로 연기하겠습니다" → update Due + log evidence
   - ✅ Human manual edit in the sheet → respect it, don't revert
   - ❌ Assumed rescheduling because "it's been a while" → NEVER do this
   - ❌ Rescheduling to avoid showing a delay → NEVER do this

## Mistake prevention workflow (before any status update)

Run this checklist before every task audit or Slack status message:

1. ✅ Did I scan BOTH channels for the full 2-month window with pagination?
2. ✅ Did I read every thread with replies via `slack_read_thread`?
3. ✅ Did I check every RC1–RC10 pattern against the current task list?
4. ✅ Did I break every multi-deliverable initiative into sub-task rows?
5. ✅ Did I validate sheet-only items against Slack evidence?
6. ✅ Did I check for client-dependency requests that need transfer tasks?
7. ✅ Did I check for convention changes that need CLAUDE.md updates?
8. ✅ Did I verify that "Complete" means verified deliverable, not announced plan?
9. ✅ Did I verify row numbers after batchUpdate by reading back a sample?
10. ✅ Did I cross-reference every task owner with actual Slack activity?

If any answer is NO, do not proceed with the status update.

## Slack status message rules

When sending task status updates to Slack:
- Always include a footer asking for corrections: "업무 현황에 수정/추가 사항이 있으시면 이 스레드에 코멘트 남겨주세요. 업무시트: [sheet link]"
- Both Slack AND the Google Sheet are sources of truth — check both, not just one
- Google Sheet items without Slack threads still need tracking and periodic verification
