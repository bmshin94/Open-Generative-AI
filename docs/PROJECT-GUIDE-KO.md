# Open Generative AI — 분석 · 설치 · 수익화 가이드 (한국어)

> 이 문서는 `Open-Generative-AI` 저장소를 직접 분석하고, 설치·사용법과 PHP 기반 수익화 방향까지
> 정리한 한국어 종합 가이드입니다.

---

## 📌 관련 저장소 & 링크

### 이 프로젝트

| 구분 | 주소 |
|---|---|
| **내 저장소 (fork)** | https://github.com/bmshin94/Open-Generative-AI |
| **원본 저장소 (upstream)** | https://github.com/Anil-matcha/Open-Generative-AI |
| **데스크톱 앱 다운로드** | https://github.com/Anil-matcha/Open-Generative-AI/releases |
| **호스팅 버전 (설치 불필요)** | https://muapi.ai/open-generative-ai |
| **커뮤니티 (Discord)** | https://discord.gg/tANKJkHck |

### 서브모듈 3개 (`.gitmodules`)

| 경로 | 주소 |
|---|---|
| `packages/Vibe-Workflow` | https://github.com/SamurAIGPT/Vibe-Workflow |
| `packages/Open-Poe-AI` | https://github.com/Anil-matcha/Open-Poe-AI |
| `packages/Open-AI-Design-Agent` | https://github.com/Anil-matcha/Open-AI-Design-Agent |

### 외부 의존 서비스 / 엔진

| 이름 | 주소 | 역할 |
|---|---|---|
| **MuAPI** | https://muapi.ai | 실제 AI 모델을 돌려주는 유료 API 게이트웨이 |
| MuAPI 키 발급 | https://muapi.ai/access-keys | API 키(값) 발급 |
| stable-diffusion.cpp | https://github.com/leejet/stable-diffusion.cpp | 내장 로컬 추론 엔진 (sd.cpp) |
| Wan2GP | https://github.com/deepbeepmeep/Wan2GP | 별도 GPU 서버용 로컬 추론 엔진 |

### upstream 연결해두기 (원본 업데이트 받기)

```bash
git remote add upstream https://github.com/Anil-matcha/Open-Generative-AI.git
git fetch upstream
git merge upstream/main      # 필요할 때만
```

---

## 1. 이게 뭐하는 프로젝트야?

**한 줄 요약: 400개 이상의 AI 이미지/영상 생성 모델을 하나의 화면에서 쓰게 해주는 오픈소스 스튜디오 앱.**

### 🍕 배달 앱 비유

| 비유 | 실제 |
|---|---|
| 배달 앱 (메뉴판 + 주문 버튼) | **이 저장소** — UI만 들어있음 |
| 음식점 (실제 조리) | Kling, Veo, Flux 등 **외부 AI 모델** |
| 음식값 / 배달비 | **MuAPI 크레딧 (유료)** |

### ⚠️ 가장 중요한 사실: 이 폴더 안에 AI는 없다

`packages/studio/src/muapi.js:66` 기준 실제 호출 흐름:

```
내 화면 (이 앱)
   ↓  POST /api/v1/{모델엔드포인트}   header: x-api-key
api.muapi.ai  (유료 게이트웨이)
   ↓
실제 모델 (Kling / Veo / Flux / Seedream ...)
```

`middleware.js`도 `/api/v1/*` 요청을 전부 `https://api.muapi.ai`로 rewrite 한다.
→ **MuAPI 키가 없으면 아무것도 동작하지 않는다.** (로컬 모델 제외)

### 💰 무료 / 유료 정리

| 항목 | 비용 |
|---|---|
| 프로그램 소스코드 | 🟢 무료 (MIT 라이선스) |
| 클라우드 생성 (이미지/영상 전부) | 🔴 **유료** — 생성할 때마다 MuAPI 과금 |
| 로컬 생성 (sd.cpp, 데스크톱 앱 전용) | 🟢 무료, 인터넷 불필요 (**이미지만**) |

> README에 "Powered by MuAPI", "White Label $49/mo" 배너가 많은 이유는
> 이 프로젝트가 **MuAPI의 마케팅용 오픈소스**이기 때문. 나쁜 건 아니지만 알고 시작할 것.

---

## 2. 폴더 구조 분석

| 폴더 | 역할 |
|---|---|
| `app/` | **Next.js 웹 버전** (App Router). `npm run dev` → localhost:3000 |
| `electron/` | **데스크톱 앱 버전** (Win/Mac/Linux 설치형) + 로컬 추론 로직 |
| `src/` | 초기 Vanilla JS 버전 잔재. 현재는 Electron 렌더러용 |
| `packages/studio/` | 🔥 **진짜 알맹이.** 모든 스튜디오 UI가 여기 있음 |
| `packages/studio/src/models.js` | **23,466줄** — 400개+ 모델 정의 (단일 소스) |
| `components/StandaloneShell.js` | 탭 네비게이션 + API 키 모달 (앱의 뼈대) |
| `middleware.js` | MuAPI 프록시 rewrite + CSP 보안 헤더 |
| `docs/`, `messages/` | 문서 + 다국어 (en / zh) |
| `tests/` | 로컬 추론 관련 테스트 4개뿐 (커버리지 거의 없음) |

### 스튜디오 탭 15개 (`components/StandaloneShell.js:21` TABS 배열)

`image` · `layers` · `video` · `audio` · `clipping` · `vibe-motion` · `lipsync` ·
`body-swap` · `cinema` · `marketing` · `workflows` · `agents` · `design-agent` ·
`apps` · `ai-influencer`

### 큰 컴포넌트 (줄 수 기준)

| 파일 | 줄 수 |
|---|---:|
| `LayersStudio.jsx` | 4,116 |
| `VideoStudio.jsx` | 3,038 |
| `ImageStudio.jsx` | 1,945 |
| `DrawModal.jsx` | 1,895 |
| `CinemaStudio.jsx` | 1,313 |

### 기술 스택

- Next.js 15 / React 19 / Tailwind CSS 3 / npm workspaces 모노레포
- Electron 33 + electron-builder (데스크톱 패키징)
- Vite 5 (Electron 렌더러 번들)

> ⚠️ README에는 "Next.js 14 / React 18"로 적혀 있지만
> `package.json`은 실제로 **Next 15 / React 19**. 문서가 뒤처져 있음.

### 동작 패턴 (Submit → Poll)

`packages/studio/src/utils/generationLifecycle.js` 기준:

1. `POST /api/v1/{endpoint}` → `request_id` 수신
2. `GET /api/v1/predictions/{request_id}/result` 를 **2초 간격 × 최대 900회 (= 최대 30분)** 폴링
3. 성공 상태: `completed` / `succeeded` / `success`
4. 실패 상태: `failed` / `error` / `cancelled` / `canceled`
5. 결과 URL: `outputs[0]` → `url` → `output.url` 순으로 탐색
6. 실패 시 응답의 `cost.refunded`, `cost.amount_credits` 로 환불 여부 확인 가능

히스토리는 **서버 DB 없이 브라우저 `localStorage`** 에만 저장된다.

---

## 3. 설치 및 사용법

### 🚦 세 가지 경로

| 경로 | 대상 | 난이도 |
|---|---|---|
| A. 설치파일 다운로드 | 그냥 쓰고 싶은 사람 | ⭐ |
| B. 소스코드 실행 | 코드를 뜯어보고 싶은 사람 | ⭐⭐⭐ |
| C. 호스팅 버전 접속 | 설치도 하기 싫은 사람 | 0초 |

---

### 🅰️ 경로 A — 설치파일

다운로드: https://github.com/Anil-matcha/Open-Generative-AI/releases

| OS | 파일 |
|---|---|
| macOS (Apple Silicon) | `Open.Generative.AI-1.0.9-arm64.dmg` |
| macOS (Intel) | `Open.Generative.AI-1.0.9.dmg` |
| Windows x64 | `Open.Generative.AI.Setup.1.0.9.exe` |
| Linux | `.AppImage` / `.deb` |

**보안 경고는 정상이다** (코드 서명/공증을 안 받은 앱이라서):

```bash
# macOS — "손상되었습니다" 뜰 때 (한 번만 실행)
xattr -cr "/Applications/Open Generative AI.app"
# 이후 앱 우클릭 → 열기 → 열기
```

- Windows: SmartScreen → **추가 정보 → 실행**
- Ubuntu 24.04+: AppImage가 안 켜지면 **`.deb` 설치 권장** (AppArmor 프로파일 자동 적용)

> 💡 **로컬(무료) 추론은 데스크톱 앱에서만 가능하다.** 웹 버전은 항상 클라우드 API를 쓴다.

---

### 🅱️ 경로 B — 소스코드로 실행

#### 1) 준비물

```bash
node -v    # v18 이상 (v20~22 권장)
npm -v
```

Node.js: https://nodejs.org

#### 2) 준비 — 딱 한 줄

```bash
cd ~/Open-Generative-AI
npm run setup
```

`package.json:24` 기준, 이 한 줄이 아래 3가지를 수행한다:

```
1. git submodule update --init --recursive   ← 빈 서브모듈 3개 채우기
2. npm install                                ← 의존성 설치
3. npm run build:packages                     ← workflow / agent / design / studio 빌드
```

> 🚨 **절대 건너뛰지 말 것.** `npm install`만 하면 화면이 뜨지 않는다.
> 클론 직후에는 `packages/Vibe-Workflow`, `packages/Open-Poe-AI`,
> `packages/Open-AI-Design-Agent` 가 **비어 있는 상태**다.
> 소요 시간 5~15분.

#### 3) 실행

```bash
npm run dev            # 웹 버전  → http://localhost:3000
npm run electron:dev   # 데스크톱 앱 (Vite 빌드 + Electron)
```

#### 4) 프로덕션 빌드

```bash
npm run build && npm run start          # 웹
npm run electron:build                  # macOS DMG
npm run electron:build:win              # Windows NSIS
npm run electron:build:linux            # Linux AppImage + deb
```

빌드 결과물은 `release/` 폴더에 생성된다.

#### 5) Docker

```bash
docker compose up -d    # → http://localhost:3001
```

---

### 🔑 API 키 발급 (필수)

1. https://muapi.ai/access-keys 접속 후 가입
2. 키 생성
3. 🚨 **키 "이름"이 아니라 "값"을 복사할 것**

```
❌ 이름:  my-first-key
✅ 값:    mu_xxxxxxxxxxxxxxxxxxxx     ← 이걸 붙여넣기
```

4. 앱 첫 화면의 입력창에 붙여넣고 Save

> `.env` 파일을 만들 필요 없다. 키는 브라우저 `localStorage`에 저장되며
> (`StandaloneShell.js:577`) 변경은 **Settings**에서 한다.
> 단, 이 방식은 개인용에만 적합하다. 공용 PC나 상용 서비스에는 부적합.

---

### 🎨 스튜디오별 사용법

#### 🖼️ Image Studio

```
1. 하단 프롬프트 입력
2. 모델 선택 (Flux / Nano Banana 2 / Seedream 5.0 ...)
3. 비율 선택 (16:9 / 1:1 / 9:16)
4. Generate
```

**모드 자동 전환:**

| 상황 | 자동 전환 |
|---|---|
| 텍스트만 입력 | 텍스트 → 이미지 모델 목록 (70개+) |
| 참고 이미지 업로드 | 이미지 → 이미지 모델 목록 (70개+) |

**다중 이미지 입력:** `Nano Banana 2 Edit` 등은 **최대 14장**까지 지원.
체크박스의 순서 번호대로 모델에 전달된다.

**업로드 히스토리:** 한 번 올린 이미지는 저장되어 재업로드가 필요 없다.

#### 🎬 Video Studio

Image Studio와 동일한 패턴. 텍스트만 → t2v, 시작 프레임 업로드 → i2v.
영상은 오래 걸리므로 (최대 30분 폴링) 창을 닫아도 재접속 시 이어받는다.

#### 🎤 Lip Sync Studio

**사진 1장 + 음성 파일 → 말하는 영상**

```
1. [Portrait Image] / [Video] 모드 토글
2. 얼굴 사진(또는 영상) 업로드
3. 음성 파일(mp3/wav) 업로드
4. 모델 선택 — Infinite Talk, LTX 2.3 Lipsync 등 9종
5. 해상도 선택 (480p / 720p / 1080p)
6. Generate
```

프롬프트는 선택사항. 넣으면 표정·동작 연출을 지시할 수 있다.

#### 🎥 Cinema Studio

프롬프트 대신 촬영 장비를 골라 프롬프트로 자동 변환한다.

| 항목 | 옵션 |
|---|---|
| 카메라 | 8K 디지털, 70mm 필름, 16mm 필름, S35 등 6종 |
| 렌즈 | 아나모픽, 매크로, 70s 시네마 프라임 등 11종 |
| 초점거리 | 8mm(초광각) ~ 85mm(인물) |
| 조리개 | f/1.4(얕은 심도) / f/4 / f/11(깊은 심도) |

인물 감성샷 조합: `85mm + f/1.4 + Warm Cinema Prime`

#### 🔀 Workflow Studio

여러 단계를 노드로 연결해 자동 실행한다.

```
[이미지 생성] → [영상 변환] → [립싱크] → 완료
```

- **Templates** — 기존 워크플로우 사용 (시작점 추천)
- **Builder** — 드래그앤드롭 편집
- **Playground** — 폼 UI로 즉시 실행

---

### ⚡ 로컬 모델 (무료, 데스크톱 앱 전용)

```
1. Settings → Local Models
2. "sd.cpp inference engine" → Install (자동 다운로드)
3. 원하는 모델 → Download
4. Image Studio → 모델 선택기 옆 ⚡ Local 토글 ON
5. Generate — API 키 불필요, 과금 없음
```

#### 사용 가능한 모델 6종 (`electron/lib/modelCatalog.js`)

| 모델 | 용량 | 비고 |
|---|---|---|
| **Dreamshaper 8** | 2.1 GB | 🥇 가장 가볍고 무난. 여기서 시작 |
| Realistic Vision v5.1 | 2.1 GB | 실사 계열 |
| Anything v5 | 2.1 GB | 애니/일러스트 |
| SDXL Base 1.0 | 6.9 GB | 고해상도 |
| Z-Image Turbo | 2.5 GB + 보조 2.7 GB | 8스텝 고속 |
| Z-Image Base | 3.5 GB + 보조 2.7 GB | 50스텝 고품질 |

> 🚨 **RAM 경고:** README 기준, 8GB 맥에서 Z-Image는 **시스템이 멈춘다.**
> Z-Image는 16GB 이상 권장. 8GB라면 Dreamshaper 8만 사용할 것.

#### 모델 저장 경로

```
macOS   : ~/Library/Application Support/open-generative-ai/local-ai
Windows : %APPDATA%\open-generative-ai\local-ai
Linux   : ~/.config/open-generative-ai/local-ai
```

다른 드라이브를 쓰려면 앱 실행 전 환경변수 `OPEN_GENERATIVE_AI_LOCAL_AI_DIR` 설정.

#### Wan2GP (별도 GPU 서버)

영상 모델을 로컬로 돌리려면 CUDA/ROCm GPU 머신에 Wan2GP를 직접 띄워야 한다.

```bash
git clone https://github.com/deepbeepmeep/Wan2GP
cd Wan2GP && ./install.sh
python wgp.py --listen --server-name 0.0.0.0
```

이후 앱에서 **Settings → Local Models → Wan2GP server** 에 URL 입력 (예: `http://192.168.1.42:7860`).

---

### 🚑 자주 겪는 문제

| 증상 | 원인 / 해결 |
|---|---|
| `Couldn't find a 'pages' directory` | 잘못된 폴더에서 실행. `package.json`이 있는 루트에서 `npm run dev` |
| 워크플로우/에이전트 탭이 비어 있음 | 서브모듈 미초기화 → `npm run setup` 재실행 |
| 401 / 403 | 키 값이 아니라 키 이름을 입력함 |
| 생성이 계속 로딩 | 영상은 원래 오래 걸림 (최대 30분 폴링) |
| macOS "손상된 앱" | `xattr -cr "/Applications/Open Generative AI.app"` |
| 로컬 모델이 매우 느림 | Metal/GPU 대신 CPU로 폴백된 상태. 엔진 재설치 |
| Ubuntu 24.04에서 실행 실패 | AppImage 대신 `.deb` 설치 |

---

## 4. 수익화 아이디어

### 🧊 냉정한 현실 체크

**이 앱을 그대로 서비스로 오픈하면 돈이 되지 않는다.**

| 문제 | 설명 |
|---|---|
| 진입장벽 0 | MuAPI 키는 누구나 발급 가능. 차별점 없음 |
| 원조가 이미 판매 중 | MuAPI가 White Label을 **$49/mo**에 직접 판매 (README 명시) |
| 종합몰의 저주 | "다 됩니다" = 특정 고객에게 안 팔림 |

> 💡 **핵심:** 이 저장소의 가치는 "완제품"이 아니라 **"부품 창고"**다.
> `models.js`의 모델 카탈로그와 Submit→Poll API 패턴만 가져오고,
> **UI는 특정 업종용으로 새로 만드는 것**이 정답.

---

### 🏆 아이디어 랭킹

지불 주체가 **개인이 아니라 사업자**인 것만 선별했다.

| 순위 | 아이템 | 고객 | 지불의사 | 난이도 |
|:---:|---|---|:---:|:---:|
| 🥇 | **상세페이지 제품컷 생성기** | 스마트스토어·쿠팡 셀러 | 💰💰💰 | ⭐⭐ |
| 🥈 | **부동산 홈스테이징** | 공인중개사 | 💰💰💰 | ⭐⭐ |
| 🥉 | **강사/설계사 아바타 영상** | 교육·보험·병원 | 💰💰 | ⭐⭐⭐ |
| 4 | 음식점 메뉴 사진 | 자영업 | 💰💰 | ⭐ |
| 5 | AI 인플루언서 대행 | 브랜드 | 💰💰💰 | ⭐⭐⭐⭐ |
| ❌ | 프로필 사진 앱 | 개인 | 💰 | 레드오션 |

---

### 🥇 1위 — 상세페이지 제품컷 생성기

**문제:** 신상품 하나 올리려면 스튜디오 예약 → 1컷당 3~5만원 → 3일 대기

**솔루션:**

```
[사장님이 폰으로 찍은 제품 사진 1장]
              ↓
- 흰 배경 누끼컷          (Background Remover)
- 대리석 위 연출컷         (Nano Banana Edit)
- 카페 감성 라이프스타일컷  (Seedream Edit)
- 착용/사용컷
- 인스타 정사각 광고컷      (Marketing Studio)
```

**왜 되는가:**

- ROI가 즉시 이해된다 — "5만원 vs 990원"
- 매달 신상이 나오므로 **구독 유지율이 높다** (일회성 아님)
- 완벽하지 않아도 된다 — 시안을 뽑고 고르는 구조
- 셀러 커뮤니티에서 입소문이 잘 퍼진다

**가격 설계 예시** (⚠️ MuAPI 실제 단가 확인 후 마진 재계산 필수):

| 플랜 | 가격 | 제공 |
|---|---|---|
| 체험 | 0원 | 5장 (카드 등록 없이) |
| 라이트 | 9,900원/월 | 100장 |
| 프로 | 29,000원/월 | 500장 + 워터마크 제거 |
| 무제한 | 79,000원/월 | 3,000장 |

---

### 🥈 2위 — 부동산 홈스테이징

```
[텅 빈 공실 사진] → [가구·소품이 배치된 인테리어 사진]
```

- 매물 1건 수수료가 수백만원이라 몇만원 지출에 저항이 적다
- 해외 경쟁사가 장당 $10~30에 판매하는 **검증된 시장**
- ⚠️ **필수:** `"AI 생성 이미지 — 실제 매물과 다를 수 있음"` 워터마크 강제
  (허위매물 이슈 방어)

---

### 🚨 하지 말아야 할 것

README가 내세우는 **"No content filters"** 로 수익을 내려는 방향은 권하지 않는다.

| 리스크 | 내용 |
|---|---|
| ⚖️ 법적 | 딥페이크 성적 영상물은 형사처벌 대상 (2024년 처벌 강화) |
| 💳 결제 | PG사 심사에서 거절 → 결제 자체가 불가능 |
| 🏢 계정 | MuAPI 계정 정지 시 서비스 전체 중단 |

**돈이 되는 건 "무검열"이 아니라 "업종 특화"다.**

---

## 5. PHP로 만들 수 있는가? → **가능하다**

MuAPI는 특별한 SDK가 없는 **평범한 REST API**다. cURL만 쓸 줄 알면 된다.

게다가 수익화에 필요한 **회원가입 · 결제 · 크레딧 · 관리자 페이지**는
라라벨 쪽이 오히려 더 빠르고, 한국 PG(토스페이먼츠·이니시스) 연동 예제도 PHP에 훨씬 많다.

### ⚠️ 단 하나의 함정: PHP에서 폴링하면 안 된다

원본 코드(`generationLifecycle.js:35`)는 `2초 × 900회 = 최대 30분` 폴링한다.
PHP가 이걸 붙잡고 있으면 `max_execution_time` 초과 → 워커 고갈 → 서비스 전체 다운.

```
❌ 브라우저 → PHP (30분 대기...) → 결과
✅ 브라우저 → PHP → request_id 즉시 반환 (0.5초)
   브라우저가 3초마다 → PHP → "완료됐나?" 반복
```

**원본 앱도 브라우저에서 폴링한다.** 같은 방식을 그대로 쓰면 된다.

---

### 🏗️ 아키텍처

```
[브라우저]
    ↓ ① 생성 요청
[내 PHP 서버]  ← 🔐 MuAPI 키는 여기에만 (프론트 노출 금지)
    │  · 로그인 확인
    │  · 크레딧 잔액 확인 & 차감
    │  · jobs 테이블 기록
    ↓
[api.muapi.ai]
    ↓ request_id
[내 PHP 서버] → 브라우저 (즉시 응답)

... 브라우저가 3초 간격으로 상태 조회 ...
```

### 🗄️ 최소 DB 스키마

```sql
CREATE TABLE users (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  email VARCHAR(255) UNIQUE,
  password_hash VARCHAR(255),
  credits INT DEFAULT 5,              -- 무료 체험 5장
  plan VARCHAR(20) DEFAULT 'free'
);

CREATE TABLE jobs (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT,
  request_id VARCHAR(100) UNIQUE,     -- MuAPI가 반환한 ID
  endpoint VARCHAR(100),
  status VARCHAR(20) DEFAULT 'pending',
  result_url TEXT,
  credits_charged INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX (user_id, created_at)
);

CREATE TABLE credit_logs (            -- 환불 분쟁 대비용, 반드시 남길 것
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT,
  amount INT,
  reason VARCHAR(50),                 -- purchase / generate / refund
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

### 💻 `generate.php` — 생성 요청

```php
<?php
session_start();
header('Content-Type: application/json');

$userId = $_SESSION['user_id'] ?? null;
if (!$userId) { http_response_code(401); exit(json_encode(['error' => '로그인 필요'])); }

$pdo = new PDO('mysql:host=localhost;dbname=mystudio;charset=utf8mb4', 'user', 'pw', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

$COST = 1;  // 이미지 1장 = 1크레딧

// ── 크레딧 차감 (동시요청 방어) ──
$pdo->beginTransaction();
$stmt = $pdo->prepare("UPDATE users SET credits = credits - ? WHERE id = ? AND credits >= ?");
$stmt->execute([$COST, $userId, $COST]);
if ($stmt->rowCount() === 0) {
    $pdo->rollBack();
    http_response_code(402);
    exit(json_encode(['error' => '크레딧이 부족합니다']));
}
$pdo->commit();

// ── MuAPI 호출 ──
$endpoint = 'nano-banana-pro';                  // models.js 에서 가져온 값
$payload  = [
    'prompt'       => $_POST['prompt'] ?? '',
    'aspect_ratio' => $_POST['ratio']  ?? '1:1',
];

$ch = curl_init("https://api.muapi.ai/api/v1/{$endpoint}");
curl_setopt_array($ch, [
    CURLOPT_POST           => true,
    CURLOPT_POSTFIELDS     => json_encode($payload),
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_TIMEOUT        => 30,               // ⚠️ 짧게. 폴링하지 않는다
    CURLOPT_HTTPHEADER     => [
        'Content-Type: application/json',
        'x-api-key: ' . getenv('MUAPI_KEY'),    // 🔐 .env 에서만 읽는다
    ],
]);
$res  = json_decode(curl_exec($ch), true);
$code = curl_getinfo($ch, CURLINFO_HTTP_CODE);
curl_close($ch);

$requestId = $res['request_id'] ?? $res['id'] ?? null;

// ── 실패 시 크레딧 즉시 환불 ──
if ($code >= 400 || !$requestId) {
    $pdo->prepare("UPDATE users SET credits = credits + ? WHERE id = ?")
        ->execute([$COST, $userId]);
    http_response_code(502);
    exit(json_encode(['error' => '생성 요청 실패 (크레딧 환불 완료)']));
}

$pdo->prepare("INSERT INTO jobs (user_id, request_id, endpoint, credits_charged) VALUES (?,?,?,?)")
    ->execute([$userId, $requestId, $endpoint, $COST]);

echo json_encode(['request_id' => $requestId]);   // ✅ 0.5초 내 응답
```

---

### 💻 `status.php` — 상태 확인

```php
<?php
session_start();
header('Content-Type: application/json');

$userId    = $_SESSION['user_id'] ?? null;
$requestId = $_GET['request_id']  ?? '';
if (!$userId || !$requestId) { http_response_code(400); exit('{}'); }

$pdo = new PDO('mysql:host=localhost;dbname=mystudio;charset=utf8mb4', 'user', 'pw');

// 🔒 타인의 job 조회 차단 (누락 시 개인정보 사고)
$stmt = $pdo->prepare("SELECT * FROM jobs WHERE request_id = ? AND user_id = ?");
$stmt->execute([$requestId, $userId]);
$job = $stmt->fetch(PDO::FETCH_ASSOC);
if (!$job) { http_response_code(404); exit('{}'); }

if ($job['status'] === 'completed') {            // 완료된 건은 캐시로 응답
    exit(json_encode(['status' => 'completed', 'url' => $job['result_url']]));
}

$ch = curl_init("https://api.muapi.ai/api/v1/predictions/{$requestId}/result");
curl_setopt_array($ch, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_TIMEOUT        => 15,
    CURLOPT_HTTPHEADER     => ['x-api-key: ' . getenv('MUAPI_KEY')],
]);
$r = json_decode(curl_exec($ch), true);
curl_close($ch);

$status = strtolower($r['status'] ?? 'pending');

// 상태값은 generationLifecycle.js 와 동일하게 맞춘다
if (in_array($status, ['completed', 'succeeded', 'success'], true)) {
    $url = $r['outputs'][0] ?? $r['url'] ?? $r['output']['url'] ?? null;
    $pdo->prepare("UPDATE jobs SET status='completed', result_url=? WHERE id=?")
        ->execute([$url, $job['id']]);
    exit(json_encode(['status' => 'completed', 'url' => $url]));
}

if (in_array($status, ['failed', 'error', 'cancelled', 'canceled'], true)) {
    $pdo->prepare("UPDATE users SET credits = credits + ? WHERE id = ?")
        ->execute([$job['credits_charged'], $userId]);
    $pdo->prepare("UPDATE jobs SET status='failed' WHERE id=?")->execute([$job['id']]);
    exit(json_encode(['status' => 'failed', 'error' => '생성 실패 (크레딧 환불 완료)']));
}

echo json_encode(['status' => 'processing']);
```

---

### 💻 프론트엔드 (JS)

```javascript
async function generate(prompt) {
  const r = await fetch('/generate.php', {
    method: 'POST',
    body: new URLSearchParams({ prompt })
  }).then(res => res.json());

  if (r.error) return alert(r.error);

  // 3초 간격, 최대 10분
  for (let i = 0; i < 200; i++) {
    await new Promise(s => setTimeout(s, 3000));
    const s = await fetch(`/status.php?request_id=${r.request_id}`).then(res => res.json());

    if (s.status === 'completed') {
      document.querySelector('#result').src = s.url;
      return;
    }
    if (s.status === 'failed') return alert(s.error);
  }
}
```

원본 앱 22,000줄 중 실제로 필요한 핵심 로직은 위 **100줄 남짓**이다.

---

### 🚨 체크리스트 (누락 금지)

| # | 항목 | 누락 시 |
|:--:|---|---|
| 1 | API 키는 서버 `.env`에만 | 프론트 노출 시 요금 폭탄 |
| 2 | 크레딧 선차감 → 실패 시 환불 | 무료 무한 생성 악용 |
| 3 | job 조회 시 `user_id` 검증 | 타인 결과물 노출 (개인정보 사고) |
| 4 | Rate Limit (분당 N회) | 봇이 API 원가 소진 |
| 5 | **결과 이미지를 내 스토리지로 복사** | MuAPI URL 만료 시 고객 데이터 유실 |
| 6 | AI 생성물 표기 | 표시광고법 이슈 |
| 7 | `credit_logs` 기록 | 환불 분쟁 시 증빙 부재 |

> 특히 5번: 생성 완료 즉시 결과 파일을 자체 스토리지(S3 등)로 복사해 둘 것.

---

### 🗓️ 4주 로드맵

| 주차 | 할 일 | 목표 |
|---|---|---|
| 1주 | MuAPI 키 발급 → 위 PHP 파일로 이미지 1장 생성 | 기술 검증 |
| 2주 | **셀러 5명 인터뷰** ("월 만원 내실 의향 있나요?") | 수요 검증 |
| 3주 | 로그인 + 크레딧 + 업로드 연결 | MVP |
| 4주 | 무료 체험 5장으로 베타 오픈 (결제는 이후) | 첫 고객 |

> 🔑 **2주차가 가장 중요하다.** 코드보다 인터뷰가 먼저다.
> 이 질문 하나가 3개월을 아껴준다.

---

## 6. 최종 요약

| 질문 | 답 |
|---|---|
| 이게 뭐야? | 400개+ AI 모델을 한 화면에서 쓰는 오픈소스 스튜디오 (UI만 있음) |
| 공짜야? | 코드는 무료, 클라우드 생성은 유료(MuAPI), 로컬 이미지 생성은 무료 |
| 뭐부터 해? | `npm run setup` → `npm run dev` → MuAPI 키 입력 |
| 뭐가 제일 쓸모있어? | `packages/studio/src/models.js` (모델 400개 사용법 카탈로그) |
| 수익화 되나? | 그대로는 안 됨. **업종 특화 자동화 도구**로 재조립하면 가능 |
| PHP로 되나? | 된다. 결제/회원 붙이기엔 오히려 더 적합. 단 **폴링은 브라우저에서** |

---

*이 문서는 저장소 코드를 직접 분석해 작성했습니다. 가격·마진 수치는 예시이며,*
*MuAPI 실제 단가를 확인한 뒤 재계산해야 합니다.*
