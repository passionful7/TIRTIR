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
- **Klaviyo Flow R&R 분기 (2026-05-04 자연님 client 공지 ts 1777886795)**: TIRTIR이 **Retentics (AI SaaS)** 사용하여 Flow(1회 구매 → 재구매 유도, loyal 전환) 자동화. 매드업은 **Campaign 영역(뉴스레터 등) 집중**. 매드업 응답안 작성 진행 중 (~5/7)
- **SHOP NOW 버튼 컨벤션 (2026-04-28 김현서 ts 1777364820)**: 4/28 이후 캠페인부터 SHOP NOW 버튼은 **이미지 형태**로 제작. 1차 적용 시 폰트 12 + Bold (김소형 우회안)
- **레퍼런스 사진 저작권 정책 (2026-04-23 자연님 19:21)**: 외부 레퍼런스 사진(Pinterest 등) 그대로 사용 금지. 참고용으로만 활용
- **발송 일정·문구 시트 일원화 (2026-05-04 이동호 ts 1777862813)**: Klaviyo 문구·일정 모두 [통합 시트](https://docs.google.com/spreadsheets/d/1FnrZWglAeb6S2wx-zIohS_kGoGYDe31v6Y4JOzaKWkM)에서 일원 관리. 변경 발생 시 시트 반영 필수 (Glow Gift 2차 발송 일정 swap 후 5/13 인지 / 5/3 실제 발송 어긋남 사후 합의)

### Ad accounts
- Meta Ad Account: `508479813992365`
- Meta Pixel: `879708003398074`
- Google Ads CID: `842-768-6345`
- GA4 Property: `244645253`
- Google MC: `648960094`

## Promotion timing convention (updated 2026-04-14, 자연님 재확정 — SWAPPED)

> **⚠️ 중요: 2026-04-14 11:40 KST 자연님 thread (ts 1776134414.015489) 명시적 재확정 — 기존 4/9 규칙과 open/close 타임존이 swap됨.** 이유: 기존 규칙(Open PST / Close EST)은 "늦게 시작해서 일찍 끝나는" 상황 → 프로모션 기간을 최대한 길게 가져가기 위해 swap. 이동호 4/14 11:42 수락.

Promotion open/close times use **different timezones** for open vs close (per TIRTIR 자연님 4/14 재확정):
- **Open:** EST 00:00 = **KST 13:00** (Eastern Time)
- **Close:** PST 00:00 (midnight Pacific) on the day AFTER the last promo day = **KST 16:00** (Pacific Time)
- **Ad creatives OFF:** 1 hour before close (PST 23:00 previous day = KST 15:00)
- Example: 프로모 4/1~4/3 → Open EST 4/1 00:00 (KST 4/1 13:00) / Close PST 4/4 00:00 (KST 4/4 16:00)
- Note: during daylight saving (Mar–Nov), actual timezones are EDT (UTC-4) and PDT (UTC-7), but the team uses "EST"/"PST" as labels regardless. The KST conversions stay the same.
- Always express times in **EST** (open) / **PST** (close) when communicating externally, and include **KST equivalent** for internal coordination.
- When creating close/rollback tasks, set the due date/time to the PST close time converted to KST (16:00).
- **Promotion landing page redirect:** immediately at close time (no delay). Redirect to dedicated "프로모션 종료 안내" page (R145), not main homepage.

### 과거 규칙 변경 이력
- **2026-04-09 (자연님 최초 확정)**: Open PST (KST 16:00) / Close EST (KST 13:00) — "늦게 시작/일찍 종료" 문제로 폐기
- **2026-04-14 (자연님 swap 재확정)**: Open EST (KST 13:00) / Close PST (KST 16:00) — **현재 규칙**

### 기존 스케줄된 프로모션 처리 (2026-04-17 정정)
- **4/14 swap 이후 ABB도 새 규칙 적용**: ABB 종료 KST 16:00 (Close PST 00:00). 이전 문서에 "ABB KST 13:00"으로 기재되어 있던 것은 착오 — 이동호님 4/14 11:42 thread에서 "종료시간: PST 4/4 00:00 (KST 4/4 오후 4시)" 일반 규칙 명시
- **특정 프로모션 예외는 client 명시적 합의가 있을 때만 인정**: 시트 Result에 "자연님 YYYY-MM-DD 명시 확정: 기존 HH:MM 유지" 슬랙 ts 증거 필수. 증거 없이 "기존 유지"로 기재하지 말 것
- **신규 라이브/종료 시간 설정은 새 규칙 적용**: Spring Matcha Refresh 4/20 라이브, Primer Launch Deal 4/27 라이브 모두 **KST 13:00 오픈**, 종료 다음날 00:00 PST = KST 16:00
- 기존 설정과 신규 설정이 혼재할 경우 시트 Result에 "2026-04-14 rule swap 이전/이후" 명시

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
- **AO 광고 = Always On (상시 광고)** (2026-04-30 홍유니 14:01 ts 1777525302) — 향후 모든 운영 보고 동일 의미

## UTM tracking convention (updated 2026-04-22, 이동호 ts 1776858473)

> **Final URL Suffix 방식으로 모든 매체 통일**. Pmax 트랙킹 이슈 대응으로 Tracking Template 방식 폐기.

- Google Ads (Search/Display/YouTube/Pmax), Meta, TikTok 모두 **Final URL Suffix**에 UTM 파라미터 입력
- 4/22 20:48 기준 Google 캠페인의 Tracking URL 일괄 제거 완료
- Meta/TikTok UTM 적용은 홍유니 4/24 휴가 복귀 후 진행 예정
- Confluence UTM 가이드(Page ID 4701454355) 동일하게 업데이트 완료
- 이전 Tracking Template 방식 사용 캠페인 발견 시 즉시 Final URL Suffix로 전환

## 소재 기획 → 시안 컨펌 7단계 표준 프로세스 (updated 2026-04-29, 홍유니 19:06 ts 1777457201)

> 디자인 시안 컨펌은 **디자인팀 1회 sync로 종결**. 의사결정 번복/오브제 변경/요청 회수 지양.

1. **티르티르 기획안** 전달 (TIRTIR)
2. **매드업 소재/배너 기획** 작성 + 1차 컨펌 단계에서 **문구 마무리** (황아인)
3. **티르티르 1차 컨펌** (자연님 / 박서우 / 김현서)
4. **매드업 디자인 제작** (김소형)
5. **티르티르 시안 컨펌** (자연 / 박서우 / 김현서)
6. **매드업 디자인 수정 + 최종본** (김소형)
7. **티르티르 시안 2차 컨펌** (디자인팀 1회 sync로 종결)

- 시안 컨펌 단계에서 디테일업·디자인팀 싱크용 추가 수정만 허용
- 의사결정 번복·오브제 변경·요청 회수는 **금지**
- 1차 컨펌 단계에서 문구를 마무리하여 디자인 단계 진입 시 변경 최소화

## 상시 광고 운영 cadence (updated 2026-05-06, 홍유니 08:47 ts 1778024861 + 이동호 5/4 의견)

- **운영 주기**: 월 1회 (기존 "주 1회"는 운영 어려움으로 변경)
- **메타 상시소재**: 5월말까지 **3개 이미지 에셋 제작** → 6월 이미지 에셋 교체 운영
- **5월 운영**: 텍스트 에셋 변경 위주
- **6월 운영**: 이미지 에셋 신규 교체
- **BDRN 상시 광고**: 재정비 시 OFF 일괄 처리 (4/30 미팅 종료 합의 후 BDRN 라이브 잔류 발견 → 이동호 5/4 직접 종료)
- **김소형이 구글/메타 상시소재 스케줄링 + 제작 요청 받음**, 5월말까지 제작 (기획은 차주 초까지)

## 프로모션 스펙 양식 시트 사용 (updated 2026-04-23, 황아인 19:13 ts 1776939231)

- **5/15 AI 샤셋 건부터 의무 적용**
- TIRTIR이 [협업 시트](https://docs.google.com/spreadsheets/d/1FnrZWglAeb6S2wx-zIohS_kGoGYDe31v6Y4JOzaKWkM)의 스펙 양식 작성 후 노티 → 매드업 검토 → 진행
- 양식 작성 없이 진행되는 프로모션 요청은 양식 작성을 먼저 요청

## NL/UK GWP 가재고 운영 (프리오더 방식, updated 2026-04-30 박서우)

- 메세지 카드 등 GWP 재고 미입고 시 **가재고 500개씩 등록** 후 입고 시 전산 반영
- 재고 부족으로 라이브 지연 방지 — 1-2일 내 도착 가정한 프리오더

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

## Self-correction protocol (META-RULE — MANDATORY)

> **메타 규칙 (최상위 우선순위)**: 잘못을 했으면 반드시 **(1) 원인 찾고 → (2) 재발 방지책을 만들어 → (3) CLAUDE.md와 해당 skill에 즉시 기입**한다. 이 프로토콜은 모든 TIRTIR 작업에 적용되며, 단순 수정으로 끝내고 문서화를 skip하는 것은 RC16(checklist skipping) 위반과 동일하게 취급한다.

**잘못이 발견되는 즉시 반드시 아래 7단계 순서로 실행** (단계 건너뛰기 금지):

1. **Identify the error** — What was wrong? (wrong status, wrong time, wrong scope, missing task, composition error, etc.)
2. **Find the root cause** — Which judgment/process rule was violated or missing? "왜 이 실수가 발생했는가?"에 대해 1-why가 아닌 5-why 수준까지 파고들어 **메커니즘**을 찾을 것 (단순 "부주의"로 끝내지 말 것).
3. **Check for siblings** — Are there other tasks/outputs with the same type of error? Scan all non-Complete tasks, all active draft sections, or all recent outputs for the same pattern. **단일 사례에서 멈추지 말고 같은 패턴의 다른 오류를 모두 찾아낼 것.**
4. **Fix immediately** — Update the sheet, send Slack correction (thread reply, not edit — audit trail 보존), redo any affected outputs.
5. **Create/Update a Root Cause (RC) rule**:
   - 새 패턴 → **RC{N+1} 신규 추가** (번호 순차, 기존 최대 번호 + 1)
   - 기존 RC의 범위 부족 → 해당 RC의 **Prevention 섹션 강화**
   - RC는 반드시 다음 4요소 포함: **Symptom (구체 사례 + 날짜) / Root cause (메커니즘) / Prevention (실행 가능한 체크리스트) / CRITICAL 여부**
6. **Update related skill files** — `.claude/skills/{skill}/SKILL.md`에도 같은 RC를 **동일 구조로 반영**. CLAUDE.md는 팀 전체 참조 문서, skill은 실행 도구 — 둘 다 sync 필수.
7. **Never repeat** — 동일 root cause가 2번 나오면 규칙이 **너무 추상적이거나 실행 단계에 묶이지 않은 것**. RC를 더 구체화하거나 process step에 직접 embed (예: "draft 작성 직전 시트 E 컬럼 재조회").

**실패 감지 트리거** (사용자가 지적했을 때만이 아님):
- 사용자가 "~아니야?", "~놓친 것 같은데" 류의 질문 → **즉시 self-audit 모드 진입**
- 발송 후 스스로 오류 발견 → 사용자 지적 없이도 자발적 정정 + RC 생성
- Audit 중 이전 audit의 RC 위반 흔적 발견 → 누적 기록 (AUDIT_LOG)

**메타 규칙을 지키지 않은 사례는 그 자체가 RC 위반** — "고치고 넘어감"은 금지. 반드시 RC 형태로 문서화하여 미래 방지 가능한 형태로 만들어야 완료 처리.

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

**Recurrence (2026-04-22 — CRITICAL REPEAT):** 4/21 EOD 봇 메시지 thread(`ts 1776733061.250129`)에 4/21 16:16~16:22 사이 6개 reply가 달림: 황아인 "ABB Tint Trio 쇼핑백 제거 완료하였습니다" (R192) + 이동호 "레드쿠션+워터리즘 틴트 30% 할인 해당 쓰레드 답변으로 완료" (R143). 4/22 audit에서 thread_ts 추측 시 `.000000`/`.xxx` 포맷 오류로 thread_not_found만 반환 → ts 포맷 확인 안 하고 "팀 reply 없음" 결론 → R192/R143 모두 ⚠️ 지연으로 오보고. 사용자 지적 후 정정 thread reply 발송.
**Root cause:** Treated the bot message as "outbound only" — assumed no replies would come back since the bot author is myself. Failed to check threads on bot-authored parent messages. **Additional mechanism (2026-04-22): thread_ts를 추측(timestamp를 Unix time 변환 + 임의 decimal)으로 구성했더니 `thread_not_found` 에러. 그 에러를 "reply 없음"으로 오해하고 검증 종료. 정확한 thread_ts는 `slack_search_public_and_private` 또는 `slack_read_channel` 결과에서 직접 가져와야 함.**
**Prevention:**
- **Every audit cycle MUST re-read the most recent bot status update threads** in BOTH channels (`C0AMP96KZHB` and `C0ALWLLQFU7`).
- Bot messages are not "send and forget" — team members reply with status updates, corrections, and new task info.
- Search for `from:매도비` (bot user) or `from:U06AHN2ALJW` (bot user_id) in last 7 days, then `slack_read_thread` for each result.
- The thread of your own status messages is the **most likely place** to find direct human responses to specific tasks you listed.
- This rule supersedes RC3 — even self-authored threads must be re-read.
- **thread_ts는 반드시 search/channel_read 결과의 `Message_ts` 값을 그대로 사용**. 절대 시간 변환으로 추측하지 말 것. `thread_not_found` 에러가 나면 "reply 없음"이 아니라 "ts 포맷 틀림"으로 간주 → `slack_search_public_and_private` query로 정확한 ts 재확보.
- **ts 확보 경로 우선순위**:
  1. `slack_search_public_and_private` with `is:thread after:YYYY-MM-DD in:#channel` → `Message_ts` 필드 직접 파싱
  2. `slack_read_channel` 페이지네이션 결과의 timestamp 필드
  3. 사용자가 공유한 슬랙 permalink URL(`/p<ts_without_dot>`)에서 역산 (`pXXXXX` 부분을 `XXXX.XXXX` 포맷으로 변환)
- 사용자 지적 트리거 시 — "thread 놓쳤다" 지적이 오면 **즉시 원 thread_ts로 재독 + 누락된 status 전수 반영 + 정정 thread reply 발송** (사용자 대기 시간 최소화).

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

### RC20: Stale task carry-over in draft composition (CRITICAL)
**Symptom (2026-04-14):** Draft 작성 시 "이번주 마감" 섹션에 R151 번들 재고 버그 추이 데이터 공유를 포함했는데, 해당 task는 4/13 봇 EOD thread reply에서 황아인이 이미 "완료. 시트 내에도 완료로 변경하였습니다"라고 알렸고 시트 Status=Complete였음. Bot scan 단계에서 완료 선언을 읽었음에도 draft 작성 단계에서 직전 EOD의 이번주-마감 리스트를 그대로 carry-over.
**Root cause:** Draft composition 시 직전 audit의 `이번주 마감` / `다음주 마감` 섹션을 템플릿처럼 재활용하면서 각 task의 **현재 시트 Status**를 재조회하지 않음. Scan 단계에서 status 변경을 읽어도, draft 단계로 넘어올 때 전달되지 않으면 무의미.
**Prevention:**
- **Draft 작성 시 모든 열거 task의 시트 E 컬럼(Status)을 최종 재조회** 필수.
  - `이번주 마감` / `다음주 마감` / `지연 태스크` / `P0` 섹션에 포함되는 모든 task는 Complete가 아닌지 먼저 확인.
  - 간단한 Python 필터: draft 본문 작성 전에 `[row for row in sheet_values if row[4] != 'Complete']` 재실행.
- 직전 EOD 메시지의 열거를 **템플릿으로 재활용 금지** — task 리스트는 매번 시트에서 fresh load.
- Scan 단계에서 발견한 status 변경은 반드시 **시트 update → draft 작성** 2단계 분리. Scan → Draft 직행 금지.
- Draft 발송 직전 (STEP 6 Last-mile 체크에 추가): `이번주 마감/다음주 마감 섹션의 모든 task를 grep → 시트에서 Status 재확인`.

### RC19: Active-task thread re-read across BOTH channels (CRITICAL)
**Symptom (2026-04-14):** R107 중국 세션 분석 was completed by 이동호 on 4/13 10:34 KST in the **client channel** thread `1775525708.773669` (parent: 4/7 박서우 question). The reply was posted 15 minutes before the 4/13 bot EOD at 10:49, but both the 4/13 EOD AND the 4/14 follow-up audit missed it, falsely reporting the task as "6일 지연".
**Root cause:** The 2-month Slack scan correctly identified recent parent messages in both channels, but the active Processing task's source thread had a **parent older than the scan window** (4/7 박서우) with a **recent reply (4/13)**. The scan default behavior — reading recent parent messages + their threads — misses this pattern. `is:thread after:YYYY-MM-DD` search should catch it, but is fragile (search indexing lag, keyword mismatch).
**Prevention:**
- **Every audit MUST re-read every active-task source thread in BOTH channels**, identified by `ts` in the sheet's Result column — regardless of how old the parent message is or when you last read it.
  1. Parse the Work tab for every Processing / Not yet task where Result contains `ts XXXXXX.XXXXXX` or `thread_ts XXXXXX.XXXXXX`.
  2. For each unique thread_ts, call `slack_read_thread` on **both channels** (the same ts may be referenced from either side during cross-channel quoting).
  3. Compare newest reply ts against your last audit timestamp — any newer reply → deep-read the full thread.
- **Client-channel task completions are HIGH RISK** — completion messages posted by Madup team in the client channel (reporting back to client) often don't appear in internal channel. RC12 already covers bot threads in both channels; RC19 extends it to ALL active-task threads.
- If a task in the Result column has a `thread_ts` explicitly written, that thread is a **canonical truth source** and must be re-read every audit cycle.
- When a client asks a question in their channel (e.g., 박서우 3/27), the RESPONSE is often days or weeks later by a Madup owner — you must check the same thread, not wait for a new parent.
- **Do NOT rely on channel-level pagination alone**; the 2-month window catches NEW parents, but misses OLD parents with NEW replies. Combine channel scan + `is:thread after:` search + sheet-driven thread re-read for complete coverage.

### RC22: 저장된 예외 규정 재검증 누락 (CRITICAL)
**Symptom (2026-04-17):** CLAUDE.md `기존 스케줄된 프로모션 처리` 섹션에 "ABB 4/17 KST 13:00 종료는 기존 설정 유지"로 기재되어 있었음. 해당 예외는 증거 ts 없이 구두 단서만으로 작성된 것. Audit 시 이 예외 규정을 **슬랙 증거 재검증 없이** 그대로 채택하여 R73(Shopify 종료) KST 13:00, R155(광고 종료) KST 12:00으로 표기. 실제로는 4/14 swap 규칙 적용 대상 = KST 16:00 / 15:00 이어야 했음. 사용자 지적으로 발견.
**Root cause:** CLAUDE.md에 저장된 문장을 **기록된 시점의 팩트**가 아니라 **변경 불가능한 진실**로 취급. 증거(슬랙 ts)를 재조회하는 과정 없이 "이미 문서화된 것"이라는 이유로 채택. 예외 규정일수록 ts 증거 요구가 강해야 하는데 오히려 검증이 느슨해지는 모순.
**Prevention:**
- **예외 규정에는 반드시 ts 증거 필수**. CLAUDE.md에 "예외: X는 Y로 유지"라고 기재할 때 반드시 슬랙 ts 또는 시트 Result의 ts 링크 포함. ts 없으면 예외 인정 금지.
- **매 audit 시 예외 규정 증거 재검증**: CLAUDE.md에서 "기존 유지", "예외", "그대로" 같은 표현 검색 → 증거 ts가 명시되어 있는지 확인. 증거 없으면 예외 삭제 또는 재검증 → 증거 확보.
- **문서는 진실의 저장소가 아니라 관측 기록의 저장소**. 현재 슬랙/시트가 상충할 때는 슬랙/시트가 우선. 문서에 있는 예외는 "과거 audit 시점의 기록"이라는 가정 하에 재검증.
- 본 규칙은 메모리 관련 CLAUDE.md 전역 지침과도 일치: "memory (or documented exceptions) can become stale over time. Verify that the memory is still correct and up-to-date by reading the current state."

### RC21: Slack 메시지에 내부 시스템 업데이트 노출
**Symptom (2026-04-14):** 프로모션 timing swap 공지 thread reply(ts 1776134749.918949)에 "CLAUDE.md + skill(tirtir-audit) 2.1 운영 규칙 섹션 모두 업데이트 완료" 문구를 포함. 팀원 입장에서 내부 문서 업데이트는 불필요 노이즈 — 실제 행동 관련 정보(시간 변경, 시트 반영)만 원함.
**Root cause:** Slack 메시지 작성 시 "audit의 완결성"을 보여주려는 compulsion으로 내부 작업 내역(CLAUDE.md 수정, skill 업데이트, AUDIT_LOG 기록 등)까지 포함. 팀 관점이 아닌 작업자 관점의 메시지.
**Prevention:**
- **Slack 메시지에는 팀의 실제 행동·의사결정에 영향을 주는 정보만 포함**. 내부 문서/skill/AUDIT_LOG 업데이트는 **완전히 제외**.
- 금지 표현 (메시지에서 완전 배제):
  - "CLAUDE.md에 반영/업데이트 완료"
  - "skill(tirtir-audit) 섹션 업데이트"
  - "AUDIT_LOG 기록 완료"
  - "RC{N} 신규 추가 / RC{N} 강화"
  - "Audit trail 보존"
  - "META-RULE 적용"
- 허용 표현: 시트 변경 사항 (R{N} 업데이트), task 상태, 시간/일정 변경, 담당자 변경, 외부 커뮤니케이션 필요 사항.
- **Draft 작성 후 발송 전 grep**: 본문에서 `CLAUDE.md`, `skill`, `AUDIT_LOG`, `RC\d+` 패턴 검색 → 있으면 제거.
- 본 규칙은 Slack 메시지 composition rule로 격상 — 아래 "Slack status message rules" 섹션에도 반영.

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
- **내부 시스템 업데이트 관련 사항은 절대 메시지에 포함 금지 (RC21)**: 팀원은 실제 행동/의사결정에 영향을 주는 정보만 필요. 제외할 표현:
  - "CLAUDE.md에 반영 / CLAUDE.md 업데이트 완료"
  - "skill 섹션 업데이트 / tirtir-audit skill 강화"
  - "AUDIT_LOG 기록 / audit trail 보존"
  - "RC{숫자} 신규 추가 / RC{숫자} 위반 감지"
  - "META-RULE 적용 / self-correction protocol 실행"
  - 기타 내부 문서·프로세스 관련 meta 언급
- **허용 내용**: 시트 변경 사항(R{N} 업데이트), task 상태 변경, 시간/일정 변경, 담당자 변경, 외부 커뮤니케이션 필요 사항, 운영 규칙 변경 내용 그 자체
- **Draft 발송 전 grep 필수**: 본문에서 `CLAUDE.md` / `skill` / `AUDIT_LOG` / `RC\d+` / `META-RULE` 패턴 검색 → 있으면 제거 후 발송
