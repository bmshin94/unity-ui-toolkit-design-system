# Unity UI Toolkit Design System — 전수조사 분석 및 활용 정리 (한국어)

> 이 문서는 레포 전체를 파일 단위로 조사한 결과와, 설치/사용법·정체 규명·수익화 아이디어까지
> 한국어로 정리한 자료입니다. 작성일: 2026-09-20

---

## 🔗 GitHub 주소

| 구분 | 주소 |
| --- | --- |
| **이 레포 (작업 중인 fork)** | https://github.com/bmshin94/unity-ui-toolkit-design-system |
| **원본 레포 (upstream)** | https://github.com/sinanata/unity-ui-toolkit-design-system |
| **라이브 웹 데모** | https://sinanata.github.io/unity-ui-toolkit-design-system/ |
| **UPM Git URL** | `https://github.com/sinanata/unity-ui-toolkit-design-system.git?path=/Assets/DesignSystem` |
| **같은 저자 Unreal 버전** | https://github.com/sinanata/unreal-umg-design-system |
| **빌드 오케스트레이터 (submodule)** | https://github.com/sinanata/unity-cross-platform-local-build-orchestrator |

---

## 1. 이게 뭐하는 건가

**Unity 6 UI Toolkit(UXML + USS) 전용 드롭인 디자인 시스템 패키지.**
웹으로 치면 Bootstrap / Tailwind 같은 UI 프레임워크를 Unity용으로 만든 것.

| 항목 | 값 |
| --- | --- |
| 패키지 ID | `com.sinanata.designsystem` |
| 버전 | **v1.5.3** |
| 라이선스 | **MIT** (상업적 이용 가능) |
| 최소 요구 | Unity **6000.0+** (머티리얼 FX / PanelRenderer는 6000.5+) |
| 저자 | Sinan Ata (`sinanata`) |
| 실전 검증 | 출시작 *Leap of Legends* (Steam / iOS / Android)의 모든 메뉴·HUD·상점 |

### 실측 규모 (전수조사 결과)

| 구성 | 수치 |
| --- | --- |
| USS 스타일시트 | 15개 / **4,216줄** |
| C# 소스 | 55개 파일 / **9,319줄** |
| SVG 아이콘 | **120개** |
| 컴포넌트 | **42개** |
| 셰이더 | `DsFx.cginc` + `DsFxBlueprint.shader` |
| 문서 | README 490줄 + docs 6종 2,100줄 + CHANGELOG **160KB** |

### 폴더 구조 요약

```
📦 루트
├── README.md / AGENTS.md / llms.txt / CLAUDE.md   ← 문서 + AI 규칙 파일
├── CHANGELOG.md (160KB)                           ← 버그 원인/해결 에세이 수준
├── docs/  ARCHITECTURE · COMPONENTS · FONTS · MATERIALS · ICONS · MOBILE
├── Assets/DesignSystem/   💎 실제 배포되는 알맹이 (이것만 복사)
│   ├── Resources/UI/Styles/DesignSystem/   USS 15개
│   ├── Resources/Textures/Icons/           SVG 120개
│   ├── Resources/UI/Themes/                Dark / Light 테마 에셋
│   ├── Resources/Fx/Shaders/               GPU 머티리얼 셰이더
│   ├── Runtime/ Behaviour · Theme · Typography · Fx
│   └── Editor/  Theme Configurator · Google Fonts 임포터 · FX 컴파일 체크
├── Assets/Showcase/   ← 데모용 호스트 프로젝트 (복사하면 안 됨)
└── Tools/             ← WebGL 빌드 자동화 + Unity 엔진 버그 패치
```

### 5개 핵심 시스템

1. **디자인 토큰** (`DesignTokens.uss`, 149줄)
   `--color-primary` 한 줄만 바꾸면 UI 전체 색이 바뀌는 CSS 변수 구조.
   게임용 희귀도 토큰(common / rare / epic / legendary)까지 내장.

2. **자동 부착 런타임** (`DesignSystemBehaviourBase.cs`, 956줄 — 최대 파일)
   Unity UI Toolkit이 못 하는 것을 C#이 보완: 토글 knob 주입, 스피너 무한 회전
   (USS는 루프 애니메이션 불가), 스켈레톤 시머, 드래그&드롭, 드롭다운 팝업 위치 보정.
   **씬의 모든 UIDocument / PanelRenderer에 자동으로 붙어 별도 배선 불필요.**

3. **테마 시스템** (`ThemeData.cs` 399줄 + 에디터 툴)
   테마가 코드가 아니라 **ScriptableObject 에셋**. `Design System > Theme Configurator`
   에서 실시간 미리보기로 편집. 베이크된 스타일시트 1장을 루트에 추가하면
   `var()` 캐스케이드가 `:hover` / `:disabled` / `:checked`까지 전부 다시 칠함.

4. **구글 폰트 파이프라인** (`Typography/`, 약 2,000줄)
   - 메뉴에서 **2,045개 패밀리** 검색 → 임포트
   - 가변 폰트의 `fvar` 테이블을 직접 파싱해 **100~900 전체 굵기** 추출
     (Unity 기본 Font Asset Creator는 1개 굵기만 뽑음)
   - 다국어 **폴백 체인** + 아랍어/히브리어 RTL, 데바나가리 재배열 등 shaping 처리
   - CJK 한자 통합(Han unification) 문제까지 `DsFonts.ApplyFace`로 대응

5. **GPU 머티리얼 FX** (`Fx/`, 약 1,600줄 + 셰이더)
   `style.unityMaterial` 기반으로 UI에 실제 셰이더를 입힘(청사진 질감, 음각 글씨 등).
   **전체 애니메이션 런타임이 "프레임당 float 1개"** — 유휴 상태 CPU 비용 0에 수렴,
   클럭을 고정하면 렌더링이 바이트 단위로 재현 가능.

### 언제 쓰나

| 상황 | 적합성 |
| --- | --- |
| Unity 6 + UI Toolkit으로 게임/앱 UI 제작 | ⭕ 최적 |
| PC + 모바일 동시 지원 (`.mobile` 클래스 1개로 전환) | ⭕ |
| 다국어 게임 (폰트 폴백 체인) | ⭕ |
| 구버전 uGUI(Canvas) 프로젝트 | ❌ 불가 |
| 웹 / React 프로젝트 | ❌ 직접 불가 (아이디어·CSS는 이식 가능) |

---

## 2. 쉬운 버전 요약

- **디자인 시스템 = 이케아 가구 세트.** 버튼·입력창·모달 42종이 이미 조립돼 있음.
- **마법 1 — 색 하나로 전체 변경:** `--color-primary` 한 줄 = 반장 옷 색 바꾸면 반 전체가 바뀜.
- **마법 2 — 자동으로 붙는 집사:** 956줄짜리 런타임이 Unity가 못 그리는 걸 알아서 채움.
- **마법 3 — 글자 하나로 모바일 변신:** `.mobile` 추가 → 버튼 36px→48px, 사이드바→하단바.
- **마법 4 — 폰트 지옥 탈출:** 에디터에선 멀쩡한데 빌드하면 □□□ 되는 Unity 악명 높은 함정을
  폴백 체인 + `Verify Fonts` 검사로 원천 차단.
- **마법 5 — 진짜 셰이더 UI:** 납작한 사각형이 아니라 GPU 머티리얼 질감, 그것도 CPU 부하 없이.

---

## 3. 자주 묻는 질문 7가지

### ① 설치 및 사용법

**설치 3가지 경로**

```bash
# A. 폴더 복사 (가장 쉬움) — Assets/DesignSystem/ 만 복사할 것
git clone https://github.com/sinanata/unity-ui-toolkit-design-system ../ds-src
cp -r ../ds-src/Assets/DesignSystem Assets/DesignSystem
```

```
# B. Package Manager → + → Add package from git URL
https://github.com/sinanata/unity-ui-toolkit-design-system.git?path=/Assets/DesignSystem
```

```bash
# C. git submodule + OS 심볼릭 링크 (원본 업데이트 추적용)
git submodule add https://github.com/sinanata/unity-ui-toolkit-design-system Vendor/ds
```

**사용법**

```xml
<ui:UXML xmlns:ui="UnityEngine.UIElements">
  <Style src="project://database/Assets/DesignSystem/Resources/UI/Styles/DesignSystem/DesignSystem.uss" />
  <ui:VisualElement class="ds-root">
    <ui:Button text="시작하기" class="ds-btn ds-btn--primary" />
  </ui:VisualElement>
</ui:UXML>
```

**클래스 문법(BEM):** 블록 `.ds-btn` / 요소 `.ds-btn__icon` / 변형 `.ds-btn--primary` / 상태 `.is-active`

**추가되는 Unity 메뉴**
- `Design System > Theme Configurator` — 실시간 미리보기 테마 편집기
- `Design System > Google Fonts` — 폰트 2,045개 검색·설치
- `Design System > FX > Compile Check` — 셰이더 컴파일 검사
- `Design System > Showcase > Verify Fonts` — 폰트 누락 검사

**절대 금지:** 인라인 `style="...: var(--token)"` — Unity 6 클론 타임에 예외가 나서
UXML 전체 로딩이 실패함. `var()`는 `.uss` 파일 안에서만 사용.

### ② 플러그인? 스킬? MCP?

**전부 아님 — Unity 에셋 패키지(C# + USS 라이브러리)다.**

| 구분 | 실행 주체 | 이 레포 |
| --- | --- | --- |
| Claude Code 플러그인 | Claude Code | ❌ (`plugin.json` 없음) |
| 스킬(Skill) | Claude | ❌ (`SKILL.md` 없음) |
| MCP 서버 | 별도 서버 프로세스 | ❌ (`.mcp.json` 없음) |
| **UPM 패키지** | **Unity 엔진** | ✅ (`package.json` + `.asmdef` 존재) |

헷갈리는 이유는 `AGENTS.md` / `llms.txt` / `CLAUDE.md` / `copilot-instructions.md` 때문인데,
이건 **"AI가 이 코드를 고칠 때 지켜야 할 규칙 문서"** 일 뿐 실행 가능한 도구가 아니다.

### ③ API 토큰이 필요한가

**필요 없음.** 코드 전체를 `api_key|token|secret` 패턴으로 검색했으나 인증 키 사용처 0건.

네트워크 호출은 3곳뿐이며 전부 공개 엔드포인트 / 인증 불필요:

| 파일 | 주소 | 인증 |
| --- | --- | --- |
| `GoogleFontsCatalog.cs` | `fonts.google.com/metadata/fonts` | 불필요 |
| `GoogleFontsCatalog.cs` | `api.github.com/repos/google/fonts/...` | 불필요(비인증) |
| `DsGoogleFonts.cs` | `raw.githubusercontent.com/google/fonts/...` | 불필요 |

게다가 **구글 폰트 기능을 쓸 때만** 호출되므로, 폰트를 쓰지 않으면 완전 오프라인 동작.
(유일한 주의: GitHub 비인증 API는 시간당 60회 제한)

### ④ 왜 GitHub에서 유명한가

1. **빈틈을 정확히 메움** — Unity 6가 UI Toolkit을 밀지만 공식 디자인 시스템이 없음.
2. **실전 검증** — 실제 출시 게임의 전 화면이 이 시스템으로 제작됨.
3. **만질 수 있는 라이브 데모** — GitHub Pages 인터랙티브 쇼케이스 + 3D 월드스페이스 갤러리.
4. **문서 집착** — CHANGELOG 160KB, 버그마다 원인·해결·재발 방지까지 서술.
   외부 기여자(@slimshader, @lmProgramming, @FlowingFrost) 이슈/PR이 실제로 머지됨.
5. **AI 시대 최적화** — `AGENTS.md` + `llms.txt` + `copilot-instructions.md` 완비.

> 참고: 별(⭐) 개수는 이번 분석 세션에서 직접 확인하지 않았으며, 위 내용은 레포 내용물 근거 분석.

### ⑤ 로컬 에이전트 구축에 도움이 되나

**직접적으로는 무관**(LLM/에이전트 로직 없음). **간접적으로는 매우 유용** — 배울 점 3가지:

1. **`AGENTS.md` 작성법** — "금지 + 이유 + 결과"를 한 세트로 못 박는 Golden Rules 9개 구조.
   (예: "인라인 style에 `var()` 금지 — Unity 6 클론 타임 예외로 UXML 전체가 로드 실패함")
2. **`llms.txt`** — 1만 줄 레포를 1페이지로 압축해 AI 컨텍스트 비용을 줄이는 기법.
3. **계층적 지시 파일 라우팅** — `llms.txt`(기계용) → `AGENTS.md`(범용) →
   `CLAUDE.md` / `copilot-instructions.md`(도구별)로 중복 없이 분기.

즉 **"에이전트가 일할 작업장을 어떻게 준비하는가"의 모범 사례**로서 가치가 있다.

### ⑥ 수익화 아이디어 → 아래 4장 참조

### ⑦ React나 PHP로 만들 수 있나

**React: 가능, 그것도 쉬움.** 이 시스템의 아이디어 자체가 웹 CSS에서 온 것이기 때문.

| 이 레포 | 웹/React 대응 |
| --- | --- |
| USS `--color-primary` | CSS 변수 (문법 거의 동일) |
| UXML `class=` | JSX `className=` |
| `ThemeData` 에셋 | JSON 테마 + Context Provider |
| `.mobile` 클래스 플립 | CSS `@media` 쿼리 |
| `DesignSystemBehaviour.cs` | React 컴포넌트 + hooks |
| SVG 아이콘 120개 | 그대로 재사용 가능 |

USS는 CSS의 부분집합이라 **4,216줄 중 대부분이 거의 그대로 이식 가능**하다.
예외: GPU 머티리얼 FX는 Unity 셰이더 기반이라 WebGL/canvas로 재작성 필요.

**PHP: 역할이 달라 절반만 가능.** 서버 언어이므로
Blade/Twig 컴포넌트 partial 렌더링과 테마 토큰 → CSS 동적 생성 API는 가능하지만,
토글 애니메이션·드래그&드롭 같은 인터랙션은 JS가 필수.

---

## 4. 수익화 아이디어

### 법적 전제 (MIT)

상업 이용·수정·유료 재배포·비공개 전환 모두 허용. **단 저작권 표시 + MIT 원문 동봉 필수.**
다만 무료 오픈소스를 원본 그대로 재판매하는 것은 현실성·평판 양면에서 비추천.
**"이걸 재료로 무엇을 더 얹느냐"가 실제 수익 포인트.**

### Tier 1 — 즉시 실행 가능 (난이도 ⭐⭐)

**① 테마 팩 판매** — 테마가 에셋이라 색·아이콘·폰트·FX 패밀리를 묶으면 전혀 다른 제품이 됨.
판타지 RPG / 사이버펑크 / 캐주얼 모바일 / 미디블 / SF 팩 등.
가격 **$15~35**, 판매처 Unity Asset Store · itch.io · Gumroad, 제작 1~2주.

**② 아이콘 팩 확장** — 현재 120개. `Icons.uss` 규격(흰색 채움 SVG + 틴트 캐스케이드)에
맞춰 MMORPG 500종 / 팜 시뮬 300종 등을 만들면 "공식 확장팩" 포지셔닝 가능. **$10~25**.

### Tier 2 — 본격 수익 (난이도 ⭐⭐⭐⭐)

**③ 웹 기반 테마 에디터 SaaS — 최우선 추천**

문제: 테마를 만들려면 Unity를 켜야 하는데, 팀의 디자이너는 Unity를 쓰지 않는다.
해결: 브라우저에서 색을 고르면 `ThemeData.asset` / `DesignTokens.uss` / `tokens.json` /
Figma 토큰 / Tailwind config를 한 번에 내보내는 웹앱.

- React로 바로 구현 가능 (원본 쇼케이스가 이미 웹에서 돌아간다는 증거 존재)
- Unity뿐 아니라 웹·Flutter·RN 토큰까지 출력하면 시장이 크게 확대
- 구독형이라 반복 수익(MRR) 확보

| 플랜 | 가격 | 내용 |
| --- | --- | --- |
| Free | $0 | 테마 3개, 워터마크 |
| Pro | **$9/월** | 무제한, 팀 공유, 전 포맷 |
| Studio | **$49/월** | 팀 시트, 브랜드 잠금, API |

유료 200명 기준 월 $1,800 (ARR 약 $21,600).
스택 제안: React + Vite + CSS 변수 + Zustand / 백엔드 Laravel 또는 Node / 결제 Lemon Squeezy.

**④ 게임 UI 킷 프리미엄** — 원본은 범용 컴포넌트만 있고 "완성된 화면"이 없음.
인벤토리 그리드, 스킬 트리, 상점·결제 플로우, 퀘스트 로그, 설정(키 리바인딩), 로비/매치메이킹,
가챠 연출 등을 얹어 판매. **$49~99**, 번들 **$149**.

### Tier 3 — 지식 판매 (난이도 ⭐⭐⭐)

**⑤ 강의 / 유튜브** — "Unity 6 UI Toolkit 완전정복". 인프런·유데미·클래스101.
₩55,000~99,000 × 수강생 500명 ≈ ₩2,750만. **한국어 UI Toolkit 자료가 거의 없어 선점 가능.**

**⑥ 컨설팅 / 외주** — "Unity UI 시스템 구축". 프로젝트당 ₩300~1,000만.
이 레포를 포트폴리오이자 출발점으로 쓰면 작업 속도가 빨라져 마진이 커짐.

### Tier 4 — AI 결합 (난이도 ⭐⭐⭐⭐⭐)

**⑦ AI UI 생성기** — "판타지 RPG 인벤토리 화면 만들어줘" → AI가 이 디자인 시스템 클래스만
사용해 UXML + USS 생성 → Unity에 바로 적용. **`AGENTS.md`와 `llms.txt`가 이미 존재한다는 점이
곧 AI에게 먹일 완벽한 룰북이 준비돼 있다는 뜻.** 크레딧($0.5/생성) 또는 $19/월 구독.

### 우선순위 정리

| 순위 | 아이디어 | 난이도 | 수익성 | 이유 |
| --- | --- | --- | --- | --- |
| 1 | ③ 웹 테마 에디터 SaaS | ⭐⭐⭐⭐ | 최상 | React로 즉시 가능 + 반복 수익 |
| 2 | ① 테마 팩 | ⭐⭐ | 중 | 당장 시작 가능, 현금 흐름 |
| 3 | ⑤ 한국어 강의 | ⭐⭐⭐ | 상 | 경쟁자 거의 없음 |
| 4 | ④ 게임 UI 킷 | ⭐⭐⭐⭐ | 상 | 제작 기간 김 |
| 5 | ⑦ AI 생성기 | ⭐⭐⭐⭐⭐ | 최상 | 기술 난이도 최상 |

**로드맵 제안**

```
1개월차 : ① 테마팩 1종 제작 → Gumroad 업로드 (시장 반응 테스트)
2~3개월 : ③ 웹 테마 에디터 MVP(React) 무료 배포 → 트래픽 확보
4개월차 : 유료 플랜 오픈 + ⑤ 유튜브로 유입 퍼널 구축
6개월차 : ④ 게임 UI 킷으로 객단가 상승
```

**현실 체크:** Unity 에셋스토어는 수수료 30%이고, 무료 대안이 있는 카테고리는 경쟁이 치열하다.
따라서 ③번처럼 **원본이 하지 못하는 영역**을 노리는 편이 안전하고 확장성이 크다.

---

## 5. 참고 문서 링크

| 문서 | 내용 |
| --- | --- |
| [README.md](../README.md) | 개요, 설치 3경로, 퀵스타트, 아키텍처 |
| [AGENTS.md](../AGENTS.md) | AI 에이전트용 규칙(Golden Rules 9개) |
| [llms.txt](../llms.txt) | LLM용 기계 판독 색인 |
| [docs/ARCHITECTURE.md](ARCHITECTURE.md) | USS 스택, 토큰, 로드 순서, 런타임 |
| [docs/COMPONENTS.md](COMPONENTS.md) | 전체 `ds-*` 클래스 레퍼런스 |
| [docs/FONTS.md](FONTS.md) | 폰트 임포터, 가변 폰트, 다국어 폴백 |
| [docs/MATERIALS.md](MATERIALS.md) | GPU 머티리얼 파이프라인 |
| [docs/ICONS.md](ICONS.md) | SVG 아이콘 120종과 틴트 캐스케이드 |
| [docs/MOBILE.md](MOBILE.md) | `.mobile` 한 클래스 반응형 전환 |
| [CONTRIBUTING.md](../CONTRIBUTING.md) | 네이밍 규칙, PR 체크리스트 |
