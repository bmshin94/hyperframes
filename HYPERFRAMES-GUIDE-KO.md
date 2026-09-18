# HyperFrames 한글 정리 노트

> HTML로 영상을 만드는 오픈소스 프레임워크 — 분석 및 활용 가이드
>
> - 원본 저장소: https://github.com/heygen-com/hyperframes
> - 이 저장소(포크): https://github.com/bmshin94/hyperframes
> - 공식 문서: https://hyperframes.heygen.com/introduction
> - 플레이그라운드: https://www.hyperframes.dev
> - 카탈로그: https://hyperframes.heygen.com/catalog/blocks/data-chart
> - 커뮤니티(Discord): https://discord.gg/EbK98HBPdk
> - 라이선스: Apache 2.0 (상업적 이용 자유)

---

## 1. 한 줄 요약

**영상을 손으로 편집하는 게 아니라, HTML로 써서 만드는 도구.**

슬로건: `Write HTML. Render video. Built for agents.`
만든 곳은 HeyGen(AI 아바타 영상 회사)이고, 완전 오픈소스다.

### 동작 방식

```html
<div id="stage" data-composition-id="launch" data-width="1920" data-height="1080">
  <video class="clip" data-start="0" data-duration="6" src="intro.mp4"></video>
  <h1 class="clip" data-start="1" data-duration="4">Launch day</h1>
  <audio data-start="0" data-duration="6" data-volume="0.5" src="music.wav"></audio>
</div>
```

- `data-start` → 몇 초에 등장
- `data-duration` → 몇 초 동안 유지
- `data-track-index` → 몇 번 트랙
- `class="clip"` → 타임라인이 관리하는 요소

즉 **편집 프로그램의 타임라인을 HTML 속성으로 옮겨놓은 것**이다.
렌더 시 헤드리스 크롬이 한 프레임씩 캡처 → FFmpeg으로 인코딩 → MP4 출력.

### 핵심 철학 3가지

| 철학                        | 의미                                                                    |
| --------------------------- | ----------------------------------------------------------------------- |
| **HTML 네이티브**           | React 불필요, 빌드 스텝 없음. `index.html`이 그대로 재생됨              |
| **결정론적(Deterministic)** | 같은 입력 = 항상 같은 영상. `Date.now()`/시드 없는 `Math.random()` 금지 |
| **에이전트 친화적**         | 애초에 AI가 쓰라고 설계됨. CLI가 비대화형(non-interactive) 기본         |

---

## 2. 저장소 구조

```
packages/     14개 패키지 (엔진 본체)
  cli/              hyperframes CLI (명령어 51개)
  core/             타입, 파서, 린터, 런타임, 프레임 어댑터
  engine/           Puppeteer 기반 프레임 캡처 엔진
  producer/         캡처 + 인코딩 + 오디오 믹싱 파이프라인
  player/           <hyperframes-player> 웹컴포넌트 (의존성 0)
  sdk/              헤드리스, 프레임워크 중립 컴포지션 편집 엔진
  studio/           브라우저 편집기 UI (webmcp 툴 레이어 포함)
  aws-lambda/       AWS Lambda 분산 렌더링
  gcp-cloud-run/    GCP Cloud Run 렌더링
  shader-transitions/  WebGL 셰이더 트랜지션

registry/     재료 창고
  blocks/           180개 완성된 씬 (차트레이스, 카메라 돌리줌, 캐러셀 등)
  components/       221개 이펙트/스니펫
  → 총 401개

skills/       AI 에이전트용 스킬 20개
docs/         Mintlify 공식 문서 사이트 소스
examples/     AWS Lambda / GCP Cloud Run / k8s 배포 예제
```

---

## 3. 설치 및 사용법

### 준비물

- **Node.js 22 이상**
- **FFmpeg** (`brew install ffmpeg` / `winget install ffmpeg`)

### 방법 A — 바로 써보기 (권장)

```bash
npx hyperframes init my-video    # 프로젝트 생성
cd my-video
npx hyperframes preview          # 브라우저 실시간 미리보기
npx hyperframes render           # MP4 렌더링
```

`init` 하면 `index.html` + `CLAUDE.md`/`AGENTS.md`(AI용 설명서)가 생성된다.

### 방법 B — AI 에이전트와 함께

```bash
npx hyperframes skills update                  # 핵심 스킬만 (권장)
npx skills add heygen-com/hyperframes          # 터미널 인터랙티브 선택
npx skills add heygen-com/hyperframes --all    # 20개 전부
```

설치 후 프롬프트 예시:

> "`/hyperframes` 써서 10초짜리 제품 인트로 만들어줘. 타이틀 페이드인, 배경 영상, 잔잔한 BGM으로."

### 방법 C — 이 저장소 자체를 개발

```bash
bun install      # npm/pnpm 금지, bun 사용
bun run build
bun run test
bun run dev      # 스튜디오 실행
```

### 작성 후 필수 검사

```bash
npx hyperframes lint    # HTML 구조 정적 검사
npx hyperframes check   # 헤드리스 크롬 검사 (런타임 에러/레이아웃/모션/WCAG 명암비)
```

두 검사를 통과해야 미리보기/완료로 간주한다.

### 린트 & 포맷 (저장소 개발 시)

**oxlint / oxfmt 사용** (eslint/prettier/biome 아님)

```bash
bunx oxlint <files>
bunx oxfmt <files>
bunx oxfmt --check <files>
```

---

## 4. 플러그인인가? 스킬인가? MCP인가?

**본체는 npm 패키지(CLI + 프레임워크)이고, 스킬/플러그인은 포장지다.**

| 층       | 정체                                                 | 위치                                                   |
| -------- | ---------------------------------------------------- | ------------------------------------------------------ |
| 본체     | npm 라이브러리 + CLI (명령어 51개)                   | `packages/`                                            |
| 스킬     | AI용 교과서 20권 (Markdown)                          | `skills/`                                              |
| 플러그인 | Claude Code / Codex / Cursor 마켓플레이스 매니페스트 | `.claude-plugin/`, `.codex-plugin/`, `.cursor-plugin/` |
| MCP      | **MCP 서버 아님**                                    | —                                                      |

### MCP 관련 정확한 사실

- `@modelcontextprotocol` 의존성 없음, `mcpServers` 설정 없음 → **MCP 서버를 제공하지 않는다**
- `packages/studio/src/webmcp/` → 스튜디오 **내부**에서 에이전트가 쓰는 툴 레이어(WebMCP)
- `/figma` 스킬 → 외부 Figma MCP를 **소비**하는 쪽

### 개념 3초 구분

- **스킬** = AI에게 주는 설명서 (필요할 때만 로드 → 토큰 절약)
- **MCP** = AI가 호출하는 외부 서버 (상시 연결 필요)
- **플러그인** = 위 둘을 한 번에 설치해주는 포장

HyperFrames는 **스킬 방식**을 선택했다. `agentDirs.generated.ts` 기준
Claude Code, Codex, Cursor, Amp, Gemini 등 **30개 이상 에이전트**에 설치 가능 → 특정 AI에 종속되지 않는다.

### 스킬 구조 (라우터 패턴)

```
/hyperframes (라우터)                  ← 입구, 의도 파악
    ↓ 필요한 것만 온디맨드 설치
/product-launch-video 등 (워크플로우 10개)
    ↓ 참조
/hyperframes-core, -animation 등 (도메인 스킬 9개)  ← 재사용 부품
```

**제작 워크플로우**

| 상황                           | 스킬                       |
| ------------------------------ | -------------------------- |
| 웹사이트 URL → 제품 홍보영상   | `/product-launch-video`    |
| 텍스트 → 얼굴 없는 설명 영상   | `/faceless-explainer`      |
| GitHub PR → 변경사항 설명 영상 | `/pr-to-video`             |
| 말하는 영상 → 자막 입히기      | `/embedded-captions`       |
| 인터뷰 영상 → 그래픽 오버레이  | `/talking-head-recut`      |
| 10초 내외 모션 그래픽          | `/motion-graphics`         |
| 음악 → 비트 싱크 영상          | `/music-to-video`          |
| 프레젠테이션/피치덱            | `/slideshow`               |
| 그 외 전부                     | `/general-video`           |
| Remotion(React) 이식           | `/remotion-to-hyperframes` |

**도메인 스킬:** `-core`(문법), `-animation`(모션), `-keyframes`, `-creative`(디자인),
`/media-use`(BGM·SFX·이미지·TTS 조달), `-audio`(믹싱), `-cli`, `-registry`, `/figma`

---

## 5. API 토큰이 필요한가?

**기본 사용은 토큰 0개, 완전 무료.**

`.env.example` 첫 줄:

> `# No environment variables required for basic usage.`

HTML 작성 → 렌더 → MP4 출력까지 전부 로컬에서 동작한다.

### 선택적 토큰 (고급 기능용)

| 토큰                                     | 용도                                           | 필수 여부 |
| ---------------------------------------- | ---------------------------------------------- | --------- |
| `GEMINI_API_KEY`                         | 웹사이트 캡처 시 이미지 자동 캡션 (~$0.001/장) | 선택      |
| `ELEVENLABS_API_KEY`                     | AI 성우 음성 생성(TTS)                         | 선택      |
| `OPENROUTER_API_KEY`                     | 이미지/음악 AI 생성                            | 선택      |
| `FIGMA_TOKEN`                            | 피그마 디자인 가져오기                         | 선택      |
| `HEYGEN_API_KEY` / `HYPERFRAMES_API_KEY` | HeyGen 클라우드 렌더                           | 선택      |

**정리**

- 소스를 직접 준비해서 로컬 렌더 → **0원, 토큰 0개**
- AI가 음악/성우/이미지까지 생성 → 해당 서비스 요금 (HyperFrames 요금이 아님)
- **HyperFrames 자체는 렌더 과금이 없다** (Apache 2.0)

---

## 6. 왜 GitHub에서 주목받는가

> 참고: 스타 개수는 직접 확인하지 않았음. 아래는 저장소 내용 기반 분석.

1. **Remotion 라이선스 탈출구** — Remotion은 Source-available이라 규모가 커지면 유료.
   HyperFrames는 Apache 2.0. README에 비교표까지 명시되어 있다.
2. **AI 에이전트 시대 타이밍** — React(JSX)보다 HTML이 AI가 훨씬 잘 쓴다.
   `Built for agents`를 전제로 설계된 영상 프레임워크.
3. **빌드 스텝 0** — `index.html`을 그대로 열면 재생된다. 진입장벽이 매우 낮다.
4. **실전 검증** — HeyGen 프로덕션 사용 + `ADOPTERS.md`에 tldraw, TanStack 등재.
5. **퍼주기 전략** — 블록 180 + 컴포넌트 221 = **401개**를 전부 무료 공개.
6. **개발 속도** — 커밋 로그 기준 PR 번호가 #4038까지 진행. 기능이 매일 추가되는 중.

---

## 7. 로컬 에이전트 구축에 도움이 되는가

### A) 영상 제작 에이전트를 만들 때 — 바로 활용 가능

- CLI가 **비대화형 기본** → AI가 호출하기 좋은 구조
- **결정론적** → 테스트/검증 가능
- `lint`, `check` 내장 → 에이전트가 스스로 결과를 검증 가능
- `@hyperframes/sdk` = 헤드리스·프레임워크 중립 편집 엔진

### B) 에이전트 설계법을 배울 때 — 최고의 레퍼런스

- **"라우터 + 온디맨드 로딩 + 원자적 도메인 스킬"** 패턴 (컨텍스트 낭비 없이 전문성 주입)
- `scripts/check-skill-mirror.mjs` → 스킬 문서 동기화를 CI로 강제 (문서 부패 방지)
- `packages/studio/src/webmcp/` → 브라우저 UI 안에 에이전트 툴 심는 방법
- `CLAUDE.md`의 "Skill catalog maintenance" → 스킬이 늘어날 때의 관리 규율

---

## 8. 수익화 아이디어

### 8-1. 왜 돈이 되는 구조인가

**무기 1 — 렌더 원가가 사실상 0원**
영상 SaaS의 최대 원가인 렌더링 비용이 없다. (경쟁 상용 API는 영상당 과금)

**무기 2 — 1개 디자인 → 1만 개 영상 (핵심!)**

```html
<html
  data-composition-variables='[
  {"id":"title","type":"string","default":"Pro"},
  {"id":"accent","type":"color","default":"#6c5ce7"},
  {"id":"logo","type":"string","default":"assets/logo.svg"}
]'
>
  <h1 data-var-text="title">Welcome</h1>
</html>
```

```bash
npx hyperframes render --batch rows.json --strict-variables --output "renders/{name}.mp4"
```

이것이 **개인화 영상 대량생산(Personalized Video at Scale)**.
AWS Lambda의 `lambda render-batch`로 병렬 처리도 가능하다.

### 8-2. 원가 추정 (30초 1080p 영상 1개)

> 주의: 아래는 일반 AWS/API 단가 기준 **추정치**. 실제 1개를 렌더해 검증 필요.

| 항목                | 비용                               | 비고                 |
| ------------------- | ---------------------------------- | -------------------- |
| 렌더링(Lambda)      | 약 10~50원                         | Chrome 캡처 + FFmpeg |
| 스토리지/CDN        | 약 5~20원                          | S3 + 전송            |
| AI 음성(TTS)        | 약 150~300원                       | **선택**             |
| AI 이미지 생성      | 약 50~200원                        | **선택**             |
| 이미지 캡션(Gemini) | 약 1.5원                           | 장당 $0.001          |
| **합계**            | **AI 없이 ~50원 / AI 포함 ~500원** |                      |

**전략 3가지**

1. AI 음성이 원가의 대부분 → 자막+BGM만 쓰면 원가가 거의 0 (쇼츠는 무음 시청이 다수)
2. **BYOK(Bring Your Own Key)** — 사용자가 자기 API 키 입력 → 내 원가 0, 투명성 확보
3. 가격 예시: 무료 3개(워터마크) / ₩19,000 50개 / ₩49,000 200개+API / ₩290,000 화이트라벨

### 8-3. 실제 사례 (ADOPTERS.md 기반 — 이미 이걸로 장사 중인 곳들)

| 회사                                | 사용 방식                                     | 시사점                                      |
| ----------------------------------- | --------------------------------------------- | ------------------------------------------- |
| **Typeframe** (한국, @kiyeonjeon21) | 말하는 영상 → 단어별 타이핑 자막 MP4 내보내기 | 기능 하나로 서비스 성립. 개인/소규모도 가능 |
| **reap.video**                      | AI 영상 클리핑/편집                           | 크리에이터 시장                             |
| **PandaStudio**                     | 데스크탑 편집기의 모션그래픽/자막 엔진        | 내 앱의 내부 엔진으로 활용                  |
| **Mini Course Generator**           | 강의 제작 워크플로우의 영상 생성              | 에듀테크 결합                               |
| **OpenMAIC** (칭화대)               | AI 교실 → 원클릭 MP4 내보내기                 | 교육/연구                                   |
| **tldraw**                          | PR 설명 영상 자동 생성                        | 자체 마케팅 활용                            |

**인사이트:** "영상 편집기 전체"를 만든 곳은 없다. 모두 **기능 한 개**로 승부한다.
그리고 "내보내기(Export) 엔진" 수요가 크다.

### 8-4. 아이디어 목록 (현실성 순)

#### TIER 1 — 지금 바로

**① 개인화 연말결산/리포트 영상 (최고 추천)**
CSV 업로드 → 고객 1000명에게 각각 다른 영상.
변수+배치 기능이 정확히 이걸 위해 존재한다. B2B라 단가가 높고,
경쟁사는 영상당 과금이라 가격 경쟁에서 압도적이다.
타겟: 헬스장, 학원, 쇼핑몰 VIP, 보험/은행, 멤버십.

**② 상품 URL → 쇼츠 자동 생성기**
`hyperframes capture <URL>` + `/product-launch-video` 스킬이 이미 워크플로우 완성품.
타겟: 스마트스토어/쿠팡 셀러(50만+), 소상공인, 부동산 중개사.
현재 외주 단가 건당 5~15만원 → 월 1.9만원 구독으로 가격 파괴.

**③ 유튜브/릴스 자동화 채널**
`/faceless-explainer` + 자막·BGM만 → 영상당 원가 약 50원.
수익: 애드센스 + 제휴 마케팅 + 협찬. 기술 난이도는 낮지만 **기획력이 전부**이고 경쟁이 심하다.

#### TIER 2 — 준비 후

**④ 템플릿/블록 팩 판매**
한국 시장 겨냥: 예능식 노란 테두리 자막 팩, 교회/병원/학원/음식점 팩 등.
판매처: Gumroad, 크몽, Lemon Squeezy. 한 번 만들면 계속 팔리는 자산형 수익.

**⑤ 개발자용 "PR → 릴리즈 영상" 봇**
`/pr-to-video` 스킬이 완성품이고 tldraw가 실제 사용 중.
GitHub App으로 만들어 팀당 월 $29~99. 영어권 진입 가능, 경쟁 거의 없음.

**⑥ "영상 내보내기 엔진" API 판매**
PandaStudio/OpenMAIC 방식. 기존 웹앱에 "MP4 내보내기" 기능을 붙여주는 B2B.
타겟: 노션 클론, 대시보드 SaaS, 퀴즈/설문 서비스, 웹 편집기.

#### TIER 3 — 큰 그림

- **외주/컨설팅** — 기업 영상 자동화 파이프라인 구축. 건당 500만~3000만원. 종잣돈 확보에 유리
- **교육 콘텐츠** — 인프런/유데미/클래스101. 현재 한국어 자료가 거의 없어 선점 기회
- **니치 버티컬 SaaS** — 부동산 시세 영상, 주식/코인 시황 영상(`bar-chart-race` 블록), 헬스 기록 영상

### 8-5. 기능 ↔ 비즈니스 매핑

| 저장소 기능                              | 가능한 사업                  |
| ---------------------------------------- | ---------------------------- |
| `data-composition-variables` + `--batch` | 개인화 영상 대량생산         |
| `hyperframes capture <URL>`              | URL → 홍보영상 자동화        |
| `packages/aws-lambda` / `gcp-cloud-run`  | SaaS 백엔드 (배포 예제 완비) |
| `@hyperframes/player` (의존성 0)         | 웹사이트 임베드 플레이어     |
| `@hyperframes/sdk` (프레임워크 중립)     | 내 앱에 편집 기능 내장       |
| `registry/` 401개 블록                   | 템플릿 팩 판매 기반          |
| `skills/` 20개                           | AI 자동 제작 = 인건비 절감   |
| `--strict-variables`                     | 고객 데이터 검증 (B2B 필수)  |

### 8-6. 리스크 및 주의사항

1. **상표(Trademark)** — LICENSE 6조: 라이선스는 상표 사용권을 주지 않는다.
   "HyperFrames"를 서비스명/로고로 쓰면 안 된다. `Powered by HyperFrames` 수준의 사실 표기는 무방.
   **코드 사용 자체는 100% 자유.**
2. **진짜 경쟁력은 디자인** — 프레임워크는 누구나 쓴다. 템플릿 퀄리티에서 갈린다.
3. **렌더 비용 관리** — 무제한 플랜은 어뷰징 위험. 큐 + 동시 실행 제한 + 월 한도 필수.
   문서도 `--batch-concurrency`를 천천히 올리라고 경고한다.
4. **개발 속도가 매우 빠름** — 버전 고정(pin) 후 점진적으로 따라가야 서비스가 안정적이다.
5. **안 맞는 영역** — 실사 촬영본 컷편집, 인물 리터칭은 이 도구의 영역이 아니다.
   디자인/모션그래픽/데이터 영상에 집중할 것.

### 8-7. 추천 실행 순서

```
1단계 (2주)     템플릿 팩 제작 → 크몽/Gumroad 판매
                 → 검증 + 첫 매출 + 템플릿 자산 확보
        ↓
2단계 (1~2개월)  그 템플릿으로 "URL → 쇼츠" 웹서비스 오픈
                 → 1단계 자산 재활용, 개발 최소화
        ↓
3단계 (3~6개월)  B2B 개인화 영상으로 확장
                 → 단가 10배, 여기가 실질 수익 구간
```

앞 단계의 산출물이 다음 단계의 상품/레퍼런스가 되므로 버려지는 작업이 없다.

**2주 실행 플랜**

| 일차      | 할 일                                            |
| --------- | ------------------------------------------------ |
| Day 1     | `npx hyperframes init` → 5초 영상 렌더까지 성공  |
| Day 2-3   | `--batch`로 영상 10개 일괄 생성 (핵심 기능 체감) |
| Day 4-5   | 실제 원가 측정 (렌더 시간 × 서버 비용)           |
| Day 6-7   | 한국형 템플릿 3개 제작 (자막 팩 추천)            |
| Day 8-10  | 크몽/Gumroad 등록, 가격 테스트                   |
| Day 11-14 | 반응 확인 후 2단계 진행 여부 결정                |

---

## 9. React / PHP로 만들 수 있는가

**핵심 원리**

> 컴포지션 = 그냥 HTML 텍스트 파일 → **어떤 언어로든 생성 가능**
> 렌더 = 헤드리스 크롬 + FFmpeg → **Node.js 환경 필요**

### PHP (가능)

```php
<?php
// 1. DB에서 데이터를 가져와 HTML 컴포지션 생성
$html = renderTemplate($product);
file_put_contents('/tmp/video/index.html', $html);

// 2. Node CLI 호출로 렌더 (PHP는 명령만 내림)
exec('npx hyperframes render /tmp/video/index.html -o out.mp4');
```

라라벨 큐에 렌더 작업을 쌓고 Node 워커가 처리 → S3 업로드가 일반적인 패턴.

### React (3가지 방식)

1. **재생만** — `@hyperframes/player`는 의존성 0의 웹컴포넌트라 프레임워크 무관
   ```jsx
   <hyperframes-player src="/my-video/index.html" controls />
   ```
2. **컴포지션 생성** — `renderToStaticMarkup()`으로 HTML 문자열을 만들어 저장.
   단, `data-start` / `data-duration` / `class="clip"` 규칙은 반드시 지켜야 한다.
3. **SDK 편집** — `@hyperframes/sdk`로 프로그래밍 방식 조립 (Next.js API 라우트에 적합)

### 불가능한 것

브라우저 안에서 직접 MP4 출력 → 렌더 엔진이 Puppeteer + FFmpeg이라 **서버/로컬 Node 환경 필수**.

### 실무 아키텍처

```
[React/Vue/PHP 프론트]  → HTML 컴포지션 생성
          ↓
[Node 렌더 서버 or AWS Lambda] → MP4 렌더
          ↓
[S3/CDN] → 사용자 전달
```

`packages/aws-lambda`, `packages/gcp-cloud-run`, `examples/k8s-jobs`에 배포 예제가 준비되어 있다.

---

## 10. 요약표

| 질문      | 답                                                             |
| --------- | -------------------------------------------------------------- |
| 설치      | `npx hyperframes init` — Node 22 + FFmpeg만 있으면 끝          |
| 정체      | npm CLI 본체 + AI 스킬 20개 (MCP 서버 아님, 플러그인은 포장지) |
| 토큰      | 기본 기능 0개. AI 음성/이미지 사용 시에만 선택적               |
| 인기 이유 | Remotion 라이선스 탈출 + AI 시대 타이밍 + HeyGen 실전 검증     |
| 에이전트  | 활용도 좋고, 설계 학습용 레퍼런스로도 최상급                   |
| 수익화    | 개인화 영상 대량생산(B2B)이 최유망, 템플릿 판매로 시작 추천    |
| React/PHP | 생성은 자유, 렌더는 Node가 담당                                |

---

## 11. 다음 단계

- [ ] `npx hyperframes init`으로 5초 영상 하나 렌더까지 성공시키기
- [ ] `--batch`로 개인화 영상 10개 일괄 생성해보기
- [ ] 실제 렌더 원가 측정 (추정치 검증)
- [ ] 한국형 자막 템플릿 설계
- [ ] URL → 쇼츠 서비스 아키텍처 설계

---

## 참고 링크

- 원본 저장소: https://github.com/heygen-com/hyperframes
- 이 저장소(포크): https://github.com/bmshin94/hyperframes
- 문서: https://hyperframes.heygen.com/introduction
- 퀵스타트: https://hyperframes.heygen.com/quickstart
- 쇼케이스: https://hyperframes.heygen.com/showcase
- 변수/템플릿 개념: https://hyperframes.heygen.com/concepts/variables
- AWS Lambda 렌더링: https://hyperframes.heygen.com/deploy/aws-lambda
- Remotion 비교 가이드: https://hyperframes.heygen.com/guides/hyperframes-vs-remotion
- 플레이그라운드: https://www.hyperframes.dev
- frame.md 디자인 템플릿: https://www.hyperframes.dev/design
- Discord: https://discord.gg/EbK98HBPdk
