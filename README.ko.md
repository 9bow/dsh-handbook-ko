# DeepSeek Harness 백서 · dsh-handbook

> **DeepSeek Harness 한국어 핸드북 × 생태계 관찰 센터** —— 0부터 1까지 dsh를 익히고, 780개 디스커션 스레드로 생태계를 이해하기 · 한국어 · [中文](./README.md) · [English](./README.en.md)

**📖 [온라인으로 읽기](https://electricitysheep.github.io/dsh-handbook/) · 📄 [PDF 다운로드](./DeepSeek-Harness-白皮书.pdf) · ⭐ [Star로 응원하기](https://github.com/Electricitysheep/dsh-handbook/stargazers)**

**1804개 플러그인 · 780개 디스커션 스레드 · 15개 챕터 핸드북 · 195개 커뮤니티 응답 · 280+ Stars**

<p align="center">
  <img src="./docs/assets/banner.svg" alt="dsh-handbook banner" width="720"/>
</p>

<div align="center">

![GitHub stars](https://img.shields.io/github/stars/Electricitysheep/dsh-handbook?style=flat&color=yellow)
![GitHub release](https://img.shields.io/github/v/tag/Electricitysheep/dsh-handbook?label=release&color=success)
![dsh-handbook](https://img.shields.io/badge/dsh--handbook-백서-blue)
![chapters](https://img.shields.io/badge/챕터-15-green)
![pdf](https://img.shields.io/badge/PDF-5.5MB-orange)
![license](https://img.shields.io/badge/license-CC--BY--NC--SA--4.0-lightgrey)
![dsh](https://img.shields.io/badge/dsh-0.1.0--rc.6-8A2BE2)

</div>

> [!WARNING]
> dsh는 현재 `0.1.0-rc.6`(프리릴리스 단계)입니다. 프로덕션 환경 도입은 신중히 평가하세요. 자세한 내용은 [ℹ️ 버전 설명](#ℹ️-버전-설명)을 참고하세요.

## 🚀 30초 빠른 체험

```bash
# 1. 설치 (Node.js ≥ 22 필요)
npx -y @deepseek-ai/dsh web

# 2. 브라우저에서 http://127.0.0.1:3080 열고 대화 시작
# 3. 또는 일회성 작업 실행 (스크립트/CI에 적합)
dsh --profile headless "안녕, 한 문장으로 자기소개 해줘"
```

> 체계적으로 배우고 싶다면 [🗺 학습 경로(3일 계획)](./docs/roadmap.md)를, 일단 실행부터 해보고 싶다면 [제2장: 5분 빠른 시작](./docs/02-quickstart.ko.md)을, 빠르게 찾아보고 싶다면 [📇 한 페이지 치트시트](./docs/cheatsheet.md)를 참고하세요

<p align="center">
  <img src="./docs/assets/demo-webui.gif" alt="dsh Web UI 실측 데모" width="720"/>
  <br/>
  <sub><b>30초 만에 이해하는 dsh Web UI</b>: 새 세션 → 작업 입력 → 모델 선택 → 전송 → AI 응답</sub>
</p>

## 🎯 이것은 무엇인가

**DeepSeek Harness(`dsh`)**는 DeepSeek가 2026-08-13에 공식 오픈소스로 공개한 Agent 런타임입니다 —— "모든 것이 플러그인"(everything is a plugin)인 프레임워크입니다.

<img width="614" height="230" alt="image" src="https://github.com/user-attachments/assets/19482c24-2208-468e-ad38-9096d9270f8d" />

하지만 공식 문서는 아키텍처 설명 위주라서 **처음부터 손에 익히는 경로가 빠져 있습니다**.

**이 백서가 그 경로를 채웁니다**: "Agent 런타임이란 무엇인가"부터 시작해서 설치, 사용, 플러그인 개발, 성능 튜닝까지 —— 모든 챕터가 그대로 복사해 실행할 수 있는 명령어이며, 전부 로컬 환경에서 실측 검증되었습니다. **목표는 어떤 개발자든 이 책을 따라오면 0부터 1까지 실제로 쓰고 만들 수 있게 하는 것입니다.**

### 왜 공식 문서만 보지 않고 이 책을 읽어야 하는가

| 공식 문서 | 이 백서 |
|---|---|
| 아키텍처 관점(AGENTS.md / architecture.md) | **초보자 관점**: 0부터 1까지 이어지는 하나의 경로 |
| 산발적인 예제 | **모든 챕터가 실행 가능**, 명령어 전부 실측 |
| 한국어 튜토리얼 없음 | **한국어 우선**, 중국어/영어와 함께 |
| 생태계 실전 경험 부재 | **실제 플러그인/PR 분석**(트러블슈팅과 보안 제약 포함) |

## 🎁 이 책이 당신에게 주는 것

| 당신이… | 얻게 되는 것 |
|---|---|
| 🆕 **dsh를 처음 접했다면** | 3일 만에 0부터 1까지 배우는 학습 경로(매일 목표+검수 기준) |
| 🛠 **개발자라면** | 그대로 클론해 쓸 수 있는 플러그인 템플릿 + 설정 레퍼런스 총정리 |
| ⚖️ **도구를 선택 중이라면** | 6개 주류 Agent 비교(표+설명) + 동일 모델 실측 벤치마크 |
| ⚡ **튜닝이 필요하다면** | 추론 강도(reasoning effort) 전략 + 캐시 적중률 특집(실측 97%) |
| 📚 **사례가 필요하다면** | 5개 실제 복잡 사례(소요 시간/산출물/검증 포함) |

## 🌟 감사와 커뮤니티

먼저 모든 Star, 댓글, 기고에 감사드립니다 —— 이 핸드북은 한 사람의 작품이 아니라 dsh 커뮤니티가 함께 "키운" 결과물입니다.

공개 이틀 만에 다음과 같은 반응을 얻었습니다:

- ⭐ **280+ Stars** —— 갓 공개된 튜토리얼치고는 기대 이상이라 감사할 따름입니다
- 💬 **공식 저장소에서 195개 응답** —— [디스커션](https://github.com/deepseek-ai/deepseek-harness/discussions)에서 계속 함께 트러블슈팅하고 교류하고 있습니다
- 🧠 **FAQ의 39개 질문 대부분이 실제 질문에서 왔습니다** —— 커뮤니티가 묻는 것을 우리가 정리합니다(#380/#817/#1052…)
- 📦 [awesome-dsh-plugin](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin)에 등재 PR 제출([#33](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin/pull/33), 병합 대기 중); 롼이펑(阮一峰) 주간지 자체 추천 제출 완료
- 🤝 30+ 커뮤니티 프로젝트와 상호 링크(dsh-usage / dsh-sgme / AgentSoul / dsh-vault / egress-guard / agentmemory…)

> 콘텐츠는 디스커션을 기반으로 지속적으로 업데이트됩니다([반영 파이프라인](./docs/research/feedback-pipeline.md), 19개 항목 추적 중). 도움이 되었다면 Star가 가장 큰 응원입니다.

## 🔭 생태계 관찰(제15장 · 전체 리포트)

> **1804개 플러그인 저장소**의 데이터 집계 × **780개 디스커션 스레드** 정성 관찰을 교차 검증해 얻은 5가지 핵심 결론:

| 결론 | 한 줄 요약 |
|---|---|
| 🪟 **Windows가 첫 번째 고통 지점** | 중국어(다국어) 경로(15+ 스레드 동일 원인) / koffi / 포트 / 자식 프로세스 —— 데이터와 디스커션이 이중으로 입증 |
| 🧩 **"공식이 안 하면 커뮤니티가 다 한다"** | 데스크톱 쉘 140+ / TUI / 메모리 77 / 비전 132 —— 건강한 상호 보완 |
| 🛡️ **보안 감사는 활발한데 도구는 희소** | sandbox 카테고리 플러그인 단 9개 —— **공급 공백** |
| 🐛 **직렬화 버그 계열** | unknown tool / reasoning 생략 / run_code 유실 —— rc 시기의 주요 전장 |
| 💰 **비용 투명화는 필수 수요** | 캐시 적중률 97% 실측 + 비용 도구가 우후죽순처럼 등장 |

**6개의 능력 공백**(공식이 우선 채워야 할 영역): 비전 채널 · 메모리 seam · 데스크톱 TUI 프로토콜 · 평가 폐루프 · Windows 1급 지원 · 플러그인 레지스트리
> 전체 리포트(개발자/도구 선택자/관망자를 위한 제안 포함): [제15장](./docs/15-ecosystem-report.ko.md)

## 📚 목차(0부터 1까지)

<div align="center">

| 🗺️ **[학습 경로(3일 계획)](./docs/roadmap.md)** | 0부터 1까지: 매일 목표 + 검수 기준 + 학습 원칙 |
|---|---|

</div>

### 🟢 1단계 · 입문: 인지와 시작하기

<div align="center">

| 📖 **[제1장 · Harness 이해하기](./docs/01-intro.ko.md)** | ⚡ **[제2장 · 5분 만에 시작하기](./docs/02-quickstart.ko.md)** |
|---|---|
| 주류 Agent와의 전면 비교 · FAQ · [中文](./docs/01-intro.md) · [EN](./docs/01-intro.en.md) | 설치 · web/headless 듀얼 모드 · 추론 강도 · [中文](./docs/02-quickstart.md) · [EN](./docs/02-quickstart.en.md) |

</div>

### 🔵 2단계 · 개발: 골격과 플러그인

<div align="center">

| 🧩 **[제3장 · profile과 플러그인 시스템](./docs/03-profiles.ko.md)** | 🛠️ **[제4장 · 플러그인 개발 실전](./docs/04-plugin-dev.ko.md)** |
|---|---|
| 커스터마이즈 가능한 골격 · 플러그인 장착 · 확장 포인트 · 실제 트러블슈팅 | 첫 플러그인을 처음부터 작성(완전한 코드 + 테스트 + 실기 검증) |

</div>

### 🟠 3단계 · 실전: 시나리오와 튜닝

<div align="center">

| 📦 **[제5장 · dsh 활용 시나리오](./docs/05-cases.ko.md)** | 🚀 **[제6장 · 심화와 성능 튜닝](./docs/06-advanced.ko.md)** |
|---|---|
| 5대 시나리오 · 고캐시 적중률 특집 · 5개 산업 관점 | 추론 강도 전략 · 소요 시간 분석 · 트러블슈팅 목록 |

</div>

### 🟣 4단계 · 생태계: 능력과 오케스트레이션

<div align="center">

| 🌐 **[제7장 · 생태계와 리소스](./docs/07-ecosystem.ko.md)** | 🧰 **[제8장 · 도구와 컨텍스트 시스템](./docs/08-tools-context.ko.md)** | 🔗 **[제9장 · MCP·서브에이전트·워크플로우](./docs/09-mcp-subagent-workflow.ko.md)** |
|---|---|---|
| 공식 채널 · 참여 경로 · 추천 읽기 순서 | 60+ 능력 패키지 지도 · 내장 도구 · compaction | 외부 도구 연동 · 병렬 서브에이전트 · 다단계 오케스트레이션 |

</div>

### 🔴 5단계 · 심화: 복잡한 사례와 전망

<div align="center">

| 🧪 **[제10장 · 복잡한 실전 사례](./docs/10-complex-cases.ko.md)** | 🔮 **[제11장 · 미래 전망](./docs/11-future.ko.md)** | ⚠️ **[제12장 · 알려진 한계와 경계](./docs/12-limitations.ko.md)** |
|---|---|---|
| dsh가 실제로 해낸 것: 데이터 정제 파이프라인 186초 · 5개 버그 수정 94초 | 기술/생태계/경쟁/기회/리스크 예측 + 타임라인 | rc 버전 솔직 리뷰: 불안정성 · 초기 생태계 · 크로스플랫폼 약점 |

</div>

<div align="center">

| 🛡️ **[제13장 · 보안과 샌드박스](./docs/13-security.ko.md)** | 💰 **[제14장 · 캐시와 비용](./docs/14-cost.ko.md)** |
|---|---|
| 샌드박스 메커니즘 · 권한 모델 · 승인 흐름 · 플러그인 보안 감사 체크리스트 | 캐시 적중률 실측 97% · 비용 모델 · 추론 강도 연계 · 예산 실전 |

</div>

| 📊 **[제15장 · 생태계 전체 리포트](./docs/15-ecosystem-report.ko.md)** | |
|---|---|
| 1804개 플러그인 × 780개 스레드 교차 검증: 5대 인사이트 + 6개 능력 공백 + 생태계 참여자를 위한 제안 |

### 📎 부록

<div align="center">

| 📚 **[부록 A·용어집](./docs/appendix-glossary.md)** · 📦 **[부록 B·공식 패키지 빠른 참조](./docs/appendix-packages.md)** · 📊 **[부록 C·Benchmark](./docs/benchmark.md)** |
|---|
| 30+ 용어 · 명령어 빠른 참조 · 공식 @deepseek-ai/* 패키지 목록 · 동일 모델 3개 Agent 실측 |

</div>

## 💎 핵심 내용 요약 미리보기(클릭하면 바로 확인, 링크만 있는 게 아닙니다)

<details>
<summary><b>📖 제1장: DeepSeek Harness 이해하기 —— 세 가지 직관 + 능력 매트릭스</b></summary>

- **세 가지 직관**: dsh = Agent의 레고 베이스플레이트; harness = 모델 바깥을 감싸는 엔지니어링 레이어; 2026 = Agent 프로그래머블 시대
- **핵심 사실**: MIT 오픈소스 · TypeScript · "모든 것이 플러그인" · 2026-08-13 공개
- **dsh vs 5개 주류 Agent 능력 매트릭스**(Claude Code / Codex / OpenCode / Gemini / Kimi): 오픈소스✅, 모델 독립✅, **공식 레벨 플러그인 체계**(유일), 커스텀 UI✅, headless CI✅
- **선택 기준**: 깊은 커스터마이징+생태계 → dsh; 바로 쓸 수 있는 것 → Claude Code
</details>

<details>
<summary><b>⚡ 제2장: 5분 빠른 시작 —— 30초 만에 실행</b></summary>

- **명령어 하나로 시작**: `npx -y @deepseek-ai/dsh web` → http://127.0.0.1:3080
- **듀얼 모드**: web(대화 UI) / headless(`dsh --profile headless "작업"`, CI 친화적)

- **추론 강도 3단계**: `low`(가장 빠름/단순 작업) · `high`(기본값) · `max`(최고 성능/복잡한 추론) —— **성능의 핵심: 도구 체인 시간의 90%가 사고(thinking)에 쓰입니다**. > 참고: `low`는 이 백서가 실측한 게이트웨이(pi-ai/opencode-go) 등급이며, **DeepSeek 공식 어댑터는 `off`(사고 끄기/최고 속도) / `high` / `max`**입니다(02-quickstart 2.3절 참고)
- **첫 플러그인**: Git 패널 4단계 장착
</details>

<details>
<summary><b>🧩 제3장: profile과 플러그인 시스템 —— 커스터마이즈 가능한 골격</b></summary>

- **profile** = bundle 스택 + 나만의 patch 레이어(`package.json` + `cordis.patch.yml`)
- **플러그인 장착은 단 2곳만 수정**(의존성 추가 + insert 줄 추가)
- **host/client 두 개의 반쪽**: npm 패키지 하나가 = Node 쪽 도구/서비스 + 브라우저 쪽 UI
- **5대 확장 포인트**: `agent/request` waterfall, `conversationEvents`, `ctx.slots`, `settings`, `ctx.provide`
- **6개의 실제 트러블슈팅**: rc.1 의존성 단절, 플러그인 main 누락, `next()` await 누락, 타입 인식 실패, ModuleLoader, 포트 점유
</details>

<details>
<summary><b>🛠 제4장: 플러그인 개발 실전 —— 완전히 실행 가능한 코드</b></summary>

- **속도 개선 플러그인을 처음부터 작성**(완전 분석): 순수 함수 결정 로직 + `agent/request` waterfall 주입
- **핵심 기법**: 결정 로직을 순수 함수로 분리(단위 테스트 밀리초 단위) → 실기에서는 "주입이 실제로 일어났는지"만 검증
- **3가지 개발 원칙**: 확장 포인트를 먼저 찾을 것 / 로직을 순수 함수로 분리할 것 / 실기 검증은 생략 불가
- **실기 로그 증거**: `calls=[{name:"write"}] => reasoningEffort=low`
</details>

<details>
<summary><b>📦 제5장: 실전 사례 —— 세 개의 실제 오픈소스 PR 완결편</b></summary>

- **Git 패널 push/pull/fetch**(PR #10): `--force-with-lease` 안전 레드라인 + 로컬 bare-repo 통합 테스트 + Playwright 실기 검증
- **HTML 초안 미리보기**(PR #11): 샌드박스 보안 제약 하의 srcdoc 결정 순수 함수
- **속도 개선 플러그인 예제**: 긴 도구 체인의 각 단계마다 사고 강도 다운그레이드
</details>

<details>
<summary><b>🚀 제6장: 심화와 성능 튜닝 —— 시간이 어디로 가는가</b></summary>

- **성능 모델**: 도구 체인 작업 시간의 90%가 모델 사고(매 도구 호출 전)에 쓰임
- **강도 전략**: 단순한 라운드는 low / 일상 작업은 high / 복잡한 작업은 max —— 강도를 낮추는 것이 가장 효과적인 속도 개선 레버
- **7개의 실제 트러블슈팅**: "단순 작업이 갑자기 빨라짐 = 캐시 적중"이라는 평가 함정 포함
- **성적표를 볼 때 물어야 할 세 가지**: 누가 측정했나 / 어떤 harness인가 / 검증기는 얼마나 엄격한가
</details>

<details>
<summary><b>🌐 제7장: 생태계와 리소스 —— dsh 생태계에 합류하는 지도</b></summary>

- **공식 채널**: 저장소 / API 문서 / Discord / Discussions
- **현재 상태**: 공식은 아직 외부 PR을 받지 않음 → **dsh-plugin 생태계 프로젝트를 만드는 것이 공식이 지목한 기여 방식**
- **초보자 경로**: 사용해보기 → 작은 PR → 플러그인 공개 → 콘텐츠 작성
</details>

<details>
<summary><b>🧰 제8장: 도구와 컨텍스트 시스템 —— 능력 엔진</b></summary>

- **60+ 공식 능력 패키지 지도**: 도구/컨텍스트/세션/서브에이전트/MCP/워크플로우/보안
- **내장 도구(실측)**: read/write/grep/glob/edit/bash/todo/skill
- **산출물 추적**: 도구가 반환하는 locations → 대화 끝에서 산출물 열람 가능
- **컨텍스트 주입**: 시스템 프롬프트 계층화 + 스킬 카탈로그
- **긴 대화 자동 압축**(compaction) + 샌드박스/권한/승인 보안 레이어
</details>

<details>
<summary><b>🔗 제9장: MCP, 서브에이전트, 워크플로우 —— Agent의 체계화</b></summary>

- **MCP**: 외부 도구 서버 연동(커뮤니티에 이미 토큰 추적 플러그인 존재)
- **서브에이전트**: 작업 병렬 위임(대규모 저장소 조사/긴 작업 분해)
- **워크플로우**: 결정론적 다단계 오케스트레이션(가져오기→정제→리포트→검증)
- **4단계 초보자 경로**: 단일 Agent → +MCP → +서브에이전트 → +워크플로우
</details>

<details>
<summary><b>🧪 제10장: 복잡한 실전 사례 —— dsh가 실제로 해낸 것</b></summary>

- **사례 A**: 데이터 품질 분석→정제→시각화(186초, 52→35줄로 축소, chart.png, 트레이드오프 설명 포함)
- **사례 B**: 5개 버그 수정 + 49개 테스트(94초, pytest 49 passed, 0으로 나누기/음수/정밀도 경계 커버)
- **특징**: 다단계 도구 체인 자동 오케스트레이션 + 판단력 + 산출물 추적 가능
- 개인정보 관련 안내: 전부 합성 데이터/가상 코드
</details>

<details>
<summary><b>📚 부록: 용어집 + 명령어 빠른 참조</b></summary>

- **30+ 용어**: harness/profile/bundle/cordis/확장 포인트/waterfall/compaction…
- **명령어 빠른 참조**: dsh 핵심 / 환경 / 트러블슈팅 / 플러그인 개발
- **Benchmark**: 동일 모델 3개 Agent 실측(3회 샘플링 중앙값)
</details>

<details>
<summary><b>🔮 제11장: 미래 전망 —— 다섯 가지 차원의 예측</b></summary>

- **기술/생태계/경쟁/기회/리스크** 5차원 예측 + 타임라인
- **기회 포인트**: 공식 생태계 초기 단계, dsh-plugin 프로젝트를 만드는 것이 선점 보너스
</details>

<details>
<summary><b>⚠️ 제12장: 알려진 한계와 경계 —— rc 버전 솔직 리뷰</b></summary>

- **불안정성**: rc 반복 속도가 빠르고 파괴적 변경이 잦음
- **초기 생태계**: 공식 패키지는 60+ 개지만 플러그인 생태계는 이제 막 시작
- **크로스플랫폼 약점**: Windows 계열 트러블슈팅 기록(Node 버전 레드라인 포함)
</details>

<details>
<summary><b>🛡️ 제13장: 보안과 샌드박스 모델 —— 프로덕션에 올릴 수 있는 핵심 근거</b></summary>

- **샌드박스 메커니즘**: 프로세스 격리(bwrap/Landlock/Seatbelt) + 권한 등급화 + 승인 흐름
- **커뮤니티가 확인한 경계**: node:vm이 안전 경계가 아님, approval 순환 참조, workspace-write의 재귀 삭제 등 실제 탈출 경로
- **플러그인 보안 감사 체크리스트**: 서드파티 감사 방법론([#454](https://github.com/deepseek-ai/deepseek-harness/discussions/454))
</details>

<details>
<summary><b>💰 제14장: 캐시와 비용 엔지니어링 —— "저렴함"을 엔지니어링으로 바꾸기</b></summary>

- **캐시 메커니즘**: 컨텍스트 캐시 + 적중률 실측 97%(Flash 할인 98% / Pro 99%+)
- **비용 모델**: 토큰이 어디에 쓰이는가 + 추론 강도 연계 + 실제 작업 예산
- **시각화**: session log / 잔액 플러그인으로 건별 비용 확인
</details>

## 🖥 데모(Demo) —— 직접 결과 보기

### ① Web UI 대화(`dsh web`)

```bash
dsh web    # → http://127.0.0.1:3080
```

![dsh Web UI 대화](./docs/assets/demo-web-chat.png)

### ② Headless CLI(일회성 작업, 스크립트/CI에 적합)

`dsh --profile headless "안녕, 한 문장으로 자기소개 해줘"`를 실행하면(명령어는 [🚀 30초 빠른 체험](#-30초-빠른-체험) 참고):

```bash
# → 안녕하세요! 저는 DeepSeek 기반 AI 코딩 어시스턴트입니다. 코드 작성, 디버깅,
#    파일 처리, 자료 검색은 물론 다양한 개발 및 업무 작업을 도와드릴 수 있습니다.
```

### ③ 플러그인 생태계(Git 패널, `dsh-better-sidebar`)

![dsh Git 패널(better-sidebar 플러그인)](./docs/assets/demo-git-panel.png)

> 전체 이미지·텍스트 데모는 [📺 30초 만에 이해하는 dsh](./docs/demo.md) 참고.

## 🧰 빠른 시작 자료(핵심만 바로 확인)

<details>
<summary><b>📇 한 페이지 치트시트</b> —— 설치 · 명령어 · 추론 강도 · 트러블슈팅</summary>

```bash
npx -y @deepseek-ai/dsh web          # 설치와 동시에 Web UI 실행
dsh --profile headless "작업"        # 일회성 작업(스크립트/CI)
```

추론 강도: `low`(가장 빠름/단순 라운드) · `high`(기본값) · `max`(최고 성능/복잡한 추론) —— `low`는 실측 게이트웨이(pi-ai) 등급이며, **공식 어댑터는 `off`/`high`/`max`**를 사용
> 도구 체인 작업 시간의 90%가 사고에 쓰입니다 —— 강도를 낮추는 것이 가장 효과적인 속도 개선 레버
> 전체 카드: [docs/cheatsheet.md](./docs/cheatsheet.md)
</details>

<details>
<summary><b>🔧 플러그인 템플릿</b> —— 장착은 단 2단계</summary>

```yaml
# ① package.json에 의존성 추가
"my-plugin": "link:C:\\path\\to\\my-plugin"
# ② cordis.patch.yml에 장착 추가
- insert:
    - id: my-plugin
      name: my-plugin
```
```bash
cd ~/.dsh/profiles/web && pnpm install && dsh web
```
> 그대로 클론 가능한 템플릿(순수 함수+waterfall+테스트 포함): [examples/plugin-template/](./examples/plugin-template/README.md)
</details>

<details>
<summary><b>⚙️ 설정 레퍼런스</b> —— settings.yaml 핵심</summary>

```yaml
agent-default-model:
  model: deepseek-v4-flash    # 또는 deepseek-v4-pro
  reasoningEffort: high       # off(사고 끄기/최고 속도) / high(기본값) / max(최고 성능)
```
> 전체 필드(profile/cordis.patch.yml/자주 쓰는 시나리오): [docs/config-reference.md](./docs/config-reference.md)
</details>

<details>
<summary><b>❓ FAQ Top 5</b></summary>

1. **dsh는 모델인가요?** 아니요 —— 런타임이며, 모델은 llm 플러그인을 통해 연결됩니다
2. **Claude Code와 차이는?** Claude Code는 "완성차", dsh는 "레고 베이스플레이트"(오픈소스로 커스터마이즈 가능)
3. **비용이 드나요?** dsh 자체는 무료 오픈소스; 대화는 사용량 기반 과금(캐시 할인: Flash 등급 98% / Pro 등급 99%+, 실측 세션 캐시 적중률 97%)
4. **플러그인 설치 시 404 오류?** rc.1 의존성 단절 —— `^0.1.0-rc.6` 라인 사용
5. **프로덕션에 쓸 수 있나요?** rc 단계라 파괴적 변경 있음; 생태계를 탐색하는 용도로는 지금 시작 가능
> 전체 FAQ(6개 카테고리): [docs/faq.md](./docs/faq.md)
</details>

## ⚖️ DSH vs 주류 Agent(능력 매트릭스)

| 항목 | **dsh** | Claude Code | OpenAI Codex | OpenCode | Gemini CLI | Kimi CLI |
|---|---|---|---|---|---|---|
| 오픈소스 | ✅ MIT | ❌ | ❌ | ✅ MIT | ❌ | ❌ |
| 모델 종속 | 모델 독립적 | Claude 계열 | GPT 계열 | 임의 | Gemini 계열 | Kimi 계열 |
| **플러그인 체계** | **공식 레벨: 모든 것이 플러그인, 공식 패키지 60+** | 설정/훅 | 설정 | 설정 | 없음 | 없음 |
| 커스텀 UI | ✅(client 반쪽) | ❌ | ❌ | 일부 | ❌ | ❌ |
| 자동화/CI | ✅ headless | ✅ | ✅ | ✅ | ✅ | ✅ |
| TUI | 플러그인으로 가능 | ✅ 내장 | ✅ 내장 | ✅ 내장 | ✅ | ✅ |
| 생태계 단계 | 초일(2026-08-13) | 성숙 | 성숙 | 성숙 | 성숙 | 초기 |
| 적합한 대상 | 깊은 커스터마이징+생태계 | 바로 사용 | 바로 사용 | OpenCode 사용자 | Google | Kimi |

> 실측 사례, 동일 모델 다중 Agent 비교 데이터는 [제1장](./docs/01-intro.ko.md)과 benchmark 챕터 참고.

## 📊 동일 모델 × 다른 Agent 실측(2026-08-13)

> 모델은 `deepseek-v4-flash`로 통일(동일 게이트웨이, 동일 키), Agent 엔지니어링 레이어만 비교. 5개 작업 모두 정확히 완료되었고, 차이는 효율성에 있습니다:

| Agent | 총 소요 시간 | 정확도 |
|---|---|---|
| **omp** | **70초** | 45/45 ✅ |
| **dsh** | **130초** | 45/45 ✅ |
| **opencode** | 172초 | 45/45 ✅ |

> 5개 작업 × 3회 샘플링 중앙값(T1 파일 생성 → T5 다중 파일 리팩터링), 45/45 전부 정답. 전체 방법론/해석은 [📊 Benchmark 부록](./docs/benchmark.md) 참고.

<p align="center">
  <img src="./docs/assets/benchmark-bar.svg" alt="benchmark 막대그래프: omp 70초 / dsh 130초 / opencode 172초" width="720"/>
</p>


## 📄 백서 PDF

- **중국어 완전판**: [DeepSeek-Harness-白皮书.pdf](./DeepSeek-Harness-白皮书.pdf)(15개 챕터 + 부록 ABC, ~13만+자, 5.5MB)
- **영어 완전판**: [DeepSeek-Harness-Handbook.pdf](./DeepSeek-Harness-Handbook.pdf)(15개 챕터 + 부록, 83페이지, 약 16만자, 1.7MB)
- **한국어 완전판**: [DeepSeek-Harness-핸드북.pdf](./DeepSeek-Harness-핸드북.pdf)(16개 챕터 + 부록 ABC, 148페이지, 3.9MB, 부록은 중국어 원문)

## 🌐 생태계와의 연계

이 백서의 방법론은 실제 오픈소스 실천에서 나왔습니다:
- [DSH-better-sidebar](https://github.com/omdsh-dev/DSH-better-sidebar) —— 커뮤니티 사이드바 플러그인(제5장 사례)

### 🧩 추천 커뮤니티 플러그인(공식 디스커션 / [awesome-dsh-plugin](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin) 선별)

| 플러그인 | 용도 |
|---|---|
| [dsh-specflow](https://github.com/lonelymoon87/dsh-specflow) | 스펙 기반 개발: 스킬+명령어+목표 구현+진행 컨텍스트 |
| [dsh-gitflow](https://github.com/lonelymoon87/dsh-gitflow) | 승인 게이트가 있는 Git 워크플로우(status/diff/commit/branch) |
| [dsh-guardian](https://github.com/lonelymoon87/dsh-guardian) | 위험한 작업 정책 검사 + 출력 마스킹 + 보안 심사 |
| [dsh-code-intel](https://github.com/lonelymoon87/dsh-code-intel) | Tree-sitter 심볼 인덱싱 + 하이브리드 검색 |
| [dsh-tianshu-tui](https://github.com/huiliyi37/dsh-tianshu-tui) | 터미널 UI(TUI) |
| [dsh-computer-use](https://github.com/Anionex/dsh-computer-use) | 접근성 우선의 macOS 컴퓨터 제어 |
| [dsh-data-agent](https://github.com/omdsh-dev/dsh-data-agent) | DB에 연결해 SQL을 작성하는 데이터 Agent |
| [dsh-balance-meter](https://github.com/Ghost011118/dsh-balance-meter) | 잔액 + 세션 비용 실시간 표시 |
| [dsh-usage](https://github.com/kestiany/dsh-usage) | 토큰 사용량 + 비용 추정 + 52주 히트맵(#1169) |
| [dsh-sgme](https://github.com/freehul/sgme) | 메모리 엔진: 시나리오별 주입 + 자동 정리(세션 65-96% 절감, #1052) |
| [AgentSoul](https://github.com/yuhui-sama/dsh-agentsoul) | 로컬 페르소나 + 장기 기억 + 메모리 증류(#1478) |
| [dsh-vault](https://github.com/akslcw/dsh-vault) | 암호화 자격 증명 보관소: TOTP/API Key/SSH 암호화 저장(#1457) |

> 전체 목록은 [awesome-dsh-plugin](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin)(122+ 플러그인) 참고. 등재를 원하시나요? [커뮤니티 사례 모집](https://github.com/Electricitysheep/dsh-handbook/discussions/12)

### 📣 공식 디스커션 활발한 응답

[deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) Discussions에서 지속적으로 활발하게 응답 중(100+ 게시물): 플러그인 트러블슈팅 #380 / 보안 감사 #817 / 캐시 비용 #1052/#1234 / 생태계 인사이트 게시물 #839 등([반영 파이프라인](./docs/research/feedback-pipeline.md)에서 피드백 지속 반영)

## 🙏 기여와 피드백

- ⭐ 도움이 되었나요? Star로 지속적인 업데이트를 응원해주세요
- 📝 **실제 사례를 실행해보셨나요?** 백서에 기고해 수록(서명 + 분기별 선정 PDF): [커뮤니티 사례 모집](https://github.com/Electricitysheep/dsh-handbook/discussions/12) ← 바로 댓글, 템플릿 준비되어 있음
- 챕터/명령어가 작동하지 않나요? rc 버전 반복으로 인한 것이니 issue로 지적해주세요
- 참여하고 싶으신가요? [🤝 기여 가이드(CONTRIBUTING)](./CONTRIBUTING.md) 참고 · 계획을 보고 싶으신가요? [🗺️ 로드맵(ROADMAP)](./ROADMAP.md) · 생태계 참여는 [제7장](./docs/07-ecosystem.ko.md) 참고

## ℹ️ 버전 설명

- dsh `0.1.0-rc.6` / DeepSeek-V4-Flash-0731(2026-08-13 오픈소스 공개) 기준
- 예시 환경: Windows 11 + Node 24

### 🔄 최근 업데이트

- **15개 챕터 완전판**(제13장 보안 샌드박스 / 제14장 캐시 비용 / 제15장 생태계 전체 리포트) + 부록 A/B/C
- 디스커션 피드백 지속 반영: FAQ 39개 / KVCache 규칙 / 내장 Agent 프리셋 / run_code 비동기 트러블슈팅([반영 파이프라인](./docs/research/feedback-pipeline.md))

## 📜 라이선스

콘텐츠 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) · 예시 코드 MIT
