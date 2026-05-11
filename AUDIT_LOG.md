# TIRTIR Audit Log

> **단일 진실의 원천 (Single Source of Truth)**: 이 파일은 체크리스트(행) + 실행 기록(열)을 모두 포함.
> RC 규칙 정의는 [CLAUDE.md](./CLAUDE.md) 참조. 이 외 audit 관련 별도 파일 없음.
>
> **사용법:**
> 1. 매 audit 시작 시 우측에 **새 열 추가** (헤더: `YYYY-MM-DD HH:MM`)
> 2. 한 행씩 순차 실행 → **완료 즉시 셀에 `✅ HH:MM (노트)` 표시** → 다음 행 (incremental — batch 기록 금지)
> 3. 단계 건너뛰기 금지 (RC16). "기억"으로 처리하지 말고 매번 실제 실행
> 4. 발송 전 LAST 단계 (RC17 last-mile + RC18 주간 분류 검증) 필수
> 5. **RC19 (CRITICAL)**: 시트 Result의 `ts` 참조는 매 audit 양 채널 thread 재독

## Audit 진행 매트릭스

| # | 단계 | 체크 항목 | RC | 4/7 19:30 | 4/8 11:30 | 4/13 13:00 | 4/15 14:00 | 4/16 15:30 | 4/17 11:22 | 4/22 | 5/11 09:30 |
|---|------|-----------|-----|-----------|-----------|------------|------------|------------|------------|------|------|
| 1.1 | 1 | C0AMP96KZHB(고객사) 페이지네이션 끝까지 | RC2 | ✅ 19:32 (1페이지 완료, 채널 시작 3/20) | ✅ 11:35 (1페이지=전체, 채널 시작 3/20, 가장 최근 4/8 10:13 박서우 자사몰 마케팅 플랜) | ✅ 13:05 (3/20~4/10 전체 스캔. 4/10 이후 메시지 없음. 주말+월요일 오전 채널 비활동) | ✅ 14:10 (3/20~4/15 전체 2페이지 스캔. 최근 4/15 11:22 OOS 처리 요청 박서우) | ✅ 15:02 (최근 4/16 11:42 이동호 리포트 대시보드 공식 안내까지 스캔) | ✅ 11:25 (최근 4/17 10:27 김소형 메시지까지 스캔. 4/16 18:15 자연님 5/1 Glow Gift Festival 신규 프로모션 + 4/16 17:02 이동호 SEO/AEO 공지 + 4/16 23:12 황아인 번들 재고 문의 3건 신규)  ✅ 10:40 (최근 4/22 09:31 황아인 박서우 콜 ts 1776817876 까지 스캔. 4/21 11:18 클리어런스 딜 배너, 4/21 15:29 인플루언서 에셋, 4/20 18:07 자연님 프로모션 페이지 이슈 3건 신규 발견) | 5/11 09:30 |
| 1.2 | 1 | C0ALWLLQFU7(내부) 페이지네이션 끝까지 | RC2 | ✅ 19:34 (1페이지 완료, 가장 최근 4/7 19:00 [4/8 TDL]) | ✅ 11:35 (1+2페이지=전체, 채널 시작 3/18, 가장 최근 4/8 11:30 동호 자사몰 플랜 회신) | ✅ 13:05 (3/27~4/10 전체 스캔. 마지막 메시지 4/10 22:16 이동호. 3일간 비활동) | ✅ 14:10 (최근 4/15 12:25 홍유니 [SEO 일정 문의]까지 스캔) | ✅ 15:02 (최근 4/16 11:17 홍유니 helpcenter히스토리 요청, 4/16 11:13 이동호 R&R 정리까지 스캔) | ✅ 11:25 (최근 4/16 19:13 김소형까지 스캔. 4/16 16:28 이동호 자사몰 매출 분석+광고 캠페인 개선 Tasks / 4/16 14:57 황아인 PDP 리뷰페이지 수정 / 4/16 12:01 홍유니 제품 USP 발굴 AI 요청)  ✅ 10:45 (최근 4/21 22:30 봇 알림까지 스캔. 4/21 18:10 홍유니 4/22 미팅 준비, 4/21 13:35 자사몰 하단 영상, 4/20 01:02 홍유니 Spring Matcha 라이브 준비, 4/19 19:25 홍유니 구글 운영 변경 등 다수) | ✅ 00:24 (양 채널 sub-agent 병렬 스캔 — 클라이언트 52 parents/25 threads 전수. 내부 채널 동일) |
| 1.3 | 1 | Thread: N replies 있는 모든 부모 → slack_read_thread | RC3 | ✅ 이번 세션 다수 thread deep-read 완료 | ✅ 4/8 신규 thread (자사몰 마케팅 플랜) deep-read | ✅ 4/8~4/10 모든 thread deep-read: 브랜드뉴스(10), 말차가격(4), 배송지연(3), 디자인미팅(4), 프로모가능사항(18), 구독제(12), 권한추가(21) 등 | ✅ 14:12 (상세페이지 이미지 오류 thread, OOS 처리 thread, PDP 수정 thread, Primer 기획 thread, SEO 일정 thread 등 deep-read) | ✅ 15:05 점검사항 전달 thread, 자사몰 하단 영상 thread, ABB 국내 제외 thread, visit 리타겟팅 thread, R&R 정리 thread 모두 deep-read | ✅ 11:40 sub-agent 병렬 deep-read: 5/1 Glow Gift Festival thread (reply 없음, setup 9개 sub-task 필요), Spring Matcha 번들 재고 thread (박서우 4/17 US 236개 정정), SEO/AEO thread (자연님 4/17 GSC 권한 부여 약속), BFCM 비활성화 thread (자연님 허가), PDP 리뷰페이지 thread (홍유니 후순위 합의), Google 광고 개선 thread (홍유니 4월말-5월초 리빌딩), USP 발굴 thread (reply 없음) 모두 확인  ✅ 10:50 Glow Gift Festival 기획 thread(김소형 4/21 21:36 타임라인 재설정 + 황아인 4/21 21:45 동의), 4/22 미팅 준비 thread(홍유니 아젠다 3종 + 이동호 레버 대시보드 연결 20:24), 프로모션 페이지 이슈 thread(자연님 4/20 이동호 해결 완료), 클리어런스 딜 배너 수정 thread, Primer 4/27 thread(프라이머 2종 목요일 리스팅) 모두 deep-read | 5/11 09:30 |
| 2.1 | 2 | 매도비(U06AHN2ALJW) 봇 메시지 전체 검색 | RC12 | ✅ 19:36 (검색 → 4/6 EOD, 4/7 EOD 2건 발견) | ✅ 11:38 (4/7 20:02 EOD 최종 봇 + 4/7 11:07 + 4/6 19:05 + 3/30 12:52 + 3/27 12:52 발견) | ✅ 13:08 (4/8 오전/EOD, 4/9 EOD, 4/10 오전 총 4건 발견) | ✅ 14:13 (4/14 10:05 EOD만 신규 발견. 이후 봇 메시지 없음) | ✅ 15:06 (4/15 14:15 EOD가 최종. 이후 봇 메시지 없음) | ✅ 11:43 (4/16 11:57 EOD 봇 메시지가 최종. 이후 신규 봇 메시지 없음)  ✅ 10:55 4/21 09:57 EOD + 4/20 09:23 EOD + 4/21 22:30 알림 + 4/21 09:01 알림 + 4/20 11:30 알림 + 4/20 09:01 알림 총 6건 발견 | ✅ 00:18 (매도비 봇 5/8 09:19 / 5/7 09:23 / 5/6 09:18 / 5/5 09:31 / 5/4 09:38 / 5/1 / 4/30 / 4/29 / 4/28 EOD 9건 + 일일 리포트 / 알림 메시지 다수 발견) |
| 2.2 | 2 | 각 봇 메시지 thread → slack_read_thread (사람 reply 확인) | RC12 | ✅ 4/6 EOD thread → 황아인 BB 메일 발송 reply 발견, 4/7 EOD thread → 이동호 4건 상태 변경 reply 발견 | ✅ 4/7 20:02 EOD 봇 thread → **이동호 20:09 "레버 엑스퍼트 워크 스페이스 생성 완료" reply 발견** (R84 → Complete) | ✅ **4/8 EOD thread → 황아인 19:10 "BB자료5건 수령완료/쇼피파이계정완료/April US ON유지"**. 4/10 오전 thread → 황아인 18:54 "Matcha+ABB 소재 세팅완료/권한 전부완료/Spring Matcha RE 기획 컨펌완료" | ✅ 14:14 (4/14 EOD thread 재독 — No thread replies. 4/14 정정 reply 2건만 기록됨) | ✅ 15:06 (4/15 EOD thread 재독 — No thread replies) | ✅ 11:43 (4/16 11:57 EOD 봇 thread 재독 — 홍유니 reply "말차는 미국 / 에이프릴은 전체로 세팅한것 맞습니다" 확인. 그 외 status update reply 없음)  ✅ 10:57 4/21 09:57 EOD thread 재독 (thread_not_found 에러로 직접 접근 불가, 채널 스캔으로 확인) — 별도 team reply 없음. 4/20 EOD thread도 동일 | ✅ 00:25 (5/8 EOD 봇 thread 재독 — No thread replies. 5/7 EOD thread → 홍유니 잔여 리스트 정리 reply 발견하여 봇이 이미 반영) |
| 3.1 | 3 | 모든 Processing 상태 task의 슬랙 thread 재확인 | RC3 | ✅ 광고 계약서 thread 재독 → R20 Complete 처리 (사용자 지적) | ✅ R17/R81/R85/R86/R107/R132 모두 thread 재독, 새 변경 없음 (R86은 박서우 4/8 약속 대기) | ✅ R17/R21/R85/R107/R122/R124/R133/R145/R146/R148/R150 모두 검증. R21 변경없음(17일지연), R107 GA4 unblock후 5일 분석 미공유 | ✅ 14:20 R17/R21/R87/R101/R122/R124/R126/R133/R145/R146/R148/R150/R156/R165 재검증. R87 오늘 D-day, R156 Due 앞당김 확인 | ✅ 15:08 R17/R21/R87(→Complete)/R101/R122/R124/R126/R133/R145/R146/R148/R150/R156/R165/R167/R168/R169/R170(→Complete) 모두 재검증 | ✅ 11:48 R17/R21/R101/R122/R124/R126/R133/R145/R146/R148/R150/R156/R165/R167/R168(→Complete)/R169/R171/R172/R173 모두 재검증. R168 공식 공지로 Complete 처리  ✅ 11:05 R17/R101/R124/R126/R133/R138/R145/R146/R148/R156/R165/R167/R169/R174-R175/R196/R198 모두 thread 재독. R138 이동호 4/21 17:47 세팅 완료, R167 홍유니 4/17 17:56 교체 진행, R196 홍유니 4/19 19:25 retargeting 완료 | ✅ 00:35 (R14/R17/R38/R41/R42/R43/R55/R92/R98/R101/R122/R123/R124/R131/R133/R145/R146/R148/R152/R154/R158/R159/R160/R163/R165/R167/R169/R176/R177/R178/R181/R185/R187/R188/R190/R193/R205/R206/R207/R208 등 활성 task 전수 재검증. Primer/Glow Gift sub-task는 라이브 완료지만 sheet Status stale — 5/8 EOD 봇 이미 "시트 업데이트 필요" 명시) |
| 3.2 | 3 | 모든 Not yet 활성 task의 thread 재확인 | RC3 | ✅ TIRTIR Clearance/No Joke Sale 종료 thread 답변 없음, R104 변화 없음 | ✅ R66/R83/R110/R125/R127 thread 재독; R127 인증 진행 중 발견 → Processing 변경 | ✅ R27(슬랙근거없음)/R36(BB dependency해소)/R38/R51/R52/R64/R87(슬랙근거없음)/R105/R118/R126(→Processing)/R129/R131 모두 검증 | ✅ 14:21 R137-R141 Spring Matcha sub-task / R155 ABB 광고 OFF / R157-R164 Primer sub-task 모두 재검증 | ✅ 15:08 Spring Matcha sub-task 4/17~4/19, Primer sub-task 4/22~4/27 변경 없음 | ✅ 11:48 R73 4/17 KST 13:00 D-day / R137-R142 Spring Matcha sub-task / R143 / R150 today / R155 today KST 12:00 / R157-R164 Primer / R174-R190 신규 변경 없음  ✅ 11:08 R14/R17/R21/R27/R38/R39/R42/R47/R54/R64/R98/R103/R143/R152-R154/R158-R164/R174-R200 활성 task 모두 재검증. R39/R47 우회 처리(레버 대시보드) Complete 판정. R192 Due 4/21 미완료 확인 | 5/11 09:30 |
| 4.1 | 4 | `is:thread after:YYYY-MM-DD` 검색 (양 채널) | RC2 | ✅ 19:42 (고객사 채널 5건 발견) | ✅ 11:40 (양 채널 after:2026-04-06 검색, 4/8 신규 7건 발견) | ✅ 13:10 (양 채널 after:2026-04-08 검색. 완료/확인/확정 키워드 검색 병행) | ✅ 14:15 (양 채널 after:2026-04-13 검색. 20+18 결과) | ✅ 15:10 (양 채널 after:2026-04-15 검색) | ✅ 11:49 (양 채널 after:2026-04-16 검색. 고객사 4건 / 내부 0건. 4/17 번들 재고 정정 thread, 메인배너 내부 제작 thread, Klaviyo 수정 thread 확인)  ✅ 10:55 양 채널 after:2026-04-17 is:thread 검색, 각 20건 반환 | ✅ 00:20 (양 채널 is:thread after:2026-04-22 검색 각 20건+ 반환. Old-parent + new-reply 패턴: AO 광고 4/30 박서우 parent + 5/7 홍유니 reply, AI 샤셰 4/28 parent + 5/7 이동호 reply 등 모두 캐치) |
| 4.2 | 4 | 검색 결과 thread_ts 모두 deep-read | RC3 | ✅ 계약서 thread, 구독제 thread, 중국세션 thread 모두 deep-read | ✅ 자사몰 4-6월 마케팅 플랜 thread, 4/8 TDL thread (인증 요청) deep-read | ✅ 프로모종료정책(자연님 EST확정), 구독제(서우님 실행안준비), 권한추가(21replies) 모두 deep-read | ✅ 14:17 SEO 일정, 상세페이지 이미지 오류, OOS 처리, PDP 수정의 건, 쇼피파이 데이터 보고서 모두 deep-read | ✅ 15:10 중국 세션 thread(권자연 4/16 회신 발견), 점검사항 thread, ABB 국내 제외 thread, visit 리타겟팅 thread 모두 deep-read | ✅ 11:49 Glow Gift Festival / 번들 재고 / SEO AEO / BFCM 비활성화 / 이메일 첫 구독 flow / Google 광고 개선 / PDP 리뷰페이지 / USP 발굴 / 클리어런스 딜배너 / R107 R167 R166 R168 thread 모두 deep-read (sub-agent)  ✅ 11:00 4/22 미팅 준비 thread / Glow Gift 기획 thread / Spring Matcha 디자인 thread / Primer 기획 thread / April Beauty Bloom 번들 thread / 프로모션 가능 사항 thread 모두 deep-read | ✅ 00:25 (검색 결과 모두 sub-agent 통해 deep-read) |
| 5.1 | 5 | Plan vs Done 검증 (모든 Processing) | RC4 | ✅ R109 BB 회신받음 → Complete | ✅ R84 워크스페이스 실제 생성 완료 확인 (이동호 20:09 명시) → Complete | ✅ 11건 Processing 모두 검증. 모두 deliverable 미완(plan≠done). R87 "시트기입-슬랙근거미확인" 플래그 | ✅ 14:22 R87은 초안 완료이나 TIRTIR 전달 미확인 → Processing 유지. R165 Notion 적재 중 → Processing 유지 | ✅ 15:12 R87 → Complete (이동호 4/16 11:42 실제 TIRTIR 전달 확인). R101/R165/R167/R168/R169 모두 Processing 유지 (deliverable 미완) | ✅ 11:50 R168 → Complete (이동호 4/16 17:02 고객사 공식 공지 실제 발송 확인). R101/R122/R124/R126/R145/R146/R156/R165/R167/R169/R171/R172/R173 모두 deliverable 미완 → Processing 유지  ✅ 11:10 R39 Databox 대체 Complete (레버 대시보드 우회, 이동호 4/21 20:24 ts 1776770678). R47 리포트 자동화 Complete (동일 솔루션). R138/R167/R174/R175/R192/R196/R198 모두 Processing/Not yet 유지 (deliverable 미완) | ✅ 00:38 (Spring Matcha Refresh 4/28 종료 황아인 ts 1777360635 deliverable 검증 / Primer 4/27 라이브 / Glow Gift 5/1 라이브 모두 deliverable 검증 — R158/R159/R160/R163/R176/R177/R178/R181 Complete 판정. R165 시트 일원화 결정으로 종결 Complete) |
| 5.2 | 5 | Scope mixing 검증 (Shopify/Media/Creative 분리) | RC5 | ✅ 이전 라운드에서 R59, R65 분할 완료 | ✅ R133 신규는 단일 owner(동호/아인) 단일 deliverable로 분리 생성 | ✅ **R73 ABB 종료/롤백 RC5 위반 감지** → Shopify(이동호 R73) + Media(홍유니 R155)로 분리 완료 | ✅ 14:23 신규 R166-R170 모두 단일 owner 단일 deliverable로 분리 생성 | ✅ 15:13 신규 R171(황아인/홍유니 co-own but 단일 deliverable)/R172(홍유니)/R173(홍유니) 모두 단일 owner | ✅ 11:50 Glow Gift Festival 9개 sub-task 각각 단일 owner 단일 deliverable로 분리 (No Joke Sale 패턴). R183~R190 모두 단일 owner. R188/R189는 홍유니 단독 광고 영역으로 R187(김소형 creative AI)과 별도 분리  ✅ 11:12 신규 R201 Klaviyo 이미지(황아인 단일 owner 단일 deliverable), R202 Shopify Shop 광고 체크(홍유니/이동호 co-own but 단일 deliverable — 체크·분석 단계라 분리 불필요) | ✅ 00:38 (Primer/Glow Gift 종료 task 신규 = Shopify(이동호) + 광고 OFF(홍유니) 분리. AI Sachet sub-task 8개 단일 owner 단일 deliverable 분리. 5-6월 인플루언서 광고 단일 owner 홍유니) |
| 5.3 | 5 | 권한 요청 task → 우회 처리 가능성 체크 | RC13 | ✅ R116 우회 처리 완료, R86 GA4 권한 박서우 4/8 부여 약속 | ✅ R86 박서우 4/8 부여 예정, R127 우회 없음 (인증 단계) | ✅ R85 MCC 권한 추가됨(아인 4/10), CID 직접 접근은 도메인 제한으로 우회 불가 | ✅ 14:23 신규 권한 요청 task 없음 | ✅ 15:13 신규 권한 요청 task 없음 | ✅ 11:51 R190 신규: GSC 권한 부여 요청(us.team@madup.com, bc_team3@madup.com). 자연님 4/16 "내일 부여" 약속 → 4/17~4/18 실제 부여 여부 follow-up 필요  ✅ 11:13 신규 권한 요청 task 없음 | ✅ 00:38 (Find My Shade SKU 자동 카트 — 업체 컨택 우회 진행. Klaviyo Shop Now 이미지 — 한 줄 2개 우회 솔루션 김소형 5/2 답변) |
| 5.4 | 5 | 전달/발송 task → 실제 발송 여부 확인 | RC14 | ✅ R23, R20 모두 발송 확인 후 Complete 처리 | ✅ R125 클린본 전달 미발송 확인 (오늘 발송 예정) | ✅ R87 레버 대시보드 안내 미전달 확인 (4일 지연, D-day). R147 브랜드뉴스 발송완료→Complete | ✅ 14:24 R87 오늘 전달 예정(슬랙 미확인) → Processing 유지 + 지연 표기 생략(D-day). R168 SEO 답변 today 미발송 → Processing | ✅ 15:14 R87 → **Complete** (이동호 4/16 11:42 TIRTIR 공식 안내 발송 확인) | ✅ 11:51 R168 → **Complete** (이동호 4/16 17:02 SEO/AEO 공식 공지 실제 발송 확인 ts 1776672157)  ✅ 11:14 R138 Klaviyo 발송 완료 확인 필요(이동호 4/21 17:47 세팅 + AI 최적 시간 발송 예약). R192 Shopify 실행 확인 필요 | ✅ 00:38 (4월 매체 정산서 5/7 발송 완료 / Lever Xpert 대시보드 5/7 client 공유 완료 / 베스트셀러 변경 5/8 1차 적용 완료 — 모두 Complete 판정. 4월말 Tasks+5월 진행계획 client 공유는 미발송 → 11일 지연) |
| 5.5 | 5 | Umbrella task → 70%+ 완료 + 잔여 분리 검토 | RC15 | ✅ R65, R59, R20 모두 분할 + 잔여 task 생성 | ✅ 신규 분할 필요 없음 | ✅ R73 ABB 종료 → RC5로 분리 (5.2에서 처리). 기타 분할 대상 없음 | ✅ 14:24 신규 분할 대상 없음 | ✅ 15:14 신규 분할 대상 없음 | ✅ 11:51 신규 umbrella 감지: 5/1 Glow Gift Festival → 9개 sub-task로 사전 분리 생성 (R174-R182). RC1 prevention 적용  ✅ 11:15 신규 분할 대상 없음. Matcha Refresh 4/27 종료·롤백은 Shopify(이동호)+광고 OFF(홍유니) 분리 필요하나 sheet에 task 미존재 — draft에서 flag | ✅ 00:38 (Primer 종료 = Shopify(이동호)+광고 OFF(홍유니) 분리 신규. Glow Gift 종료 동일 분리 신규. AI Sachet sub-task 8개 분리 신규) |
| 6.1 | 6 | 시트 모든 task의 슬랙 근거 확인 | RC6 | ✅ 모든 신규 task에 ts/근거 명시 | ✅ R84/R127/R133 모두 ts 명시 | ✅ R27(상시소재) 슬랙근거없음 플래그. R87(레버대시보드) 슬랙근거없음 플래그. 신규 6건 모두 ts 명시 | ✅ 14:24 R166~R170 모두 슬랙 근거 명시 | ✅ 15:15 R171/R172/R173 모두 슬랙 근거 명시 (4/15 17:52 슬라이드/4/15 19:21 ABB 제외 요청/4/15 19:31 visit 리타겟팅) | ✅ 11:52 R174-R190 모두 슬랙 ts 명시: Glow Gift (ts 1776676532), 번들 재고 (ts 1776701544), BFCM (ts 1776667365), 하단 영상 (ts 1776241151), 3D모델링 (ts 1776239314), USP발굴 (ts 1776661277), Google 광고 리빌딩 (ts 1776677298), GSC 권한 (ts 1776672157)  ✅ 11:16 R201 ts 1776756295/1776817876, R202 ts 1776762618/1776772719 모두 명시. R39/R47은 기존 task Status 변경이며 근거 ts 1776770678 명시 | ✅ 00:38 (모든 신규 task에 ts 명시 — GMC ts 1778231941 / Summer Big Sale ts 1778234837 / 베스트셀러 ts 1778221147 / VAT ts 1778207278 / 프라이머 후속 번들 ts 1778029978 / V포토북 ts 1777538236 / 5-6월 인플 ts 1777367548 등) |
| 7.1 | 7 | 내부 메시지 → 클라이언트 종속 요청 감지 | RC7 | ✅ 레버 엑스퍼트 GA4 권한 요청 transfer task 존재 | ✅ R133 (자사몰 플랜) 클라이언트 회신 transfer task로 생성 | ✅ R124 Google 결제 김현서 재승인 대기. R21 모두싸인 TIRTIR 대기. 4/9 VC follow-up TIRTIR 액션 3건 확인 | ✅ 14:25 R170 중국 BOT 차단 앱 검토 (TIRTIR 의사결정 대기) transfer-like task로 생성 | ✅ 15:15 R171 점검 사항 고객사 전달 예정 (내부 정리→외부 전달). R172 자연님 helpcenter 답변 후 프로모션별 적용 (dependency on TIRTIR) | ✅ 11:52 R190 GSC 권한 부여 follow-up (TIRTIR owner, 자연님 4/17 약속). R183 번들 재고 정정은 박서우가 실제 수량 이미 공유 (TIRTIR side 액션 완료). R177 Glow Gift 메인배너 고객사 제작 (4/23 고객사 데드라인)  ✅ 11:17 R201 박서우 리마인드(최종 컨펌 대기 TIRTIR 액션). R202 TIRTIR Shop 광고 확인 필요. VC 미팅(R198)이 최대 client-dependency transfer surface 예정 | ✅ 00:38 (Today 5/11 client 요청 4건 transfer task 명시: GMC Warehouse/Summer Big Sale 답변/VAT 답변/AI Sachet 디자인 일정. AO 5월 개선안 client 전달 4일 지연) |
| 8.1 | 8 | 운영 규칙 변경 키워드 검색 (기준/변경/앞으로) | RC8 | ✅ PST 24시간제 규칙 CLAUDE.md 반영됨 | ✅ 신규 규칙 변경 없음 | ✅ **프로모종료시간 EST 00:00 변경 감지** (자연님 4/9 확정). CLAUDE.md 즉시 업데이트 완료 | ✅ 14:25 신규 운영 규칙 변경 없음 (4/14 swap 이후 변화 없음) | ✅ 15:16 신규 운영 규칙 변경 없음. 이동호 R&R 3~6개월 제안은 내부 논의 중 (규칙 확정 전) | ✅ 11:53 자연님 4/16 08:55 "이후 프로모션부터 국내+일본 제외타깃 기본 적용" - 신규 운영 규칙 감지. R180 Glow Gift 광고 세팅 Description에 반영 완료  ✅ 11:18 신규 운영 규칙 변경 없음. 4/14 프로모션 시간대 swap + 4/16 국내+일본 제외타깃 기본 적용 규칙 기존 반영됨 | ✅ 00:38 (운영 규칙 변경 5건 감지: Klaviyo SHOP NOW 이미지 형태 4/28 박서우 / 글로벌 쉐이드 차별 주의 5/1 박서우 CS 2건 / 프로모션 라이브 시간 communication 사전 확인 4/27 / 광고 소재 디자인 가이드 4/30 박서우 / AO 신규 소재 cadence 5/7 홍유니 2주 목표) |
| 9.1 | 9 | 지연 task → N일 지연 표기 (Due 임의 변경 금지) | RC11 | ✅ Due 변경은 모두 슬랙 ts 증거 함께 기록 (이동호 4/30 재설정 등) | ✅ 지연 태스크 모두 N일 지연 표기 (R21 12일, R27 12일, R38/R48-52 7-8일, R54/R55/R64 1일) | ✅ 지연 태스크 전수 표기: R21(17일), R38(12일), R51/R52/R64(6일), 다수 4일/3일/5일 지연 | ✅ 14:26 R21(19일), R124/R146(5일), R122/R126(7일), R145(1일) 모두 표기. R156 Due 변경은 김소형 명시적 재설정 증거 기록 | ✅ 15:17 R21(20일), R122/R126(8일), R124/R146(6일), R145(2일), R101/R165/R168(1일 지연) 모두 표기. Due 임의 변경 없음 | ✅ 11:53 R21(21일), R122/R126(9일), R124/R146(7일), R145(3일), R101/R165(2일 지연), R156/R167/R172(1일 지연) 표기. R14 Due 4/14→4/30 (이동호 4/16 17:02 공식 공지 명시적 재설정 ts 1776672157) — 정당한 Due 변경  ✅ 11:20 R21(26일), R122/R126(14일), R124/R146(12일), R133(13일), R138(3일), R143(5일), R145(8일), R148(3일), R165(7일), R167(6일), R169(4일), R183(4일), R185(3일), R192(1일) 모두 표기. Due 변경 2건: R174 4/21→4/22 (김소형 4/21 21:36 ts 1776775000 + 황아인 21:45 ts 1776775531), R175 4/25→4/29 (같은 재설정) 모두 슬랙 ts 증거 | ✅ 00:38 (계약서 45일/R38 40일/R55 34일/R122 33일/R123-R145 27-32일/R133 32일/R146 31일/R92 28일/R41-R43 26일/R148 22일/R169 23일/R185 22일/R152-R154 21일/R101 17일/R14-R17-R42-R98-R187 11일/R206 13일 등 표기. Due 임의 변경 0건) |
| 10.1 | 10 | 신규 다중 deliverable 이니셔티브 → sub-task 분해 | RC1 | ✅ 4/8 TDL → 6개 sub-task 분해 | ✅ 4-6월 마케팅 플랜은 단일 deliverable (회신) → R133 단일 task | ✅ 4/9 VC follow-up 3건 분해(R152/R153/R154). 배송 지연/번들 버그 등 개별 task 생성 | ✅ 14:27 신규 umbrella 없음. R166-R170 모두 단일 deliverable | ✅ 15:17 신규 umbrella 없음. R171~R173 모두 단일 deliverable | ✅ 11:54 5/1 Glow Gift Festival → 9개 sub-task 분해 (R174-R182, No Joke Sale 패턴 준수). 타 신규(R183-R190)는 모두 단일 deliverable | ✅ 11:21 Matcha Refresh 4/27 종료·롤백 umbrella 신규 감지 → draft에 Shopify 롤백(이동호)+광고 OFF(홍유니) 분리 언급 (추후 task 생성 필요 flag) | ✅ 00:38 (AI Sachet 5/15 → 8 sub-task 분해. Primer/Glow Gift 종료 → 각각 Shopify+광고 분리. 5-6월 인플루언서 광고 → 단일 deliverable 유지. V포토북 → 기술 가능 여부 확인 후 9개 sub-task 분해 예정) |
| 11.1 | 11 | batchUpdate row 번호 정확성 검증 | RC9 | ✅ R&R 시트 실수로 삭제 → 즉시 복구 (다른 RC9 위반은 없음) | ✅ R84(row84), R127(row127), R133(row133) 모두 batchGet으로 사전 검증 + 사후 verify 완료 | ❌ **RC9 위반**: 초기 batchUpdate에서 row 45-47, 64에 잘못된 데이터 기록 (파일 라인번호≠시트행번호). 즉시 감지→복구→python으로 정확한 행번호 매핑 후 재적용 | ✅ 14:28 batchUpdate 7건 (R166-R170 신규 + R87 Result + R156 Due/Result). 사후 R166-R170 read-back verify 성공 | ✅ 15:20 batchUpdate 6건 (R87 E/F + R107 F + R170 E/F + R171-R173 신규). 사후 R87/R170/R171-R173 read-back verify 성공 | ✅ 11:55 batchUpdate 21건 (R14 D/E/F, R17 F, R168 E/F, R172 F + R174-R190 신규 17건). 사후 R14/R17/R168/R174-R190 read-back verify 성공. Row 번호 misalignment 0건 | ⚠️ 11:28 batchUpdate 9건 (R39/R47 E·F + R138 F + R167 F + R174 D·E·F + R175 D·E·F + R192 F + R196 F + R198 F). **RC9 부분 위반: R174/R175 range D:F 입력 시 값 3개가 D/E/F로 매핑되며 E(Status) 컬럼에 owner 이름(황아인/김소형) 덮어쓰기** 즉시 감지 + E 컬럼만 "Not yet" 복구. 사후 R174/R175 read-back verify 성공. 신규 2건 append(R201/R202) verify 성공 | ⚠️ 00:40 GWS sheets API auth 영구 실패 (invalid_grant — refresh token 만료). batchUpdate 39건 (status 변경 10건 ×2 + 신규 19건) 미적용. 슬랙 메시지에 "시트 업데이트 필요" 섹션 명시. /tmp/sheet_batch.json 보존 (14430 bytes). |
| L.1 | LAST | 발송 직전 30분~2시간 슬랙 cross-check | RC17 | ✅ 19:42 last-mile check 실행 | ✅ 11:42 last-mile (4/7 EOD audit 이후 ~16시간 격차) | ✅ 14:23 양 채널 최근 5건 확인. 내부: 김소형 4/13 10:32 파일 첨부(텍스트 없음). 고객사: 4/10 이후 비활동. 변경사항 없음 | ✅ 14:29 양 채널 최근 5건 재조회. 변경사항 없음 | ✅ 15:22 양 채널 최근 10건 재조회. 4/16 11:42 이동호 리포트 대시보드(이미 반영), 4/16 11:13~11:17 내부 R&R/helpcenter 논의 (규칙 변경 아님). 변경 없음 | ✅ 11:58 양 채널 최근 10건 재조회. **4/17 11:31 이동호 내부 FYI "Klaviyo 이름 매칭 삽입 주의사항"** (informational, action 없음). 기타 변경 없음  ✅ 11:30 양 채널 최근 10건 재조회. 4/21 22:30 봇 알림(R174/R192 미완료), 4/21 09:57 EOD 봇 메시지, 4/22 09:31 황아인 박서우 콜 진행 중(R201 연관) 이미 draft 반영 | 5/11 09:30 |
| L.2 | LAST | 새 메시지 thread 모두 deep-read | RC17 | ✅ 4/7 17:23 박서우 클린본 요청, 4/7 15:58 박서우 PoC 조건, 4/7 17:37 박서우 GA4 권한 약속 모두 발견 | ✅ 4/8 10:13 박서우 자사몰 플랜, 4/8 11:02 황아인 내부 검토, 4/8 11:30 동호 분석 모두 반영 | ✅ 김소형 4/13 10:32 메시지 thread 확인 — 파일 첨부만, reply 없음 | ✅ 14:29 변화 없음 | ✅ 15:22 변화 없음 | ✅ 11:58 이동호 4/17 11:31 FYI thread 확인 — 단일 메시지, reply 없음, 새로운 task 시사하지 않음. 기존 Klaviyo 이름 관련 디자인 참고사항  ✅ 11:31 4/22 09:06 박서우/09:31 황아인 전화 건 확인 — R201로 task化 반영. 기타 신규 thread 없음 | 5/11 09:30 |
| L.3 | LAST | draft에 영향 있는 task 변경사항 반영 | RC17 | ✅ R109 Complete, R81/R86 Result 업데이트, R132 신규 추가 | ✅ R84 Complete, R127 Processing, R133 신규 모두 draft에 반영 | ✅ R85 Complete (사용자 확인: Google Ads 권한 확보) → draft P0에서 제거, Complete에 추가 | ✅ 14:30 draft에 R166-R170 / R87 / R156 모두 반영 | ✅ 15:23 draft에 R87(→Complete), R170(→Complete), R107(Result 업데이트), R171-R173 신규 모두 반영 | ✅ 11:59 draft에 R168(→Complete) + Glow Gift 9개(R174-R182) + 단일 task 8개(R183-R190) + R14/R17/R172 업데이트 모두 반영  ✅ 11:32 draft에 R39(Complete)/R47(Complete)/R201(신규)/R202(신규)/R174-R175(Due 재설정)/R138/R167/R192/R196/R198 모두 반영 | ✅ 00:43 신규 변경사항 없음으로 draft 추가 반영 불필요. R209+ 신규 task는 시트 미반영 상태 + 슬랙 메시지에서 descriptive label로 처리. |
| L.4 | LAST | 주간 분류 검증 (이번주/다음주 경계 명시) | RC18 | ❌ 위반 감지 — 4/10, 4/11을 다음주로 잘못 분류 (사용자 지적) | ✅ 명시: today=4/8(Wed) → 이번주 4/6(월)~4/12(일), 다음주 4/13(월)~4/19(일). 4/10, 4/11 모두 이번주 | ✅ 명시: today=4/13(Mon) → 이번주 4/13(월)~4/19(일), 다음주 4/20(월)~4/26(일) | ✅ 14:30 today=4/15(Wed) → 이번주 4/13(월)~4/19(일), 다음주 4/20(월)~4/26(일). draft에 명시 | ✅ 15:23 today=4/16(Thu) → 이번주 4/13(월)~4/19(일), 다음주 4/20(월)~4/26(일). draft에 명시 | ✅ 11:59 today=4/17(Fri) → 이번주 4/13(월)~4/19(일), 다음주 4/20(월)~4/26(일), 다다음주 4/27(월)~5/3(일). draft에 3개 boundary 모두 명시 (Glow Gift 5/1 라이브 커버)  ✅ 11:33 today=4/22(Wed) → 이번주 4/20(월)~4/26(일), 다음주 4/27(월)~5/3(일). draft에 명시 | 5/11 09:30 |
| L.5 | LAST | Draft 내 모든 열거 task의 시트 Status 재조회 | RC20 | - | - | - | ✅ 14:31 draft 내 R87/R101/R156/R165/R73/R155/R142/R137-R141/R157-R164/R148/R169 등 모두 시트 Status 재조회. Complete 없음 | ✅ 15:24 draft 내 R87(Complete → 완료섹션), R170(Complete → 완료섹션), 이번주 마감/다음주 섹션 모두 시트 Status 재조회 | ✅ 12:00 draft 내 모든 열거 task(R14/R17/R21/R68/R73/R98/R101/R122/R124/R126/R133/R137-R143/R145/R146/R148/R150/R152-R173/R174-R190) 시트 Status 재조회. R168 Complete로 반영, 기타 Complete 없음  ✅ 11:34 draft 내 모든 열거 task(R14/R17/R21/R39/R47/R98/R101/R103/R122/R124/R126/R133/R138/R143/R145/R146/R148/R157-R164/R165/R167/R169/R174-R202) 시트 Status 재조회. R39/R47 Complete 반영. 기타 Processing/Not yet 유지 | ✅ 00:43 draft 내 모든 열거 R번호 (R14~R208 37개) 시트 Status 재조회. 시트 stale 항목 (R158/R159/R160/R163/R165/R176/R177/R178/R181/R208)은 "시트 업데이트 필요" 섹션으로 명시 (사용자 시트 미동기화 → 슬랙 메시지에서만 Complete로 표기). |
| L.6 | LAST | Draft 본문 grep 금지표현 확인 (CLAUDE.md/skill/AUDIT_LOG/RC[0-9]+/META-RULE) | RC21 | - | - | - | ✅ 14:32 grep CLEAN (매치 0건) | ✅ 15:24 grep CLEAN (매치 0건) | ✅ 12:00 초기 grep에서 "RC8 trigger" 표현 1건 감지 → 즉시 제거 후 재grep CLEAN  ✅ 11:35 grep CLEAN (금지 표현 패턴 매치 0건) | ✅ 00:43 초기 grep에서 "RC8 — CLAUDE.md 반영 필요" 1건 감지 → 제거 후 재grep CLEAN (매치 0건). 발송 전 예방 성공. |
| S.1 | 요약 | RC 위반 감지 건수 | - | 1건 (RC18 — 발송 후 사용자 지적) | 0건 | 2건 (RC9 row misalignment — 즉시 복구, RC16 AUDIT_LOG 미작성 — 사용자 지적 후 작성) | 0건 | 0건 | 1건 (RC21 사전감지 — draft grep에서 "RC8 trigger" 발견, 발송 전 제거 완료. 발송 전 예방 성공)  1건 (RC9 부분 위반 — R174/R175 batchUpdate range D:F에서 E 컬럼에 owner 이름 덮어쓰기. 즉시 감지 + 복구 완료. 실제 값 손실 없음. 발송 전 예방 성공) | 1건 (RC21 사전감지 — draft grep에서 "RC8 — CLAUDE.md 반영 필요" 표현 발견, 발송 전 제거 완료) + 환경적 제약 1건 (GWS sheets auth 영구 실패, 시트 batchUpdate 미적용 — RC9 적용 시점 아님) |
| S.2 | 요약 | 발견된 신규 task / 상태 변경 건수 | - | 신규 1건 (R132 구독제 PoC) + Complete 3건 (R109, R115, R128) + Result 업데이트 2건 (R81, R86) | 신규 1건 (R133 자사몰 4-6월 플랜) + Complete 1건 (R84) + Processing 1건 (R127) | 신규 8건 (R149-R155) + Complete 12건 (BB5건, R71, R136, R147, R149, R132, R144 Result) + Processing 3건 (R124, R145, R126) + RC5 분리 1건 (R73→R73+R155) + 규칙변경 1건 (EST 00:00) | 신규 5건 (R166-R170) + Complete 1건 (R166 OOS 당일 완료) + Processing 3건 (R167/R168/R169) + Not yet 1건 (R170) + Due 변경 1건 (R156 4/17→4/16) + Result 업데이트 1건 (R87) | Complete 2건 (R87 레버 대시보드 TIRTIR 전달 / R170 중국 BOT 차단 자연님 결론) + Result 업데이트 1건 (R107 권자연 4/16 회신 + GA4 세그먼트) + 신규 3건 (R171 4/15 미팅 점검사항 정리, R172 ABB 국내+일본 제외, R173 visit 리타겟팅 재문의) | 신규 17건 (R174-R182 Glow Gift Festival 9개 sub-task + R183-R190 단일 태스크 8개) + Complete 1건 (R168 SEO/AEO 공식 공지) + Due 변경 1건 (R14 4/14→4/30 이동호 명시적 재설정) + Result 업데이트 2건 (R17 Merchant Center 통합, R172 스코프 변경) + 운영 규칙 변경 1건 (프로모션 국내+일본 제외타깃 기본 적용)  Complete 2건 (R39 Databox 레버 대시보드 우회 / R47 리포트 자동화 동일 솔루션) + 신규 2건 (R201 Klaviyo 팝업 이미지 수정 컨펌 / R202 Shopify Shop 광고 체크 VC 후속) + Due 재설정 2건 (R174 4/21→4/22 / R175 4/25→4/29 김소형 타임라인 재설정) + Result 업데이트 5건 (R138/R167/R192/R196/R198) | 5/11 09:30 |

## 범례

- ⏳ = 진행 전
- 🔄 = 진행 중
- ✅ = 완료 (시각 기록)
- ❌ = 실패 / skip (사유 기록)
- ⚠️ = 부분 완료 (사유 기록)

## 진행 노트

### 5/11 audit
- 시작 시각: 2026-05-11 09:18 KST (월요일)
- 종료 시각: 2026-05-11 09:45 KST
- 트리거: 사용자 `/tirtir-audit` skill 호출, "organize tasks for the TIRTIR campaign"
- 주간 경계: today=5/11(Mon) → 이번주 5/11(월)~5/17(일), 다음주 5/18(월)~5/24(일)
- 4/22 audit 이후 19일 경과 (4/28~5/8 사이 봇 EOD 9건 자동 발송: 4/28/4/29/4/30/5/1/5/4/5/5/5/6/5/7/5/8). manual audit은 본 5/11이 첫 round.
- 발송 ts: 1778459905.960369 (C0ALWLLQFU7 내부 채널)
- **환경 제약 1건 — GWS sheets API auth 영구 실패**: `invalid_grant` (refresh token 만료). 시트 batchUpdate 39건 (상태 변경 10건×2 + 신규 19건) 미적용. /tmp/sheet_batch.json 보존. 슬랙 메시지에서 "시트 업데이트 필요" 섹션 + descriptive 신규 task 명시로 우회.
- 주요 발견:
  - **Complete 추정 10건 (시트 미적용)**: R158/R159/R160/R163 (Primer 4/27 라이브 sub-task), R176/R177/R178/R181 (Glow Gift 5/1 라이브 sub-task), R165 (Klaviyo Notion → 시트 일원화 결정으로 종결, 이동호 5/4 12:46 ts 1777862813), R208 (Meta Ads Audience Segments — 이동호 4/25 15:46 직접 세팅 완료)
  - **신규 task 19건 (시트 미적용, 슬랙에 명시)**:
    - AI Sachet 5/15 라이브 sub-task 8개 (PDP/Find My Shade follow-up, Shopify, 리스팅, Klaviyo, Meta/Google 캠페인, QA, 라이브)
    - 프로모션 종료/롤백 4개 (Primer 종료 Shopify+광고, Glow Gift 종료 Shopify+광고)
    - 프라이머 후속 번들 2종 + 번들 SOP 문서화 (이동호/황아인, Due 5/9)
    - V포토북 6/1~10 기술 가능 여부 검토 (이동호, Due 5/11 today)
    - 5-6월 유튜브 인플루언서 파트너 광고 (홍유니, 5월 3주차~6월 2주차, 4천만 paid + 1천만 섭외)
    - Today 5/11 client 응답 4건 (GMC Warehouse, Summer Big Sale 답변, AI Sachet 디자인 일정, VAT 답변)
    - 4월말 Tasks + 5월 진행계획 client 공유 (이동호, Due 4/30 자연님 명시 — 11일 지연)
  - **운영 규칙 변경 5건 감지 (CLAUDE.md 후속 반영 필요)**:
    - Klaviyo SHOP NOW 버튼 이미지 형태 적용 (4/28 박서우)
    - 글로벌 쉐이드 할인 차별 주의 (5/1 박서우, CS 2건 발생 후)
    - 프로모션 라이브 시간 communication 사전 확인 (4/27 박서우)
    - 광고 소재 디자인 가이드 (4/30 박서우 — 미래지향 배경 지양/AI 최소화/그라데이션 지양/성별 미특정)
    - AO 광고 신규 소재 cadence (5/7 홍유니 — 2주 목표, 프로모션 변수 시 조율)
  - **Spring Matcha Refresh 종료 완료** (4/28 16:17 ts 1777360635, 황아인): 광고/딜배너 PC,MO/메인배너/가격 롤백/GWP/랜딩 redirect 모두 완료
  - **R&R 탭 stale 발견**: 박서우 신규 user_id `U0AUAKVQKGR` (vs 시트 R&R `U09D7F2BVMW`), 김현서 신규 user_id `U0ATVM0SXSA` (vs 시트 R&R `U0AMX4ACS68`). 슬랙 메시지에는 신규 active user_id 사용
- **RC 위반: 1건 사전감지**
  - RC21 (사전감지): draft grep에서 "RC8 — CLAUDE.md 반영 필요" 표현 발견, 발송 전 제거 완료. 발송 전 예방 성공
- **메모**: 시트 업데이트 미실행으로 R209+ 신규 task는 슬랙 메시지에서만 descriptive 명시. 후속 audit에서 GWS auth 복구 시 /tmp/sheet_batch.json 재적용 필요

### 4/22 audit
- 시작 시각: 2026-04-22 10:40 KST (수요일)
- 종료 시각: 2026-04-22 11:36 KST
- 트리거: 사용자 `/tirtir-audit` skill 호출, "organize tasks for the TIRTIR campaign"
- 주간 경계: today=4/22(Wed) → 이번주 4/20(월)~4/26(일), 다음주 4/27(월)~5/3(일)
- 4/17 audit 이후 5일 경과 (4/20 09:23 및 4/21 09:57에 자동 EOD 봇 메시지 기록 확인됨 — manual audit은 아님)
- 신규 발견:
  - **R39 Databox 대체 리포팅 → Complete** (이동호 4/21 20:24 tirtir.leverdashboard.com 도메인 연결 ts 1776770678, Shopify/Klaviyo 연동 진행 중. 우회 처리로 Complete 판정)
  - **R47 리포트 자동화 솔루션 도입 → Complete** (동일 솔루션 우회 처리)
  - **신규 2건**:
    - R201 Klaviyo 팝업 가입 신규 고객 이메일 이미지 수정 최종 컨펌 (황아인, 4/23)
    - R202 Shopify Shop 광고 운영 체크 (VC 후속, 홍유니/이동호, 4/28)
  - **Due 재설정 2건** (명시적 증거):
    - R174 Glow Gift 기획 4/21 → 4/22 (김소형 4/21 21:36 ts 1776775000 + 황아인 4/21 21:45 동의 ts 1776775531)
    - R175 Glow Gift 디자인 제작 4/25 → 4/29 (같은 재설정)
  - **Result 업데이트 5건**: R138 (이동호 4/21 Klaviyo 세팅 완료 + 발송 예약) / R167 (홍유니 4/17 A콘텐츠 대체 교체 진행) / R192 (Due 4/21 미완료 확인 필요) / R196 (구글 retargeting 4/19 완료, New Audience 진행 중) / R198 (VC 미팅 아젠다 3종 + 자료 준비 완료)
- **RC 위반: 1건**
  - RC9 부분 위반: R174/R175 batchUpdate range D:F에서 values 3개 입력 시 E(Status) 컬럼에 owner 이름 덮어쓰기 → 즉시 감지 + E 컬럼만 "Not yet" 복구. 실제 Due/Result 값은 정상. 사용자 지적 전 자체 감지 → 복구. 패턴: `D:F` range 배열 길이 3 시 E 컬럼이 중간 값을 받음. Prevention: range/values 매칭 시 intent 명확히 (E를 건드리지 않으려면 D 단건 + F 단건으로 분리)
- 발송 ts: 1776818707.055399 (C0ALWLLQFU7 내부 채널, 확인 없이 자동 발송)
- **RC 위반 추가 (사용자 지적으로 발견 — CRITICAL REPEAT)**: RC12 재발
  - 증상: 4/21 EOD 봇 thread(`ts 1776733061.250129`)에 4/21 16:16~16:22 팀 reply 6개가 있었으나 누락
    - 황아인 16:22 (ts 1776756121): "ABB Tint Trio Holiday Edition 쇼핑백 제거 완료하였습니다" → R192 Complete였음
    - 이동호 16:19 (ts 1776755992): "레드쿠션+워터리즘 틴트 30% 할인 세팅 건 — 이 부분 아인님 해당 쓰레드 답변으로 완료" → R143 Complete였음
  - 원인: audit 중 thread_ts를 `1776733061.000000` 형태로 추측하여 `slack_read_thread` 호출 → `thread_not_found` 에러 반환 → "reply 없음"으로 오판정. 실제 thread_ts는 `1776733061.250129`. 사용자 permalink 공유 후 정확한 ts 확보
  - 영향: 4/22 EOD 본문에서 R192 "⚠️ 1일 지연" / R143 "⚠️ 5일 지연"으로 오기. 시트 Status도 Not yet/Processing으로 유지됨
  - 대응: (1) R192/R143 시트 Status Complete + Result ts 증거 반영 (2) 정정 thread reply 발송 ts 1776836044.283439 (3) CLAUDE.md RC12 강화: thread_ts 추측 금지 + Message_ts 직접 파싱 + ts 확보 경로 우선순위 명시 (4) SKILL.md RC12 동기화
- 정정 발송 ts: 1776836044.283439 (4/22 EOD thread reply)

### 4/7 19:30 audit
- 시작 시각: 2026-04-07 19:30 KST
- 트리거: 사용자가 R20 광고 계약서가 완료된 것을 지적 → 전수 재점검 시작
- 신규 RC 추가: RC16 (checklist skipping), RC17 (last-mile freshness)

### 4/8 11:30 audit
- 시작 시각: 2026-04-08 11:30 KST (수요일)
- 트리거: 사용자 "Check all tasks, update the task sheet and give me the slack draft1"
- 주간 경계 검증: 이번주 = 4/6(월)~4/12(일), 다음주 = 4/13(월)~4/19(일)
- 4/7 19:30 audit 이후 약 16시간 경과
- 신규 발견:
  - **R84 (레버 엑스퍼트 워크스페이스 생성) → Complete** (이동호 4/7 20:09 EOD 봇 thread reply, ts 1775560155.759879). 4/13 런칭 P0 블로커 1건 해소
  - **R127 (쇼피파이 계정 생성) → Processing** (황아인 4/8 10:50 인증 요청 → 홍유니 4/8 11:04 진행)
  - **R133 (NEW) TIRTIR 자사몰 4-6월 마케팅 플랜 검토 + 회신** (박서우 4/8 10:13 제안 → 황아인 4/8 11:02 내부 검토 시작 → 이동호 4/8 11:30 분석 + TIRTIR 추가 확인 필요 사항 도출). Due 4/9
- 시트 업데이트: 3건 batchUpdate, 모두 사후 verify 완료
- RC 위반: 0건

### 4/14 11:30 audit
- 시작 시각: 2026-04-14 11:30 KST (화요일)
- 종료 시각: 2026-04-14 12:40 KST (첫 발송) / 13:00 KST (RC19 정정 reply)
- 트리거: 사용자 `/tirtir-audit` slash command
- 주간 경계: 이번주 4/13(월)~4/19(일), 다음주 4/20(월)~4/26(일)
- 4/13 13:00 audit 이후 약 22시간 경과
- 신규 발견:
  - **R107 중국 세션 분석 → Complete** (이동호 4/13 10:34 고객사 채널 thread_ts 1775525708.773669, ts 1776044056.307869. GA4 3/16~4/12 전수 분석: Paid Traffic 7 sessions 광고 무관, Bot/크롤러 트래픽 결론. csv 첨부)
  - **신규 11건 추가**: R156~R164 Primer Launch Deal 9개 sub-task (자연님 4/13 15:56 기획안, 4/27 라이브, ts 1776063366.786039) + R165 Klaviyo Notion 적재 (이동호 4/13 21:21, ts 1776082886.561019)
  - **업데이트 3건**: R87 레버 대시보드 (Owner 홍유니→이동호, Due 4/9→4/15, 이동호 4/13 21:07 초안 완료 ts 1776082022.830449), R101 상시 Pmax (Owner 홍유니→홍유니/황아인, Due 4/13→4/15, 황아인 4/13 14:51 재정비 완료 ts 1776059478.350029), R148 카테고리 구조 (Due 4/14→4/19, 황아인 4/13 TDL "금주 마무리" ts 1776048330.904909)
- **RC 위반: 2건 (RC19 + RC20 신규 패턴)**
  - RC19: R107 중국 세션 분석 완료 누락. 원인: 기존 thread(3/27 박서우 parent)의 old parent + new reply(4/13) 패턴이 2개월 채널 스캔에서 잡히지 않음. 고객사 채널 active task thread를 시트 Result `ts` 기반으로 매 audit 재독하는 방식 누락.
  - RC20: R151 번들 재고 버그 추이 데이터 공유가 "이번주 마감"에 잘못 포함. 시트 Status는 이미 Complete였으나 draft 작성 시 직전 EOD 리스트를 carry-over, 시트 E 컬럼 재조회 skip. 사용자 지적으로 발견.
  - 대응: (1) R107/R151 모두 Complete 확인 (2) 2개 정정 thread reply 발송 (ts 1776129198.136069, 1776129512.252359) (3) CLAUDE.md RC19+RC20 추가 (4) SKILL.md 섹션 3 + STEP 1 + STEP 6 Last-mile L.5 강화 (5) **CLAUDE.md + SKILL.md에 META-RULE "잘못 → 원인 → RC 생성 → 문서화" 최상위 명시 (사용자 요청)** (6) AUDIT_LOG 사용법에 RC19 추가
- 발송 ts: 첫 메시지 1776128758.323829 (C0ALWLLQFU7), 정정 reply 1 (R107) 1776129198.136069, 정정 reply 2 (R151) 1776129512.252359, 규칙 변경 공지 1776134749.918949
- **RC21 신규 추가 (사용자 지적)**: 프로모션 timing swap 공지에 "CLAUDE.md + skill 섹션 업데이트 완료" 포함 → 사용자가 "시스템 업데이트 관련 사항은 포함시키지 말아" 지적. Slack 메시지 composition rule로 격상: 금지 표현 목록(CLAUDE.md/skill/AUDIT_LOG/RC{N}/META-RULE) + 발송 전 grep 필수 반영
- **운영 규칙 변경 (4/14 11:40 자연님 재확정, RC8 trigger)**: 프로모션 시간대 SWAP
  - 기존(4/9): Open PST(KST 16:00) / Close EST(KST 13:00)
  - 신규(4/14): Open EST(KST 13:00) / Close PST(KST 16:00)
  - 이유: 기존 규칙이 "늦게 시작/일찍 종료" → 프로모 기간 최대화
  - 반영: CLAUDE.md `Promotion timing convention` 전면 재작성 + 변경 이력 섹션 추가, SKILL.md 2.1 섹션 swap, R142 Spring Matcha + R164 Primer 라이브 시간 → KST 13:00 업데이트, R73/R155 ABB 종료는 기존 합의 시간 유지

### 4/17 11:22 audit
- 시작 시각: 2026-04-17 11:22 KST (금요일)
- 종료 시각: 2026-04-17 12:02 KST
- 트리거: 사용자 `/tirtir-audit` slash command
- 주간 경계: 이번주 4/13(월)~4/19(일), 다음주 4/20(월)~4/26(일), 다다음주 4/27(월)~5/3(일)
- 4/16 15:30 audit 이후 약 20시간 경과
- 신규 발견:
  - **R168 SEO/GEO 일정 답변 → Complete** (이동호 4/16 17:02 고객사 채널 공식 공지 ts 1776672157.253829. "자사몰 SEO/AEO 제가 리딩 / 4월말 진단 → 5월 본격 진행" 답변 공식 전달. 자연님 수락 + GSC 권한 부여 약속)
  - **신규 17건 추가**:
    - **5/1 Glow Gift Festival 프로모션 9개 sub-task** (R174-R182, 자연님 4/16 18:15 기획안 전달 ts 1776676532.447389. 메인배너=고객사 제작 1728x677, 매드업=딜페이지+Klaviyo+광고+QA. 4/23 고객사 데드라인, 5/1 KST 13:00 라이브)
    - **R183 Spring Matcha 번들 제품 재고 수량 정정** (이동호, 4/18. 황아인 4/16 23:12 문의 → 박서우 4/17 물류업체 확인: US 236 / KR 96 / NL X / UK 2016)
    - **R184 Shopify 스케줄 앱 BFCM 2025 비활성화** (이동호, 4/17 today. 이동호 4/16 13:42 공지 + 자연님 허가)
    - **R185 자사몰 하단 영상 섹션 업로드 (말차 3건)** (이동호, 4/19. 자연님 4/15 18:19 요청)
    - **R186 레드쿠션 3D 모델링 업체 인계 담당자 지정** (황아인, 4/18. 자연님 4/15 17:48 요청, 영문 성함 필수)
    - **R187 제품별 USP 발굴 + Klaviyo/Meta/Pmax 에셋 자동 생성 (AI)** (김소형, 4/25. 홍유니 4/16 내부 요청)
    - **R188 Google 광고 캠페인 구조/KPI 리빌딩** (홍유니, 4/30. 이동호 4/16 분석 기반)
    - **R189 Red Cushion 상시캠페인 스위칭 준비** (홍유니, 4/24)
    - **R190 GSC 권한 부여 follow-up (TIRTIR)** (TIRTIR, 4/17 today. 자연님 4/16 17:02 약속)
  - **업데이트 3건**:
    - **R14 SEO 가이드 Due 4/14 → 4/30** (이동호 4/16 17:02 공식 공지 명시적 재설정)
    - **R17 카탈로그 피드 Result 업데이트** (홍유니 4/16 Merchant Center Shipping costs/limited 이슈 병행 요청)
    - **R172 Result 업데이트** (자연님 4/16 08:55 스코프 변경: "이후 프로모션부터" 기본 적용, ABB는 현 상태 유지)
  - **운영 규칙 변경 1건**: 자연님 4/16 08:55 신규 기본 적용: "이후 진행하는 프로모션부터 메타/구글 국내+일본 제외타깃 설정" → R180 Glow Gift 광고 세팅부터 반영
- **RC 위반 / 예방 1건**:
  - RC21 사전 감지: Draft 본문에 "RC8 trigger" 표현 포함 → grep 단계에서 즉시 제거 후 재grep CLEAN 확인 → 발송. 발송 전 예방 성공 (사용자 지적 없이 자체 발견)
- 시트 batchUpdate: 21건 (기존 4건 업데이트 + 신규 17건 append). 사후 R14/R17/R168/R172 + R174-R190 read-back verify 성공. Row 번호 misalignment 0건.
- 발송 ts: 1776393774.253639 (C0ALWLLQFU7 내부 채널, 확인 없이 자동 발송)
- **RC22 신규 추가 (사용자 지적으로 발견)**: ABB 종료 시간 표기 오류. R73 KST 13:00 / R155 KST 12:00로 표기됐으나 4/14 swap 규칙 적용 시 KST 16:00 / 15:00이 맞음. CLAUDE.md `기존 스케줄된 프로모션 처리` 섹션에 "ABB 4/17 KST 13:00 종료 기존 유지"가 ts 증거 없이 기재되어 있어 audit에서 그대로 채택. 대응: (1) R73 B/F/G 정정 → 16:00 (2) R155 B/F/G 정정 → 15:00 (3) Slack thread reply ts 1776393995.980489 발송 (4) CLAUDE.md `기존 스케줄된 프로모션 처리` 섹션 전면 재작성 (5) RC22 신규 추가 (6) SKILL.md 2.1 + 섹션 3 동기화
- 정정 발송 ts: 1776393995.980489 (방금 발송한 EOD message의 thread reply)

### 4/15 14:00 audit
- 시작 시각: 2026-04-15 14:00 KST (수요일)
- 종료 시각: 2026-04-15 14:35 KST
- 트리거: 사용자 `/tirtir-audit` slash command
- 주간 경계: 이번주 4/13(월)~4/19(일), 다음주 4/20(월)~4/26(일)
- 4/14 11:30 audit 이후 약 27시간 경과
- 세션 초기 이슈: bot token이 client 채널 접근 불가(channel_not_found) → 사용자에게 보고 후 MCP Slack 도구 로드하여 재개
- 신규 발견:
  - **R166 OOS 처리 (마스크핏 레드파운데이션 #37N)** → Complete 당일 (박서우 4/15 11:22 요청 ts 1776226925.112089 → 황아인 당일 공유 ts 1776228065.040379)
  - **R167 PDP 수정 (밀크트리오 A콘텐츠 교체)** → Processing (박서우 4/14 19:01 ts 1776163319.620759 / 이동호 4/14 19:57 내부 확인 / 홍유니 4/15 교체 진행)
  - **R168 SEO/GEO 일정 답변** → Processing (홍유니 4/15 12:25 내부 문의 ts 1776223506.936959 / 이동호 답변안 "4월 말 진단 → 5월 초 계획 → 단계적")
  - **R169 쇼피파이 데이터 보고서 지표 개선** → Processing (황아인 4/14 19:05 공유 ts 1776164250.961399 / 홍유니 4개 개선 요청: month단위/CVR·객단가/ROAS=0 수정/전환 기준)
  - **R170 중국 BOT 차단 앱 검토** → Not yet (박서우 4/15 R107 후속 질문 / 이동호 Kedra Shield + Blocky 제안)
  - **R87 Result 업데이트**: 오늘 Due, 이동호 4/13 초안 생성 ts 1776082022, TIRTIR 전달 예정
  - **R156 Due 변경**: 4/17 → 4/16 (김소형 4/14 EOD 명시적 재설정 — 17일 패밀리데이 반영)
- RC 위반: 0건
- 발송 ts: 1776230139.713929 (C0ALWLLQFU7 내부 채널)
- 운영 규칙 업데이트: SKILL.md STEP 7 "사용자 확인 없이 자동 발송" 원칙 명시 (사용자 요청으로 audit 결과 slack은 채널 확인 요청 생략)

### 4/13 13:00 audit
- 시작 시각: 2026-04-13 13:00 KST (월요일)
- 트리거: 사용자 "find all tasks and update the sheet after that give me the slack message draft"
- 주간 경계 검증: 이번주 = 4/13(월)~4/19(일), 다음주 = 4/20(월)~4/26(일)
- 4/8 11:30 audit 이후 약 5일 경과 (4/10 금요일 이후 주말 비활동)
- 채널 비활동: 양 채널 모두 4/10 22:16 이후 메시지 없음 (3일 비활동)
- 서브에이전트 6건 병렬 실행: 고객사채널 스캔, 내부채널 스캔, 봇/thread 검색, Processing 태스크 검증, Not yet 태스크 검증, RC5 ABB 분리 검증
- 신규 발견:
  - **BB 자료 5건 (R48-52) → Complete** (황아인 4/8 19:10 EOD 봇 thread reply: "메일로 수령 완료", ts 1775643042.697969)
  - **R71 (Matcha+ABB 소재) → Complete** (황아인 4/10 18:54: Meta 그룹별 세팅 완료)
  - **R136 (Spring Matcha RE 기획) → Complete** (황아인 4/10 18:54: 기획+고객사 컨펌+제작 요청 완료)
  - **R147 (브랜드 뉴스 업로드) → Complete** (자연님 4/10 12:05 확인 → 이동호 4/10 12:12 라이브)
  - **R124 (Google 결제수단) → Processing** (홍유니 4/10 11:56 시스템 오류 해소, 재요청)
  - **R145 (프로모션 종료 안내 페이지) → Processing** (이동호 4/10 11:47 이번 주 개발)
  - **R126 (세팅 체크리스트) → Processing** (TDL 배정 + 가이드 작성 진행 중)
  - **신규 8건**: R149 말차가격변경(Complete), R150 배송지연팝업, R151 번들버그데이터, R152 MYOB/GWP, R153 Reddit, R154 Klaviyo CRM, R155 ABB 광고 OFF
  - **RC5 분리**: R73 ABB 종료 → Shopify(이동호 R73) + Media(홍유니 R155)
  - **운영 규칙 변경**: 프로모션 종료시간 PST 23:00 → EST 00:00 (자연님 4/9 확정) → CLAUDE.md 즉시 반영
  - **P0 경고**: R87 레버 대시보드 안내 4/13 D-day이나 슬랙 근거 없음. R27 상시 소재 4/13 Due이나 슬랙 근거 없음
- RC 위반: 2건
  - RC9: 초기 batchUpdate row misalignment (파일라인번호≠시트행번호) → 즉시 감지+복구
  - RC16: AUDIT_LOG 미작성 → 사용자 지적 후 작성
