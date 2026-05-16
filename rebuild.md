# 🏆 뚝딱밸런스 (QR 제외 버전) — 재구축 스펙

> 이 문서는 클로드 코드(또는 비슷한 AI 코더)에게 통째로 던져서  
> **단일 `index.html` 파일** 하나로 동작하는 뚝딱밸런스 앱을 재현하기 위한 명세서입니다.  
> 빌드 도구·서버 없음. 브라우저에서 `index.html`만 열면 동작합니다.

---

## 0. 목표 한 줄

> 교실에서 학생들과 즉석 이상형 월드컵(8강/16강)을 돌리는 **단일 HTML 앱**.  
> 토너먼트 생성 → 1:1 투표 → 챔피언 결정 → 결과 카드 이미지 저장 까지.  
> **데이터는 localStorage**, 백엔드 없음.

---

## 1. 산출물 / 실행 방식

- **최종 산출물**: 프로젝트 루트의 `index.html` **단 한 파일**
  - CSS·JS 모두 인라인. ES Modules 사용 OK (`<script type="module">`).
  - 외부 의존: **CDN 3종만 허용** — Pretendard 폰트, Tailwind CDN(개발 편의용), html2canvas.
  - QR / 라이브러리 / 빌드 도구 **금지**.
- **실행**: 더블클릭 또는 `Live Server`로 `index.html` 열기. 모바일 사파리·크롬에서도 동작해야 함.
- **부가 파일(선택)**: 배포용 `vercel.json` (SPA fallback). 없어도 동작.

---

## 2. 디자인 시스템 — 세이지 그린 뉴모피즘

### 2.1 컬러 토큰 (`:root`)

```
--dd-bg: #E4E9DC          /* 세이지 크림 — 배경/카드/버튼 공통 표면 */
--dd-surface: #E4E9DC     /* 동일 (뉴모피즘의 핵심: 표면 = 배경) */
--dd-coral: #7A9B6E       /* 메인 강조 (세이지) */
--dd-coral-deep: #5C7A52  /* hover/포인트 (세이지 딥) */
--dd-mint: #B8C9A8        /* 보조 (라이트 세이지) */
--dd-text: #3D4A36        /* 본문 (딥 포레스트) */
--dd-text-sub: #6E7A65    /* 보조 텍스트 */
--dd-border: rgba(60,74,54,0.08)  /* 거의 안 보이는 미세 보더 */

--dd-sh-dark:  #BFC4B7    /* 뉴모피즘 그림자 톤 */
--dd-sh-light: #FAFFF2    /* 뉴모피즘 하이라이트 톤 */

--dd-radius: 22px
```

> 변수명에 `coral`이 남아있는 건 과거 파스텔 코랄 컬러웨이에서 토큰 키만 그대로 가져온 흔적이다.  
> 변수 *키*는 그대로 두고 *값*만 세이지로 채우는 게 핵심 — 그래야 기존 컴포넌트 CSS가 그대로 동작한다.

### 2.2 뉴모피즘 음영 셋

```
--dd-shadow:    8px 8px 18px var(--dd-sh-dark), -8px -8px 18px var(--dd-sh-light);
--dd-shadow-lg: 12px 12px 28px var(--dd-sh-dark), -12px -12px 28px var(--dd-sh-light);
--dd-shadow-sm: 4px 4px 10px var(--dd-sh-dark), -4px -4px 10px var(--dd-sh-light);
--dd-shadow-in:    inset 5px 5px 10px var(--dd-sh-dark), inset -5px -5px 10px var(--dd-sh-light);
--dd-shadow-in-sm: inset 3px 3px 6px  var(--dd-sh-dark), inset -3px -3px 6px  var(--dd-sh-light);
```

### 2.3 컴포넌트 규칙

| 상태 | 표현 |
|---|---|
| 일반 카드/버튼 | `box-shadow: var(--dd-shadow)` — 솟아오른 표면, 보더 없음 |
| 호버 | `--dd-shadow-lg` + `translateY(-2px ~ -4px)` |
| 클릭(active) | `--dd-shadow-in-sm` + `scale(0.97~0.98)` (눌림) |
| 인풋·트랙·매치 행 | `--dd-shadow-in-sm` (inset) |
| 선택됨(투표 카드) | `--dd-shadow-in` + 사지 글로우 링 `0 0 0 3px rgba(122,155,110,0.35)` |

### 2.4 폰트 / 형태

- **폰트**: Pretendard Variable (CDN)
  ```html
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/variable/pretendardvariable-dynamic-subset.min.css">
  ```
- **무드**: 부드럽고 둥글둥글 (radius 14~30px). 채도 낮은 그린·연갈색만 사용.  
  채도 높은 원색·딥블랙·흰배경 카드 **금지** (배경과 같은 세이지 표면을 사용).
- **타이포**: `font-feature-settings: 'ss01','ss02'`. 한글 keep-all.

### 2.5 핵심 CSS 클래스 (필수 작성)

| 클래스 | 역할 |
|---|---|
| `.dd-card` | 솟은 표면 카드. hover 시 더 솟음 |
| `.dd-btn-primary` | 세이지 그라데이션 + 흰 글자, hover 살짝 떠오름, active 눌림 |
| `.dd-btn-secondary` | surface 배경 + 세이지딥 글자, raised |
| `.dd-btn-ghost` | inset 표면, hover 시 raised로 전환 |
| `.dd-logo-badge` | 세이지 그라데이션 텍스트 (`background-clip: text`) |
| `.dd-empty` | inset 빈상태 박스 |
| `.dd-vs-card` | 투표 카드 (raised → hover lg → picked inset+ring → lost opacity 0.25) |
| `.dd-vs-badge` | "VS" 알약 (raised sm) |
| `.dd-progress-track` / `.dd-progress-fill` | inset 트랙 + 그라데이션 채움 + 글로우 |
| `.dd-champion-card` | `linear-gradient(135deg, #EEF3E5, #DCE5D0, #C9D6B8)` + 깊은 솟음 + 상단 흰 반사광 `::before` |
| `.dd-champion-ribbon` | 진한 세이지(`#5C7A52`) 알약, 흰 글자 "CHAMPION" |
| `.dd-champion-emoji` | 96px, `ddChampPop` 스프링 등장 |
| `.dd-round-card` | 라운드 블록, raised sm, hover raised |
| `.dd-match-row` | inset 행 |
| `.dd-modal` / `.dd-modal-backdrop` / `.dd-modal-body` | 백드롭 blur, 모달 raised lg, 스프링 등장 |
| `.dd-path-row` / `.dd-path-stage` / `.dd-path-vs` | 우승 경로 리스트 (점선 구분, 라운드 라벨은 raised sm 알약) |
| `.dd-fade-in` / `.dd-round-enter` | 페이지·라운드 전환 키프레임 (translateY+opacity, 0.25~0.35s) |

### 2.6 인풋·QR 박스 처리

- 텍스트 인풋·textarea·임시 박스: `background: var(--dd-surface); border: none; box-shadow: var(--dd-shadow-in-sm); color: var(--dd-text);`
- **QR 박스는 만들지 않는다** (이 빌드에서는 QR 기능 제외).

---

## 3. 정보 구조 / 라우팅

해시 라우터 1개. 페이지 5개:

| 경로 | 화면 |
|---|---|
| `#/` | 홈 (CTA + 내 토너먼트 목록) |
| `#/create` | 토너먼트 생성 (탭 2개: 템플릿 / 커스텀) |
| `#/play?t={id}` | 투표 진행 |
| `#/result?t={id}` | 결과 (챔피언 + 경로 + 대진표) |
| `#/settings` | 설정 (백업/복구/초기화) |

- `parseHash()` → `{ path, query }` 반환
- 라우트 미스매치 → `renderNotFound`
- `navigate(path)` 헬퍼로 `location.hash` 설정
- 라우터 진입 시 `<main id="app">` 비우고 `.dd-fade-in` 재시작 (offsetWidth 강제 리플로우)

---

## 4. 데이터 모델 / 스토리지

### 4.1 모델

```
Tournament = {
  id: string,           // crypto.randomUUID() 또는 fallback
  title: string,
  type: 'template' | 'custom',
  templateKey?: string, // template일 때만
  items: [{ id: 'i0'..'i15', emoji: string, text: string }],
  createdAt: number,    // Date.now()
}

Votes = {
  rounds: [{ size: 16|8|4|2, matches: [{a, b, winner: 'a'|'b'}] }],
  champion: Item,
  completedAt: number,
}

State = {
  tournaments: { [id]: Tournament },
  votes:       { [id]: Votes },
  results:     { [id]: {...} }, // 예비 슬롯 (이 빌드에서는 비움)
}
```

### 4.2 `storage` 객체 — **localStorage 격리 계층**

UI/로직 코드는 **절대 localStorage 직접 호출 금지**. 오직 이 객체만 통과.  
스토리지 키: `ddalkak-balance-v1`.

필수 메서드:

- `getTournament(id)` / `saveTournament(t)` / `listMyTournaments()` (최신순) / `deleteTournament(id)` (관련 votes·results 함께 삭제)
- `getVotes(id)` / `saveVotes(id, votes)`
- `exportBackup()` → JSON 문자열
- `importBackup(json)` → boolean (스키마 보정 후 덮어쓰기)
- `resetAll()` → key 삭제
- `getStats()` → `{ tournamentCount, completedCount, bytes }`

읽기 실패 시 기본 상태로 fallback. 쓰기 실패는 콘솔 경고 + `false` 리턴.

---

## 5. 내장 템플릿 3종

```js
templates = [
  { key:'good-neighborhood', title:'살기 좋은 동네', size:16,
    description:'우리 동네에 꼭 필요한 건 무엇일까?',
    items:[
      ['🎢','놀이공원'],['🌲','숲'],['🛒','마트'],['⚽','운동장'],
      ['🚌','대중교통'],['🚲','자전거'],['👫','친구'],['📚','도서관'],
      ['☀️','따뜻한 기후'],['🍂','뚜렷한 사계절'],['🍜','맛있는 먹거리'],['🥗','건강한 식단'],
      ['🏙️','고층빌딩'],['🏡','단독주택'],['🐶','반려동물'],['🧼','깨끗한 거리'],
    ]},
  { key:'best-snack', title:'우리 반 최고의 간식', size:16,
    description:'쉬는 시간에 제일 먹고 싶은 간식은?',
    items:[
      ['🍡','떡볶이'],['🍗','치킨'],['🍕','피자'],['🍔','햄버거'],
      ['🍜','라면'],['🍙','김밥'],['🍫','초콜릿'],['🍦','아이스크림'],
      ['🍪','과자'],['🍿','팝콘'],['🍬','사탕'],['🐻','젤리'],
      ['🍮','푸딩'],['🍰','케이크'],['🍩','도넛'],['🌸','마카롱'],
    ]},
  { key:'class-rules', title:'학급 규칙 우선순위', size:8,
    description:'우리 반에서 가장 중요한 규칙은?',
    items:[
      ['⏰','수업 시간 지키기'],['🤝','친구 존중하기'],['📝','숙제 열심히 하기'],['🧹','정리정돈'],
      ['👋','인사 잘하기'],['🧼','손 씻기'],['🚶','복도 걸어 다니기'],['👂','경청하기'],
    ]},
];
```

저장 시 `id`는 `'i'+index`, `createdAt = Date.now()`, `type = 'template'`, `templateKey = tpl.key`.

---

## 6. 화면별 명세

### 6.1 🏠 홈 `#/`

```
[로고 헤더: "🏆 뚝딱밸런스" + 부제 "우리 반이 뽑는 최고의 선택"]   [⚙️ 우상단]

[ ✨ 새 토너먼트 만들기 ]  ← dd-btn-primary, 풀폭, py-5

📚 내 토너먼트                                       총 N개
[ dd-card 행: 🏆? 제목 / (N강 · 날짜 · 완료?)        [투표] [🗑️] ]
...
(비었으면 dd-empty + "🌱 아직 만든 토너먼트가 없어요")
```

- ⚙️ 클릭 → `/settings`
- ✨ 클릭 → `/create`
- 행 좌측 → 제목·메타. 완료 시 제목 앞 🏆, "· 완료" 표기
- 행 우측 버튼: 미완료 → "투표"(secondary, `/play?t=id`) / 완료 → "결과"(primary, `/result?t=id`)
- 🗑️ → confirm 후 `storage.deleteTournament(id)` → 재렌더
- **QR 섹션은 만들지 않는다** (원본의 "📱 앱 바로가기 QR"은 이 빌드에서 제거)

### 6.2 ✨ 생성 `#/create`

상단: `← 뒤로` + 제목 "새 토너먼트 만들기"

탭 2개:
- **템플릿** (기본): 위 3종을 카드로 나열. 각 카드 우측 "사용하기" 클릭 → `saveFromTemplate(tpl)` → `/`
- **커스텀**:
  - 제목 인풋 (2자 이상 40자 이하)
  - 라운드 크기 선택: `[8강][16강]` 토글 — 활성: 세이지 그라데이션 + 흰 글자 + raised, 비활성: surface + inset
  - 후보 입력 행 N개: `[1] [이모지 ≤4] [이름 ≤20]`
    - 이모지 비우면 저장 시 `🎯`로 대체
  - 저장 버튼 (primary, full)
  - 검증: 제목 필수, 후보 전부 채움, 이름 중복 없음. 실패 시 `formMsg`에 빨간 톤(coral-deep) 텍스트.
- 탭 전환 시 커스텀 상태는 변수에 보존 (재진입해도 입력 유지). 라운드 크기 변경 시 기존 입력 보존하며 length 조정.

### 6.3 🏟️ 투표 `#/play?t={id}`

```
[← 뒤로]  부제(타이틀)
         16강 1/8

[━━━━━━━━━░░░] (진행 바, 채움 = currentIdx/totalMatches)

┌──────────────┐
│  이모지(64~80) │ ← dd-vs-card, 위
│  이름          │
└──────────────┘
       VS                ← dd-vs-badge
┌──────────────┐
│  이모지       │ ← 아래
│  이름        │
└──────────────┘

"마음에 드는 쪽을 탭하세요"
```

흐름:
1. items 셔플(Fisher-Yates) → `buildMatches` (2개씩 짝)
2. 매치 카드 클릭 → `locked=true`, picked는 `is-picked`, 반대는 `is-lost`, 550ms 후 다음 매치
3. 마지막 매치 끝 → `finishRound()`:
   - `rounds.push({ size, matches })`
   - winners.length === 1 → `storage.saveVotes(...)` → `/result?t=id`
   - 아니면 winners로 다음 라운드 매치 생성 (셔플 X — 경로 추적용)
4. ← 뒤로 클릭 시 confirm("진행 중인 투표가 사라져요")

예외:
- `tid` 없음/토너먼트 없음 → `renderPlayError("토너먼트를 찾을 수 없어요", ...)`
- items 개수가 2의 거듭제곱 아님 → 에러 화면

라운드 라벨: 2→결승, 4→4강, 8→8강, 16→16강, 32→32강.

### 6.4 🏆 결과 `#/result?t={id}`

```
[← 뒤로]  부제 "투표 결과" / 토너먼트 제목

┌─── 챔피언 카드 (세이지 그라데이션) ────┐
│        [ CHAMPION ]                    │ ← 진한 세이지 리본
│            🏆                          │ ← 96px 스프링 등장
│          이름                          │
│   YYYY.MM.DD 우승                      │
└────────────────────────────────────────┘

🛤️ 우승까지의 여정  (dd-card)
[ 16강 ]  🏆 챔피언 vs 🎯 상대(취소선)
[ 8강  ]  🏆 챔피언 vs 🎯 상대
[ 4강  ]  ...
[ 결승 ]  ...

📋 전체 대진표
(라운드별 dd-round-card 안에 매치 행들 — 승자 색=coral-deep, 패자 취소선)

[ 📸 이미지 저장 ]   ← primary, 풀폭
[ 🔗 링크 복사하기 ]  ← secondary
[ 🔁 다시 투표하기 ]  ← ghost, confirm("덮어써져요")
[ 홈으로 ]            ← ghost
```

**중요**: 원본은 `📸 이미지 저장`과 `📱 QR 공유`를 2칸 그리드로 묶었지만,  
이 빌드에서는 **QR 버튼 제거** → `📸 이미지 저장`만 풀폭 primary로 둔다.  
링크 복사·다시 투표·홈으로 버튼은 그대로 유지.

링크 복사: `location.origin + location.pathname + '#/play?t={id}'` → `navigator.clipboard.writeText`. 실패 시 `prompt()` fallback.

#### 📸 이미지 저장 (html2canvas)

```js
const card = buildShareCard(tournament, votes);  // 오프스크린 540px wide div
document.body.appendChild(card);
const canvas = await html2canvas(card, { backgroundColor:'#E4E9DC', scale:2, useCORS:true, logging:false });
canvas.toBlob(blob => download(blob, `ddalkak-${safeTitle}-${Date.now()}.png`));
document.body.removeChild(card);
```

공유 카드 레이아웃 (`buildShareCard`):
- 540px wide, 36px padding, `background: linear-gradient(135deg, #E4E9DC 0%, #D2DCC4 100%)`
- 상단: 세이지 딥(`#5C7A52`) 작은 로고 + 큰 타이틀
- 챔피언 박스: `linear-gradient(135deg, #EEF3E5, #DCE5D0, #C9D6B8)` + 뉴모피즘 shadow (`10px 10px 22px #BFC4B7, -10px -10px 22px #FAFFF2`) + CHAMPION 리본 + 108px 이모지 + 30px 이름 + 완료일
- "🛤️ 우승까지의 여정" 카드 (raised 6px 뉴모) — 라운드별 한 줄씩 점선 구분
- 푸터: "© 2026 무궁무진클래스 · 용쌤 · mumuclass.kr"
- html2canvas로 캡처 후 PNG 다운로드. 라이브러리 미로딩 시 안내 메시지.

### 6.5 ⚙️ 설정 `#/settings`

```
[← 뒤로]  ⚙️ 설정

📦 저장 현황 (dd-card)
  · 만든 토너먼트: N개
  · 투표 완료: N개
  · 사용 용량: KB

💾 백업 & 복구 (dd-card)
  · 안내문
  [📥 백업 다운로드 (.json)]  ← primary
  [📤 백업 불러오기]           ← secondary (label for hidden input)

⚠️ 위험한 작업 (dd-card)
  · 안내문
  [🔥 모든 데이터 초기화]      ← secondary

설정 메시지 영역 (flash)
"뚝딱밸런스 · L1"
```

- 백업 파일명: `ddalkak-balance-backup-YYYY-MM-DD.json`
- 불러오기: file input change → confirm → `FileReader.readAsText` → `storage.importBackup`
- 초기화: 2중 confirm 후 `storage.resetAll()` → 재페인트

---

## 7. 유틸 함수

```js
escapeHtml(s)       // <>&"' → 엔티티 (XSS 가드)
formatDate(ts)      // YYYY.MM.DD
newId()             // crypto.randomUUID() 또는 't_'+ts+'_'+rand
shuffle(arr)        // Fisher-Yates (불변)
buildMatches(items) // 짝수 길이 → [{a,b,winner:null}, ...]
roundLabel(size)    // 2→'결승' 4→'4강' 8→'8강' 16→'16강' 32→'32강' else 'N강'
getChampionPath(rounds, champion)  // 라운드별 [{ size, opponent }]
```

---

## 8. 푸터 (필수)

```html
<footer style="text-align:center; padding:24px 0; color:#64748b; font-size:12px;">
  © 2026 <a href="https://mumuclass.kr" style="color:#64748b;">무궁무진클래스</a> · 용쌤
</footer>
```

`footer a:hover { color: var(--dd-coral-deep) !important; }`

---

## 9. head 메타

```html
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<meta name="theme-color" content="#7A9B6E">
<title>🏆 뚝딱밸런스 — 우리 반이 뽑는 최고의 선택</title>
```

CDN 3종 (이 순서):
1. Pretendard variable
2. Tailwind CDN (`https://cdn.tailwindcss.com`)
3. html2canvas (`https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js`, defer)

---

## 10. 레이아웃 셸

```html
<body class="min-h-screen flex flex-col">
  <main id="app" class="flex-1 w-full max-w-[640px] mx-auto px-4 py-6"></main>
  <footer>…</footer>
  <script type="module">…</script>
</body>
```

---

## 11. ❌ 이 빌드에서 만들지 말 것 (QR 제외 사항)

원본 앱의 다음 요소는 **모두 빼고 만든다**:

1. 홈 화면의 "📲 학생 초대 QR" 섹션 (`#inviteQR`, `renderHomeInviteQR`)
2. 결과 화면의 "📱 QR 공유" 버튼 (`#btnQR`, `showQRModal`)
3. QR 이미지 URL 빌더 (`qrImgUrl`, 외부 `api.qrserver.com` 호출)
4. QR URL 카피용 `.dd-qr-url` CSS
5. QR 모달 (`showQRModal` 함수 자체)

대신:
- 홈은 [로고] + [✨ 새 토너먼트 만들기 CTA] + [📚 내 토너먼트 목록] 만 둔다.
- 결과 페이지의 액션 영역은 `📸 이미지 저장 (풀폭 primary) → 🔗 링크 복사 → 🔁 다시 투표 → 홈으로` 순서.

---

## 12. ✅ 수용 기준 (체크리스트)

- [ ] `index.html` 더블클릭으로 동작. 빌드 명령 없음.
- [ ] 홈에서 ✨ 클릭 → 템플릿 1개 사용 → 자동 홈 복귀 → 목록에 한 줄 추가.
- [ ] "투표" 클릭 → 16강 8경기 진행 → 진행 바·1/8 카운터 정상.
- [ ] 결승 끝 → 결과 화면. 챔피언 카드 스프링 등장. 우승 경로에 4행(16→8→4→결승) 표시.
- [ ] 📸 이미지 저장 → PNG 다운로드 (세이지 톤, html2canvas).
- [ ] 🔗 링크 복사 → 클립보드에 `…#/play?t=...` 들어감. 클립보드 권한 거부 시 `prompt()` 폴백.
- [ ] 🔁 다시 투표 → confirm → 같은 토너먼트 재진행.
- [ ] 설정 → 백업 다운로드 / 불러오기 / 초기화 모두 동작.
- [ ] localStorage 키: `ddalkak-balance-v1` 하나.
- [ ] 호버 시 카드/버튼이 살짝 떠오르고 그림자가 깊어진다 (뉴모피즘 raised → raised-lg).
- [ ] 채도 높은 색 / 흰 카드 / QR 흔적 없음. 전체가 세이지 그린 + 크림 톤.

---

## 13. 빌드 순서 제안 (2시간 안에 끝내기)

1. **스켈레톤** (15분): head/CDN/style 토큰·뉴모 음영·기본 컴포넌트 CSS / `<main>` / 푸터 / 라우터·`navigate` / `storage` 객체.
2. **홈 + 생성(템플릿만)** (30분): 템플릿 3종 카드 → 저장 → 홈 목록 표시.
3. **투표** (30분): 셔플 → 매치 카드 → picked 애니메이션 → 라운드 진행 → 결승 후 saveVotes.
4. **결과** (25분): 챔피언 카드 + 우승 경로 + 라운드별 대진표.
5. **이미지 저장 + 링크 복사** (15분): `buildShareCard` + html2canvas + 클립보드.
6. **설정 + 커스텀 탭** (15분): 백업/복구/초기화, 커스텀 폼.

> 작동 후에야 디자인 마감(반응형 폰트 사이즈, 애니메이션 타이밍 미세 조정)을 한다.

---

## 14. 참고: 협업 톤

- 한국어 응답.
- "왜 이렇게 했는지" 한 줄 설명. 칭찬·해설 짧게.
- 라이브러리 추가는 사전 보고 (이 빌드는 html2canvas + Pretendard + Tailwind CDN 외 금지).
- 사용자(용쌤)는 18년차 초등교사. 교실 5초 사용성이 최우선.
