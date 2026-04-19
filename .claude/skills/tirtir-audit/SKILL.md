---
name: tirtir-audit
description: "TIRTIR 업무 시트 + 슬랙 전체 audit 프로세스. Slack 2개월치 전수 스캔(thread 댓글 포함) → 시트 업데이트 → 슬랙 상태 메시지 초안 생성. CLAUDE.md의 RC1~RC19 규칙과 AUDIT_LOG.md 체크리스트를 **incremental 기록 방식**으로 강제 적용. Use when user says: TIRTIR audit, update the sheet, check all tasks, slack status draft, EOD update, 업무 현황 공유, 태스크 점검."
user-invokable: true
---

# TIRTIR Full Audit & Status Update

TIRTIR US 운영의 전수 audit 프로세스. 슬랙 2개월치 스캔 → 시트 업데이트 → 슬랙 발송까지 하나의 흐름으로 실행한다. 모든 단계는 `CLAUDE.md`의 RC1~RC20 규칙을 **강제**하고, `AUDIT_LOG.md`에 실행 기록을 남긴다. **본 skill은 CLAUDE.md의 모든 운영 규칙을 자급자족적으로 내장**하므로, skill만 읽고도 audit이 완결되어야 한다.

---

## 🚨 META-RULE (최상위 우선순위): 잘못 → 원인 → 재발 방지책 → 문서화

> **잘못을 했으면 반드시**: (1) 원인 찾고 → (2) 재발 방지책 만들어 → (3) CLAUDE.md + 본 SKILL.md에 즉시 기입.

모든 실수는 다음 7단계로 처리 (단계 건너뛰기 금지 — skip 자체가 RC16 위반):

1. **Identify**: 무엇이 잘못됐는지 구체 기록 (task 이름, 증상, 날짜)
2. **Root cause**: 5-why 수준까지 메커니즘 탐색 (단순 "부주의" 금지)
3. **Siblings scan**: 같은 패턴의 다른 오류가 있는지 전수 검사
4. **Immediate fix**: 시트 수정 + Slack 정정 (thread reply, edit 금지 — audit trail 보존)
5. **Create/Update RC**: 새 패턴 → `RC{N+1}` 신규 / 기존 범위 부족 → 기존 RC 강화
   - RC 필수 4요소: Symptom(구체 사례+날짜) / Root cause(메커니즘) / Prevention(실행 체크리스트) / CRITICAL 여부
6. **Sync skill files**: CLAUDE.md + 관련 `.claude/skills/*/SKILL.md` 모두 동일 내용 반영
7. **Verify non-repeat**: 동일 root cause 재발 시 규칙이 추상적이거나 process step에 embed 안 된 것 → 더 구체화

**트리거 (사용자 지적만이 아님)**:
- 사용자가 "~아니야?", "~놓친 것 같은데" → 즉시 self-audit
- 발송 후 스스로 발견 → 자발적 정정 + RC 생성
- Audit 중 이전 RC 위반 흔적 발견 → AUDIT_LOG 누적 기록

**"고치고 넘어감" 금지** — RC 문서화까지 완료해야 실제 완료 처리.

---

## 0. 핵심 리소스

### Google Sheets (Single Source of Truth)
- **작업 시트**: `1BLQfFg7TZaRuw6CYwe6rjGgyqThkrbjzbWRYiiyhIe4`
  - 탭: `업무` (메인), `R&R`, `캠페인 보드`, `mix`, `Promotion summary`
  - 컬럼 구조: A=Tasks / B=Description / C=In-charge / D=Due / E=Status / F=Result / G=Due Time (KST)

### Slack 채널
- **고객사**: `C0AMP96KZHB` (`#매드업x티르티르us`)
- **내부**: `C0ALWLLQFU7` (`#mg_mkt_tirtir`)
- **봇 user_id**: `U06AHN2ALJW` (매도비)
- **봇 토큰**: 환경변수 `SLACK_TOKEN`에서 조회 (`os.environ["SLACK_TOKEN"]`). 로컬에서 `export SLACK_TOKEN=xoxb-...`로 설정, git에는 절대 커밋 금지
  - 용도: `chat.postMessage` 전용 (MCP Slack tool은 읽기/검색에만 사용)

### 외부 시스템
- **Shopify**: `tirtir.global` (Shopify Plus), 테마 Broadcast, PowerX(할인)/BOGOS(GWP)
- **Klaviyo**: 이메일 전용 ($4,005/mo, TIRTIR 직접 결제), SMS 비활성화
- **Meta Ad Account**: `508479813992365`, Pixel `879708003398074`
- **Google Ads CID**: `842-768-6345`, GA4 `244645253`, MC `648960094`
- **Confluence UTM doc**: Page ID `4701454355` (Space: MG)

### 관련 문서
- **규칙 정의**: [CLAUDE.md](../../../CLAUDE.md)
- **실행 로그**: [AUDIT_LOG.md](../../../AUDIT_LOG.md)

---

## 1. R&R (Source of Truth = 시트의 R&R 탭)

신규 task 배정 + 슬랙 태깅 시 **반드시 R&R 탭의 Slack Tag 사용**. 이름 평문 표기는 금지.

### Madup
| 이름 | Slack Tag | 역할 | Responsibilities |
|------|-----------|------|------------------|
| 이동호 | `<@UCMKVTWP9>` | CEO, Tech Lead | Shopify/Klaviyo/UTM/계약서/카탈로그 피드/데이터·트래킹/SEO/리포팅 |
| 홍유니 | `<@U02L20Z4MLP>` | Media Lead | 매체 이관·믹스·운영·결제, 권한, 고객사 커뮤니케이션 |
| 황아인 | `<@U09PWQEJTMY>` | Creative Planning | 소재 기획, 디자인팀 커뮤니케이션, Klaviyo 콘텐츠, 고객사 에셋 컨펌 |
| 김소형 | `<@U09FFQVH2FJ>` | Creative Production | 소재 제작(이미지/영상/상세페이지) |
| 하지현 | `<@U06ND4MHK5W>` | Media Operations | Meta/Google 캠페인 세팅 지원 |

### TIRTIR (Client)
| 이름 | Slack Tag | 역할 |
|------|-----------|------|
| 권자연 | `<@U0AMPHRRLPP>` | Marketing Manager — 프로모션 기획/컨펌 |
| 박서우 | `<@U09D7F2BVMW>` | Marketing — 신규 요청(구독제/번들/GWP) |
| 김현서 | `<@U0AMX4ACS68>` | Media/Ad Ops — 광고 결제·계정 권한 |

### Usergroup
- `<!subteam^S0ANECM32QY>` — tirtir usergroup (팀 전체 멘션)

### Task 배정 규칙
Task 성격 → owner 매칭:
- Shopify 설정/연동/카탈로그 → **이동호**
- Klaviyo/UTM/계약/리포팅/SEO → **이동호**
- Meta/Google 매체 운영·캠페인·결제 → **홍유니**
- 매체 소재 **기획**(디자인팀 커뮤니케이션) → **황아인**
- 매체 소재 **제작**(이미지/영상) → **김소형**
- Meta/Google 캠페인 세팅 지원 → **하지현**
- 여러 영역에 걸치면 **RC5에 따라 sub-task로 분리**

---

## 2. 운영 규칙 (Operational Conventions)

### 2.1 Promotion 시간대 (자연님 2026-04-14 재확정 — SWAPPED)
> **⚠️ 2026-04-14 11:40 자연님 thread (ts 1776134414.015489) 명시적 재확정 — 기존 4/9 규칙 swap.** 이유: 기존 규칙(Open PST/Close EST)이 "늦게 시작/일찍 종료" → 프로모 기간을 최대한 길게. 이동호 4/14 11:42 수락.

- **오픈**: EST 00:00 = **KST 13:00** (Eastern Time)
- **종료**: PST 00:00 (마지막 프로모 다음날 자정) = **KST 16:00** (Pacific Time)
- **광고 OFF**: 종료 1시간 전 (PST 23:00 전날 = KST 15:00)
- 예: 프로모 4/1~4/3 → 오픈 EST 4/1 00:00 (KST 4/1 13:00) / 종료 PST 4/4 00:00 (KST 4/4 16:00)
- DST 기간(3~11월)에도 "EST"/"PST" 라벨 사용 (실제는 EDT/PDT). KST 환산은 동일.
- 외부 커뮤니케이션: EST(오픈) / PST(종료) + **KST 병기 필수**
- **랜딩 페이지 redirect**: 종료 시점 즉시 (지연 없음). 전용 "프로모션 종료 안내" 페이지(R145)로 redirect — 메인 홈페이지 X.

**규칙 변경 이력**:
- 2026-04-09 (자연님 최초): Open PST (KST 16:00) / Close EST (KST 13:00) — 폐기
- 2026-04-14 (자연님 swap): Open EST (KST 13:00) / Close PST (KST 16:00) — **현행**

**기존 스케줄된 프로모션 처리 (2026-04-17 정정 — RC22 참조)**:
- **4/14 swap 이후 모든 프로모션 기본적으로 새 규칙 적용**: Close PST 00:00 = KST 16:00, 광고 OFF 1시간 전 = KST 15:00
- ABB 4/17 종료 **KST 16:00** (이전 문서의 "KST 13:00" 기재는 착오 — 이동호 4/14 11:42 thread 일반 규칙 명시)
- **특정 프로모션 예외는 client 명시적 ts 증거가 있을 때만 인정** (시트 Result에 "자연님 YYYY-MM-DD 명시 확정" + 슬랙 ts 필수). ts 없으면 예외 삭제 or 재검증
- 신규 라이브/종료는 당연히 새 규칙 적용 (Spring Matcha 4/20, Primer 4/27 모두 **KST 13:00 오픈** / **KST 16:00 종료**)

### 2.2 Promotion 3-Phase 규칙 (필수)
모든 프로모션은 3단계로 **각 단계별 task 분리**:
1. **Setup** — 소재, Shopify(페이지/콜렉션/할인), Klaviyo 뉴스레터, 배너
2. **Live** — 라이브, QA, 모니터링
3. **Close / Rollback** — 배너 OFF, 할인 코드 비활성, 프로모 페이지 hide, 광고 OFF

### 2.3 Promotion Sub-task 분해 (No Joke Sale 패턴)
모든 프로모션 Setup은 **단일 umbrella 금지**. 아래 sub-task 템플릿 적용:
- Shopify 세트/할인/GWP 세팅 — **이동호**
- Shopify 프로모션 페이지 + 메인배너/딜배너 — **이동호**
- Klaviyo 뉴스레터 세팅 및 발송 — **이동호**
- 광고 소재 문구 작성 — **황아인/김소형**
- 광고 캠페인 세팅 Meta/Google — **홍유니**
- 라이브 전 QA — **이동호**
- 라이브 + 종료/롤백 (각각 별도 row)

### 2.4 Shopify 프로모션 세팅 프로세스
1. 콜렉션 생성 (할인 단위별 분리)
2. PowerX: 할인 코드 + 제품별 할인율 (Cart qualifier: Item quantity > 0)
3. Shopify 페이지 생성 (Featured Collection로 콜렉션 연결)
4. 테마 에디터: 메인 배너 / 딜 배너 (web + mobile)
5. Klaviyo 뉴스레터 + `/discount/<code>?redirect=/pages/<landing>` URL (자동 할인)
6. QA: 웹/모바일 프리뷰, 할인 테스트, 이메일 테스트
7. 라이브
8. 종료 후 수동 롤백

### 2.5 Naming Conventions
- 상태: `Complete` / `Processing` / `Not yet` / `Urgent`
- Due: `26.03.30` (YY.MM.DD)
- BB 관련: `[BB]` prefix

---

## 3. Root Cause Registry (RC1~RC18) — 판정 근거

**매 audit마다 이 표를 순서대로 체크**. RC16이 특히 위험 (체크리스트 skip).

### RC1 — Umbrella 붕괴
- **증상**: 대규모 이니셔티브를 single row로 추적 → 구체 deliverable/의존성 누락
- **방지**: 여러 deliverable → **owner별 row 분리**. 슬랙 "1) X 2) Y 3) Z" → 각 row. No Joke Sale 패턴 적용.

### RC2 — 얕은 Slack 스캔 (첫 페이지만)
- **증상**: `slack_read_channel` 첫 페이지에서 종료 → 주요 메시지 누락
- **방지**: 2개월 전수 스캔, 양 채널, 모든 thread reply. `next_cursor` 비어있을 때까지. 활성 프로젝트 채널당 최소 5+ 페이지.

### RC3 — Thread reply 무시
- **증상**: Parent만 읽고 종료. Reply #20+에 종료 시간, 규칙 변경, 액션 아이템 누락.
- **방지**: 
  - 모든 thread(≥1 reply) → `slack_read_thread`
  - 20+ reply는 **마지막 3개 reply 중점** (결정은 뒤에)
  - 기존 추적 thread도 **매 audit 재독** (새 reply 가능성)
  - 특히 **Processing task thread는 매번 무조건 재독**

### RC4 — Plan vs Done 혼동
- **증상**: "~예정", "~계획", "~하겠습니다"를 Complete로 처리
- **방지**: Complete = **검증된 deliverable 존재**. 계획·발표·선언은 Processing/Not yet.

### RC5 — Scope mixing (team 경계 무시)
- **증상**: Shopify 롤백(이동호) + 광고 OFF(홍유니)을 하나의 task로 묶음
- **방지**: 1 task = **1 owner + 1 deliverable + 1 timeline**. Shopify ops ≠ Media ops ≠ Creative ops.

### RC6 — 시트-only task를 근거 없이 urgent로 surface
- **증상**: 슬랙 discussion 없는 시트 항목을 마감 임박으로 표시
- **방지**: 슬랙 근거 없으면 `시트 기입 항목 — 슬랙 근거 미확인` 플래그. Hard deadline 금지.

### RC7 — 클라이언트-종속 요청 chain 누락
- **증상**: 내부 메시지 "X를 해주시면..." → TIRTIR 액션 필요하지만 task化되지 않음
- **방지**: 내부 메시지에 implicit client action 있으면 **"TIRTIR에 X 요청 전달" task 생성** (owner = 내부 communicator)

### RC8 — 운영 규칙 변경 누락
- **증상**: "기준", "변경", "앞으로", "이후부터", "통일" 언급된 규칙 변경이 CLAUDE.md 미반영
- **방지**: 키워드 발견 즉시 **CLAUDE.md에 반영**. Task가 아닌 문서 업데이트.

### RC9 — Row 번호 mismatch
- **증상**: batchUpdate가 row N+1에 기록 (배열 index ≠ sheet row)
- **방지**: 
  - Header row 오프셋 주의 (`values[i]` = sheet row `i+1`)
  - batchUpdate 후 **반드시 2~3 row read-back 검증**
  - `'업무'!A{N}:G{N}` 명시적 range 사용

### RC10 — Cursor 누락
- **증상**: `slack_read_channel` 재호출 시 `next_cursor` 미전달 → 같은 첫 페이지 반환
- **방지**: 매 호출 후 `pagination_info.next_cursor` 확인. 있으면 반드시 전달.

### RC11 — 지연 태스크 silent reschedule
- **증상**: Due 지남 → 말없이 Due 미래로 이동 (팀에 지연 숨김)
- **방지**: 
  - Due 변경은 **(a) 담당자 슬랙 명시 재설정, 또는 (b) 수동 시트 편집**만 허용
  - 지연 task는 `⚠️ N일 지연` 라벨 필수
  - Due 변경 시 Result에 슬랙 ts 증거 기록 (예: "이동호 4/6 재설정: 4/14 ts 1775...")

### RC12 — 봇 메시지 thread reply 무시 (CRITICAL)
- **증상**: EOD 봇 메시지 → 팀이 thread에 상태 업데이트 reply → 무시됨
- **방지**:
  - 매 audit 최근 봇 메시지 thread 전부 재독 (양 채널)
  - 검색: `from:U06AHN2ALJW` 최근 7일
  - 봇 메시지는 "send and forget" 아님 — 가장 유력한 status update 장소

### RC13 — 권한 요청 우회 처리 놓침
- **증상**: "Shopify 메뉴 권한" 요청 → 실제론 이동호 계정으로 우회 처리됨 → task는 "Not yet"
- **방지**: 
  - 권한 요청 task는 1-2일 후 재확인
  - 우회 사용 시 **"우회 처리됨 — N의 계정" 로 Complete**
  - Result 컬럼에 "권한 요청 vs. 우회 처리 결과 추적 필요" 명시

### RC14 — 전달/발송 task가 Processing에 고착
- **증상**: 실제 발송 완료됐지만 "후속 예정" 문구 때문에 Processing
- **방지**: 
  - task name "전달/발송/공유" → 아티팩트 발송 = Complete
  - 후속 iteration은 **별도 task**
  - Result "~예정" 나오면 주 deliverable 완료 여부 재확인

### RC15 — 부분 완료 umbrella 고착
- **증상**: 5개 sub 중 4개 완료, 1개 blocked → 전체 Processing 유지
- **방지**: ≥70% 완료 + 잔여가 **별개 의존성** → umbrella **Complete + 잔여 신규 row**
- 예: 말차 종료 → 배너/할인 OFF(Complete) + 랜딩 정책 적용(신규 row)

### RC16 — Checklist 건너뛰기 (laziness)
- **증상**: 규칙을 "기억"으로 대체 → 동일 실수 반복
- **방지**: **AUDIT_LOG.md에 매 audit 새 열 추가** → 한 행씩 실제 실행. Result 컬럼 StaleDefault로 간주, 재검증.

### RC17 — Last-mile freshness check 누락 (CRITICAL)
- **증상**: Audit~발송 간 시간차 동안 새 슬랙 메시지 → draft 반영 안 됨
- **방지**: 발송 직전 30초~2분 체크 필수:
  1. 양 채널 최근 20건 재조회
  2. Audit 시작 이후 timestamp 메시지 모두 확인
  3. 새 thread reply → deep-read
  4. Draft 영향 있으면 → 수정 후 발송

### RC18 — Week boundary 오분류
- **증상**: 이번주/다음주 임의 분류 ("이번주 = 이틀 안"처럼)
- **방지**: 
  - **Monday ~ Sunday 기준**으로 명시 계산
  - Draft에 "이번주 = 4/13(월)~4/19(일)" **날짜 범위 명시 기재**
  - 모든 Due가 해당 범위에 들어가는지 post-check

### RC21 — 슬랙 메시지에 내부 시스템 업데이트 노출 금지
- **증상 (2026-04-14)**: 프로모션 timing swap 공지 thread reply에 "CLAUDE.md + skill 섹션 모두 업데이트 완료" 문구 포함 → 팀원 입장에서 내부 문서 업데이트는 노이즈. 실제 행동 정보만 원함.
- **원인**: "audit 완결성" 보여주려는 작업자 관점 compulsion. 팀 관점이 아닌 self-centric 작성.
- **방지**:
  - 금지 표현 완전 배제: `CLAUDE.md 반영`, `skill 업데이트`, `AUDIT_LOG 기록`, `RC{N} 추가/위반`, `META-RULE 적용`, `audit trail 보존`, `self-correction protocol` 등
  - 허용: 시트 변경(R{N}), task 상태·시간·담당자 변경, 외부 커뮤니케이션 필요 사항, 운영 규칙 내용 자체
  - Draft 발송 전 **grep 필수**: `grep -E "CLAUDE.md|skill|AUDIT_LOG|RC\d+|META-RULE" draft.txt` → 매치되면 제거
  - Rule of thumb: "팀원이 이걸 읽고 내일 무엇을 할지 결정하는 데 필요한가?" — NO면 제외

### RC20 — Draft composition 시 시트 Status 재조회 (CRITICAL)
- **증상 (2026-04-14)**: Draft "이번주 마감"에 R151 번들 재고 버그 추이 데이터 공유 포함 → 실제로는 황아인이 4/13 봇 EOD thread reply에서 이미 "완료" 선언 + 시트 Complete 처리. Scan에서 완료 메시지를 읽었음에도 draft에 반영 안 됨.
- **원인**: 직전 EOD의 `이번주 마감` 섹션을 템플릿처럼 재활용 → 각 task 현재 Status 재조회 skip. Scan → Draft 단계 전환 시 status 변경 전달 실패.
- **방지**:
  - Draft 작성 직전 **모든 열거 task의 시트 E 컬럼 재조회**. 간단 필터:
    ```python
    # draft 본문 작성 전 반드시 실행
    active = [r for r in sheet_values if r[4] != 'Complete']
    # 모든 section 구성은 `active`에서만 pick
    ```
  - 직전 EOD 메시지를 **템플릿 재활용 금지**. 매번 시트에서 fresh load.
  - Scan → 시트 update → Draft 작성: **3단계 분리**. Scan → Draft 직행 금지 (status 변경 증발).
  - Last-mile 체크(STEP 6)에 추가: `draft 내 모든 task 이름 → 시트 Status 재확인`

### RC19 — Active-task thread 양 채널 재독 (CRITICAL)
- **증상 (2026-04-14)**: R107 중국 세션 분석이 이동호 4/13 10:34 KST에 **고객사 채널** thread `1775525708.773669` (parent: 4/7 박서우 질문)에서 완료 공유되었으나, 4/13 봇 EOD 15분 전이었음에도 4/13/4/14 audit 모두 놓침 → "6일 지연"으로 오보고
- **원인**: 2개월 스캔은 최근 parent 메시지 + thread 읽기로 동작하지만 이 패턴은 **old parent + new reply**. 특히 고객사 채널 오래된 질문에 대한 답변이 Madup 팀에서 올라올 때 놓칠 확률 최고.
- **방지**:
  - **매 audit마다 시트 Result 컬럼의 모든 `ts` 참조 thread를 양 채널에서 재독**:
    1. `업무` 탭의 모든 Processing/Not yet task parse → Result에 `ts XXXXXX.XXXXXX` 또는 `thread_ts XXXXXX.XXXXXX` 있는 항목 추출
    2. 각 unique thread_ts → `slack_read_thread` 양 채널 모두 호출 (cross-channel quote 가능)
    3. 마지막 reply ts ≥ 직전 audit ts이면 → 전체 thread deep-read
  - **고객사 채널 task 완료 메시지 HIGH RISK** — Madup 팀이 client에게 보고하는 답변은 old parent 아래 late reply로 붙음. 내부 채널에는 재공유되지 않는 경우 많음.
  - **채널 페이지네이션만으로 불충분** — 2개월 윈도우는 new parent만 캐치. Combined scan: 채널 페이지네이션 + `is:thread after:` 검색 + **시트-driven thread 재독** 3중 커버.
  - Result에 `thread_ts` 명시된 task → **canonical truth source**로 취급, 매 audit 재독 필수

### RC22 — 저장된 예외 규정 재검증 누락 (CRITICAL)
- **증상 (2026-04-17)**: CLAUDE.md `기존 스케줄된 프로모션 처리`에 "ABB 4/17 KST 13:00 종료 기존 유지" 예외가 ts 증거 없이 기재되어 있었음 → audit에서 그대로 채택 → R73 13:00, R155 12:00으로 표기. 실제 4/14 swap 규칙 적용 대상 = 16:00 / 15:00 이어야 했음. 사용자 지적으로 발견.
- **원인**: 문서에 저장된 문장을 **기록된 시점의 팩트**가 아닌 **변경 불가능한 진실**로 취급. 증거(슬랙 ts)를 재조회하는 과정 없이 "이미 문서화된 것"이라는 이유로 채택. 예외일수록 증거 요구가 강해야 하는데 오히려 검증이 느슨해지는 모순.
- **방지**:
  - 예외 규정에는 **반드시 ts 증거** 포함 (슬랙 ts 또는 시트 Result 링크). 증거 없으면 예외 인정 금지
  - 매 audit 시 CLAUDE.md에서 `기존 유지` / `예외` / `그대로` 키워드 검색 → 증거 ts 명시 여부 확인 → 없으면 재검증 또는 제거
  - 문서는 진실의 저장소가 아닌 관측 기록의 저장소 — 슬랙/시트와 상충 시 슬랙/시트 우선
  - 예외일수록 증거 요구 강화 (일반 규칙 → 예외는 2단계 검증: 1단계 근거 ts + 2단계 최근 슬랙에서 번복 없음 확인)
  - 프로모션 시간대 처럼 규칙이 swap/변경된 적이 있으면, 기존 합의 증거를 **변경 시점 이후**로 한정해서 검증해야 함 (변경 이전 합의는 자동 무효)

---

## 4. 프로세스 (절대 순서대로)

### STEP 0 — Audit 시작 전 준비

1. **AUDIT_LOG.md 새 열 추가**: 헤더 `YYYY-MM-DD HH:MM` (KST)
   - **⚠️ 중요**: 열 추가만 해두고 **각 행은 STEP 진행 중에 즉시 채움** (incremental). 한 번에 batch 채우지 말 것 — 실제 실행 시점의 사실을 기록해야 정확함.
   - 예: STEP 1에서 채널 페이지네이션 완료 → 즉시 행 1.1 ✅ HH:MM 기록. STEP 3에서 Plan vs Done 검증 완료 → 즉시 행 5.1 기록.
2. **오늘 날짜 기준 주간 경계 명시** (RC18):
   - 이번주 = 월요일 ~ 일요일 (today 포함)
   - 다음주 = 다음 월요일 ~ 일요일
   - 예: today 2026-04-14(화) → 이번주 4/13(월)~4/19(일) / 다음주 4/20(월)~4/26(일)
3. **직전 audit 시간 격차** 계산 (격차 클수록 Last-mile 강도 ↑)
4. **RC1~RC19 + Root cause registry 재독** (RC16: 기억에 의존 금지)

### STEP 1 — Slack 전수 스캔 (RC2, RC3, RC10, RC12)

**Default scope = 최근 2개월, 양 채널, 모든 thread reply 포함.**

토큰 제한 시 **병렬 sub-agent** 활용 (채널별/날짜별 split).

1. **채널 페이지네이션**:
   - `slack_read_channel(channel_id, limit=100)` 각 채널
   - `pagination_info.next_cursor` 확인 → 있으면 `cursor=...` 재호출
   - 2개월 경계 도달 or `next_cursor` 비어있을 때까지 (최소 5+ 페이지 예상)

2. **Thread deep-read** (RC3):
   - `Thread: N replies` 있는 모든 parent → `slack_read_thread(channel_id, message_ts)`
   - 이전에 읽었어도 **매번 재독**
   - 20+ replies → 마지막 3개 reply 중점

3. **Old-parent + new-reply 탐지**:
   - `slack_search_public_and_private`:
     - `is:thread after:YYYY-MM-DD in:#매드업x티르티르us`
     - `is:thread after:YYYY-MM-DD in:#mg_mkt_tirtir`
   - 반환된 모든 `thread_ts` → `slack_read_thread` 전수

4. **봇 메시지 thread 재독** (RC12):
   - `slack_search_public_and_private("from:<@U06AHN2ALJW> after:YYYY-MM-DD")`
   - 모든 결과 → `slack_read_thread` (팀 status reply 검출 최고 확률)

5. **Active task thread 양 채널 재독 (RC19 — CRITICAL)**:
   - 시트 `업무` 탭 Parse → Processing/Not yet task 중 Result에 `ts XXXXXX.XXXXXX` 또는 `thread_ts XXXXXX.XXXXXX` 있는 항목 **모두 추출**
   - 각 unique thread_ts → `slack_read_thread` 호출 시 **양 채널 모두 시도** (cross-channel quote로 같은 ts가 다른 채널에서 참조될 수 있음)
   - 마지막 reply ts ≥ 직전 audit ts → **전체 thread deep-read**
   - **Old parent + New reply 패턴 특히 주의**: 2개월 전 질문에 오늘 답변 올라오는 케이스. 채널 페이지네이션으로는 catch 불가.
   - Client 채널은 특히 risk high — Madup 팀이 client 질문에 답할 때 **internal 채널에 재공유되지 않는 경우 다수**. R107 중국 세션 분석이 대표 사례(2026-04-14 audit).
   - **시트 Result의 `ts`는 canonical truth source** — 매 audit 무조건 재독.

**신규 task 감지 시 watch**:
- 예정된 마감 (e.g., "4/5 PST 오후 11시에 종료 예정")
- 사람 배정 (e.g., "동호님 크칙요청 예정")
- 고객사 채널 요청 (implicit 신규 작업)
- 슬랙 reminder — 미래 액션 signal
- **운영 규칙 변경 키워드**: `기준`/`변경`/`앞으로`/`이후부터`/`통일` (RC8)

**금지 사항**:
- ❌ 첫 페이지만 읽고 종료
- ❌ Parent만 읽고 thread skip
- ❌ "이전에 읽었으니 skip"
- ❌ 1~2주 윈도우로 축소
- ❌ 토큰 아끼려 메시지 생략

### STEP 2 — 시트 현재 상태 로드

1. **탭 이름 = `업무`** (not "Work" — 실제 한글 탭)
2. Google Sheets API:
   ```bash
   gws sheets spreadsheets values get \
     --params '{"spreadsheetId":"1BLQfFg7TZaRuw6CYwe6rjGgyqThkrbjzbWRYiiyhIe4","range":"업무!A1:G200"}' \
     --format json
   ```
3. 각 row 파싱: `row_number`, `Tasks`, `Description`, `In-charge`, `Due`, `Status`, `Result`, `Due Time (KST)`
4. **R&R 탭 읽기**: `업무` 아닌 `R&R` 탭에서 Slack Tag 매핑 확인 (신규 담당자 있을 수 있음)

### STEP 3 — Cross-check & 판정 (RC4~RC15)

AUDIT_LOG 행 **순서대로** 실행 (skip 금지):

| # | 체크 항목 | RC |
|---|-----------|-----|
| 3.1 | 모든 Processing task의 슬랙 thread 재독 | RC3 |
| 3.2 | 모든 Not yet 활성 task의 슬랙 thread 재독 | RC3 |
| 5.1 | Plan vs Done 검증 (모든 Processing) | RC4 |
| 5.2 | Scope mixing 검증 (Shopify/Media/Creative 분리) | RC5 |
| 5.3 | 권한 요청 task → 우회 처리 체크 | RC13 |
| 5.4 | 전달/발송 task → 실제 발송 여부 확인 | RC14 |
| 5.5 | Umbrella task → 70%+ 완료 + 잔여 분리 검토 | RC15 |
| 6.1 | 시트-only item → 슬랙 근거 확인 | RC6 |
| 7.1 | 내부 메시지 → 클라이언트 종속 요청 감지 | RC7 |
| 8.1 | 운영 규칙 변경 키워드 검색 → CLAUDE.md 업데이트 | RC8 |
| 9.1 | 지연 task → `N일 지연` 표기 (Due 임의 변경 금지) | RC11 |
| 10.1 | 신규 다중 deliverable 이니셔티브 → sub-task 분해 | RC1 |

**판정 단축 가이드**:

| 상황 | 판정 | 근거 RC |
|------|------|--------|
| "~예정", "~하겠습니다" | **Processing** | RC4 |
| Due 지남 + 담당자 "완료/확정/종료처리" | **Complete** (ts 증거 기록) | — |
| 전달/발송 task + 실제 메시지 발송 확인 | **Complete** (후속 iterate는 별도 task) | RC14 |
| 권한 요청 + 우회 경로 사용됨 | **Complete** ("우회 처리됨 — N의 계정") | RC13 |
| Umbrella 70%+ 완료 + 잔여 별개 의존성 | **본 task Complete + 잔여 신규 row** | RC15 |
| 시트만 있고 슬랙 근거 없음 | `시트 기입 항목 — 슬랙 근거 미확인` 플래그 | RC6 |
| Due 지남 + 담당자 언급 없음 | **지연** `⚠️ N일 지연` (Due 유지) | RC11 |

**Task prioritization (우선순위 판단)**:
1. **프로모션 launch 데드라인 최우선** (3일 이내 라이브 = P0)
2. **블로킹 의존성 식별** (계약/권한/믹스/소재)
3. **외부 딜리버리 지연 → 능동 follow-up** (BB, TIRTIR)
4. **매출 영향 > 운영 개선**
5. **시간 버킷**: P0(지금) / P1(이번주) / P2(다음 2주) / P3(백로그)

### STEP 4 — 시트 업데이트 (RC9)

1. **업데이트 대상 수집**:
   - Status 변경 (→ Complete, → Processing, → Not yet)
   - Result 업데이트 (ts 증거 포함)
   - 신규 task append (row 이어서)
   - Umbrella 분할 시 신규 row + "RXX에서 분리" 표기

2. **Row 번호 매핑 (RC9)**:
   - Header offset 주의: `values[0]` = row 1 (header), `values[i]` = sheet row `i+1`
   - 신규 append: 현재 마지막 row + 1부터 순차
   - batchUpdate 실행:
     ```bash
     gws sheets spreadsheets values batchUpdate \
       --params '{"spreadsheetId":"..."}' \
       --json '{"valueInputOption":"USER_ENTERED","data":[{"range":"업무!A87:G87","values":[[...]]},...]}'
     ```
   - **batchUpdate 후 필수**: 2~3개 row `values get`으로 read-back 검증

3. **신규 task 필드**:
   - **A Tasks**: 한 줄 요약 (owner 관점, 단일 deliverable, promo면 "[Promo명] - [deliverable]")
   - **B Description**: 배경 + 참조 링크
   - **C In-charge**: R&R 탭 Responsibilities 매칭 후 이름 (평문, tag X — 시트 내부용)
   - **D Due**: `YY.MM.DD` (슬랙 증거 기반)
   - **E Status**: Not yet / Processing / Complete / Urgent
   - **F Result**: `[담당자] [날짜 HH:MM] [요약] (ts XXXXXXX.XXXXXX)`
   - **G Due Time (KST)**: `HH:MM` (라이브 시간 등 명시)

4. **프로모션 신규 시**: No Joke Sale 패턴으로 **9개 sub-task** 자동 생성:
   - {Promo명} - 딜 페이지 + 클라비요 기획 (황아인)
   - {Promo명} - 딜 페이지 + 클라비요 디자인 제작 (김소형)
   - {Promo명} - Shopify 세트/할인/GWP 세팅 (이동호)
   - {Promo명} - Shopify 페이지 + 배너 적용 (이동호)
   - {Promo명} - Klaviyo 뉴스레터 세팅 및 발송 (이동호)
   - {Promo명} - 광고 소재 문구 작성 (황아인/김소형)
   - {Promo명} - 광고 캠페인 세팅 Meta/Google (홍유니)
   - {Promo명} - 라이브 전 QA (이동호)
   - {Promo명} 라이브 (이동호/홍유니, PST 00:00 = KST 16:00)
   - (+ 종료 task: Shopify 롤백(이동호) + 광고 OFF(홍유니) 2개 분리 — RC5)

### STEP 5 — 슬랙 초안 작성 (확장 템플릿)

```
*[M/D 업무 현황 업데이트]* <!subteam^S0ANECM32QY>
_{직전 감사 이후 변경사항 종합. 신규 N건, 업데이트 M건, Complete K건.}_

:tada: *{이전 audit 이후} 완료* (최근 1~2 cycle 분만)
• *{task}* (<@UXXX> M/D) — {근거 요약, ts XXXXX}

:new: *신규 {프로모션/이니셔티브}* — {상세 맥락}
{배경 문단: 슬랙 소스 ts, 주체, 데드라인, 스코프 명확화}

• *{sub-task 1}* (<@UXXX>, M/D) — {비고}
• *{sub-task 2}* (<@UYYY>, M/D) — {비고}
... (RC1: 모든 deliverable sub-task로 분해)

:arrows_counterclockwise: *업데이트* (Status/Owner/Due 변경)
• *{task}* (R{N}, <@UXXX>, Due {before} → {after}) — {명시적 재설정 ts 증거}

:bell: *운영 규칙 변경* (RC8) — {있으면}
• {규칙 변경 내용} ({확정자} M/D 확정) → CLAUDE.md 반영 완료

---

:red_circle: *P0 — {임박 프로모션} {라이브 날짜} (D-N)*
• {task} (<@UXXX>, M/D)
... (promo launch 기준 모든 sub-task)
• *{M/D 라이브 PST 00:00 (KST 16:00)}* (<@UXXX> <@UYYY>)

:large_orange_circle: *이번주 마감 ({M/D(월)}~{M/D(일)})*
• {task} (<@UXXX>, M/D{ today 면 "today" + ⚠️ D-day })
...

:date: *다음주 ({M/D(월)}~{M/D(일)})*
• {task} (<@UXXX>, M/D)
...

:warning: *지연 태스크*
• *{task}* (<@UXXX>, Due {original}) ⚠️ N일 지연 — {블로커/상태}
... (Due 임의 변경 금지, N일 지연 표기)

:clipboard: 업무 현황에 수정/추가 사항이 있으시면 이 스레드에 코멘트 남겨주세요.
업무시트: <https://docs.google.com/spreadsheets/d/1BLQfFg7TZaRuw6CYwe6rjGgyqThkrbjzbWRYiiyhIe4/edit>
```

**작성 규칙 (필수)**:
- 모든 사람 이름 → **R&R Slack Tag** (`<@Uxxx>`) 
- 주간 경계 → **명시적 날짜 범위**
- 지연 → `⚠️ N일 지연` 절대 숨기지 말 것
- 프로모션 → **3-phase(Setup/Live/Close) + 9-sub-task(No Joke Sale 패턴)** 전개
- Due 라벨 `today` → `⚠️ D-day` 병기
- P0/이번주/다음주/지연은 **각각 별도 섹션**, 중복 금지
- 완료 섹션은 최근 1~2 cycle만 (장기 완료 나열 X)
- 시간은 `EST 00:00 (KST 13:00)` / `PST 00:00 (KST 16:00)` 병기 (4/14 swap 규칙)

**⛔ 메시지 포함 금지 (RC21)**: 내부 시스템 업데이트 관련 사항은 슬랙 메시지에 **절대 포함하지 말 것**. 팀은 실제 행동·의사결정 정보만 필요.
- 금지 표현: `CLAUDE.md 반영/업데이트`, `skill 섹션 강화`, `AUDIT_LOG 기록`, `RC{숫자} 추가/위반`, `META-RULE 적용`, `self-correction protocol 실행`, 기타 내부 문서 언급
- 허용 내용: 시트 변경(R{N}), task 상태·시간·담당자 변경, 외부 커뮤니케이션 필요 사항, 운영 규칙 내용 자체
- **발송 전 grep 필수**: 본문에서 `CLAUDE.md` / `skill` / `AUDIT_LOG` / `RC\d+` / `META-RULE` 패턴 검색 → 제거 후 발송

**예시 output (실제 4/14 audit 결과)**:
```
*[4/14 업무 현황 업데이트]* <!subteam^S0ANECM32QY>
_4/13 EOD 이후 변경사항 종합. 신규 11건 추가(Primer Launch Deal 9건 + Pmax + Klaviyo Notion), 업데이트 3건._

:new: *신규 프로모션 — Primer Launch Deal 4/27 (D-13)*
자연님 4/13 15:56 기획안 전달. 메인배너=고객사 제작, 매드업=딜페이지+클라비요 5건+셋팅. 고객사 데드라인 4/23, 딜 배너 전달 예정 4/17-18.

• *딜 페이지 + 클라비요 5건 기획* (<@U09PWQEJTMY>, 4/17) — 김소형에 제작 요청 진행
• *딜 페이지 + 클라비요 5건 디자인 제작* (<@U09FFQVH2FJ>, 4/22)
• *Shopify 세트/할인/GWP 세팅* (<@UCMKVTWP9>, 4/25)
... (9개 sub-task)

:arrows_counterclockwise: *업데이트*
• *레버 대시보드* (R87, <@UCMKVTWP9>, Due 4/9 → 4/15) — 이동호 4/13 21:07 초안 생성 완료, 금주 수요일 TIRTIR 전달 예정 (명시적 재설정)
...

:red_circle: *P0 — Spring Matcha Refresh 4/20 (D-6)*
...

:large_orange_circle: *이번주 마감 (4/14~4/19)*
...

:warning: *주요 지연 태스크*
• 계약서 서명 (Both, Due 3/27) ⚠️ 18일 지연 — 모두싸인 전자서명 진행 중
...
```

### STEP 6 — LAST-MILE 체크 (RC17 + RC20) — **절대 생략 금지**

발송 직전 30초~2분. 격차가 클수록 필수.

1. **양 채널 최신 20건 재조회** (`slack_read_channel limit=20`)
2. Audit 시작 timestamp 이후 메시지 모두 확인
3. 새 thread reply 있으면 → deep-read
4. Draft 영향 있는 변경 있으면 → **draft 수정**
5. **Draft 내 모든 열거 task의 시트 Status 재조회** (RC20):
   - `이번주 마감` / `다음주 마감` / `지연 태스크` / `P0` 섹션의 모든 task 이름/row 번호 추출
   - 시트 E 컬럼에서 Complete된 task 있는지 확인 → 있으면 draft에서 제거
   - 직전 EOD의 리스트를 템플릿으로 재활용한 경우 특히 주의
6. AUDIT_LOG L.1~L.5 행 체크:
   - L.1: 30분~2시간 cross-check
   - L.2: 신규 thread deep-read
   - L.3: 변경사항 draft 반영
   - L.4: 주간 분류 재검증 (RC18)
   - L.5: Draft 내 task Status 재조회 (RC20)
7. 30초 freshness 체크는 필수 (사용자 추가 확인 없이도 자동 실행)

### STEP 7 — 슬랙 자동 발송 (확인 없이)

**⚠️ MCP Slack tool로 chat.postMessage 호출 금지. 봇 토큰 API 직접 호출.** (CLAUDE.md 규칙)

**Python + urllib 권장 (안정적)**:
```python
import json, os, urllib.request
token = os.environ["SLACK_TOKEN"]  # 사전에 `export SLACK_TOKEN=xoxb-...` 필요
with open('/tmp/tirtir_draft.txt', 'r', encoding='utf-8') as f:
    text = f.read()
payload = {
    "channel": "C0ALWLLQFU7",  # 또는 C0AMP96KZHB
    "text": text,
    "unfurl_links": False,
    "unfurl_media": False,
}
req = urllib.request.Request(
    "https://slack.com/api/chat.postMessage",
    data=json.dumps(payload).encode("utf-8"),
    headers={
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json; charset=utf-8",
    },
    method="POST",
)
with urllib.request.urlopen(req) as resp:
    body = json.loads(resp.read().decode("utf-8"))
print("ok:", body.get("ok"), "ts:", body.get("ts"))
```

- **기본 채널 (확인 없이 자동 발송)**:
  - **EOD 업무 현황 / audit 결과 메시지 → `C0ALWLLQFU7` (내부) 자동 발송**. 확인 요청 금지.
  - 고객사 직접 공유가 명시적으로 필요한 메시지 → `C0AMP96KZHB` (고객사)
- **사용자 확인 없이 발송 원칙**: audit 결과 slack 메시지는 `/tirtir-audit` 호출 자체가 발송 승인. 발송 채널/시점 추가 확인 요청 금지.
  - 예외: 사용자가 명시적으로 "draft만 보여줘" 또는 채널을 지정한 경우만 그에 따름
  - STEP 6 last-mile + RC21 grep 통과 → STEP 7 즉시 발송 (중간 사용자 확인 금지)
- 발송 후 `ok: true` 확인, `ts` 기록 → AUDIT_LOG에 기록
- **발송 직전 grep 필수 (RC21)**: draft 본문에서 아래 패턴 매치되면 제거 후 발송
  ```bash
  grep -E "CLAUDE\.md|skill|AUDIT_LOG|RC[0-9]+|META-RULE|self-correction" /tmp/tirtir_draft.txt
  ```
  매치 0건 확인되어야 발송 가능. 내부 시스템 용어가 메시지에 새는 순간 팀원 신뢰 하락.

### STEP 8 — Audit 마감

1. **AUDIT_LOG 새 열 모든 행 ✅ 확인** — incremental 기록이 제대로 쌓였는지 점검
   - **원칙**: 각 단계 완료 즉시 해당 행에 `✅ HH:MM (노트)` 기록 (STEP 0 때 세팅한 방식)
   - batch 기록 금지 — 사후 정리 시 정확도 떨어짐 (RC16)
2. **진행 노트 추가** (`## 진행 노트`):
   - 시작/종료 시각
   - 트리거 (사용자 요청 내용)
   - 주간 경계
   - 신규 발견 (Complete N건 / Processing N건 / 신규 N건 / 규칙 변경)
   - RC 위반 건수 (있으면 상세 기록)
   - 발송 ts (봇 API 응답의 `ts`)
3. **RC 위반 있으면** → 해당 RC 규칙을 CLAUDE.md에서 **강화** (동일 원인 재발 방지)
4. **새 패턴 발견 시** → 새 RC 추가 (RC20, RC21, ...) + 본 SKILL.md 섹션 3에도 반영
5. **사용자 지적으로 정정 발송한 경우**: 원 메시지에 **thread reply**로 정정 (삭제/수정 금지 — audit trail 보존)

---

## 5. 상태 판정 세부 규칙 (빠른 참조)

### Complete / Processing / Not yet 구분

1. **Complete = 검증된 deliverable 존재**. "~예정", "~계획", "~하겠습니다"는 Processing/Not yet.
   - ❌ "레버 엑스퍼트 연동 계획 수립" → Complete
   - ✅ "레버 엑스퍼트 연동 계획 수립" → Processing
   - ✅ "레버 엑스퍼트 대시보드 구축 완료, 리포트 발송 시작" → Complete

2. **Due 지남 + 담당자 명시적 "완료/확정/종료처리" → Complete** (ts 증거 기록)

3. **Shopify ops vs Media ops 절대 분리** (RC5)
   - Shopify 롤백(이동호) + 광고 OFF(홍유니) = 2개 row

4. **시트-only 항목 슬랙 validation** (RC6)
   - 슬랙 근거 없으면 `시트 기입 항목 — 슬랙 근거 미확인` 플래그

5. **중복 감지**: 신규 task 생성 전 기존 task scan → 같은 scope이면 consolidate

### Processing task 소유자 체크

매 Processing은 "**누가 지금 이걸 하고 있는가?**"에 답 가능해야 함:
- "아무도 없음" → 실제론 Not yet 또는 blocked
- "외부 대기" → Result에 누구/언제부터 명시
- "assigned but activity 없음" → stale flag + 슬랙 재확인

---

## 6. 금지 사항 (Hard Rules)

- ❌ 첫 페이지만 읽고 audit 종료
- ❌ Parent 메시지만 보고 thread reply skip (특히 활성 Processing task)
- ❌ Due date 임의 변경 (슬랙 증거 없이 silent reschedule)
- ❌ 한 promotion을 한 row로 묶기 (Setup/Live/Close 3-phase + sub-task 분리 필수)
- ❌ 여러 owner 태스크를 한 row로 묶기 (RC5 scope mixing)
- ❌ 사람 이름을 평문으로만 표기 (슬랙 메시지 R&R 태그 필수)
- ❌ last-mile 체크 생략 후 발송
- ❌ MCP Slack tool로 `chat.postMessage` 호출 (봇 토큰 API 사용)
- ❌ AUDIT_LOG 작성 생략 (RC16)
- ❌ 규칙 내용을 "기억"으로 처리 (매 audit RC1~RC18 재독)
- ❌ 시트 탭 이름 "Work" 사용 (실제: `업무`)
- ❌ 슬랙 메시지에서 지연 task 숨김 (RC11)

---

## 7. 자주 발생하는 오류 & 대처

### 시트 읽기/쓰기
- **`Unable to parse range: Work!A1:H200`** → 탭 이름이 한글 `업무`. `업무!A1:G200` 사용
- **`unexpected argument '--request-body'`** → `gws` CLI는 `--json` 옵션 사용
- **`unexpected argument '--spreadsheet-id'`** → `--params '{"spreadsheetId":"..."}'` 형식

### 슬랙 API
- 메시지가 너무 길어 **`result exceeds maximum allowed tokens`** → 파일로 저장됨, `python3` + `json.load`로 chunk 읽기
- **`response_format: concise`** 사용하면 응답 크기 감소
- `from:<@Uxxx>` 검색 시 bot user_id 직접 사용 (`<@U06AHN2ALJW>`)

### 데이터 일관성
- Row 번호 오프셋 실수 → batchUpdate 후 read-back 필수
- 프로모션 종료 시점 → 항상 EST 00:00 = KST 13:00 (PST X)
- 규칙 변경 키워드 발견 → 즉시 CLAUDE.md 섹션 업데이트 (task로 미루지 말 것)

---

## 8. 확장 자료

전체 규칙 및 사례는 CLAUDE.md 참조:
- Slack 스캔 범위 규칙 (MANDATORY 섹션)
- Promotion 라이프사이클 3-phase 규칙
- Status judgment rules
- Root cause registry (RC1~RC18)
- Mistake prevention workflow 체크리스트
- Overdue task reporting rule
- Team & R&R (source of truth)

**본 SKILL.md는 CLAUDE.md의 모든 운영 규칙을 실전 audit 수행에 필요한 형태로 내장**하고 있으므로, skill만 읽어도 독립적으로 실행 가능해야 한다. CLAUDE.md가 진화하면 본 SKILL.md도 동기화 필요.
