# 제 3 장: profile과 플러그인 시스템

> 본 장의 목표: dsh의 커스터마이즈 가능한 골격을 이해합니다 —— profile은 어떻게 구성되는지, 플러그인은 어떻게 장착하는지, host/client 두 반쪽이란 무엇인지. **이것이 "사용자"에서 "개발자"로 넘어가는 분기점입니다.**

## TL;DR(본 장 핵심, 30초 버전)

1. **profile = 하나의 실행 가능한 형태**: `~/.dsh/profiles/<name>/` 디렉터리이며, `package.json`(플러그인 매니페스트) + `cordis.patch.yml`(패치 레이어)로 구성됩니다
2. **플러그인 장착은 두 단계뿐**: `package.json`에 의존성 추가 + `cordis.patch.yml`에 장착 줄 추가, 그다음 `pnpm install` 후 재시작
3. **host 반쪽은 Node에서, client 반쪽은 브라우저에서 실행**: npm 패키지 하나가 `exports["."]`와 `exports["./client"]`로 두 얼굴을 동시에 가질 수 있습니다
4. **동작을 바꾸려면 확장 포인트를 찾고, 코어를 fork하지 마세요**: `agent/request` waterfall, `conversationEvents`, `ctx.slots.inject`, `settings` 서비스가 4대 자주 쓰는 훅입니다
5. **rc 단계의 3대 트러블슈팅**: 의존성 버전은 `^0.1.0-rc.6` 라인 사용, `next()`는 반드시 await, client 테스트에는 dsh 런타임이 필요

<details><summary>본 장 목차</summary>
- [3.1 profile: 실행 가능한 설정 스택](#31-profile-실행-가능한-설정-스택)
  - [내장 Agent 프리셋: 표준 / PTC / 미니멀 / 생성](#내장-agent-프리셋-표준-ptc-미니멀-생성)
- [3.2 플러그인 하나 장착하기: 두 곳만 수정](#32-플러그인-하나-장착하기-두-곳만-수정)
- [3.3 host 반쪽과 client 반쪽: 하나의 패키지, 두 얼굴](#33-host-반쪽과-client-반쪽-하나의-패키지-두-얼굴)
- [3.4 확장 포인트: 동작을 바꾸려면 훅을 먼저 찾고, 코어를 fork하지 마세요](#34-확장-포인트-동작을-바꾸려면-훅을-먼저-찾고-코어를-fork하지-마세요)
- [3.5 흔한 트러블슈팅(실제 경험)](#35-흔한-트러블슈팅실제-경험)
  - [의존성 해석 트러블슈팅](#의존성-해석-트러블슈팅)
  - [플러그인 설치 형식 트러블슈팅](#플러그인-설치-형식-트러블슈팅)
  - [그 외 흔한 트러블슈팅](#그-외-흔한-트러블슈팅)
</details>

## 3.1 profile: 실행 가능한 설정 스택

dsh는 **profile**로 "실행 가능한 형태 하나"를 표현합니다. 공식이 두 개를 내장하고 있고, 나머지는 플러그인으로 만듭니다:

| profile | 용도 | 명령어 |
|---|---|---|
| `web` | Web UI(대화 + 사이드바 + 도구) | `dsh web` |
| `headless` | 일회성 CLI 작업 | `dsh --profile headless "작업"` |
| `tui`(플러그인 필요) | 터미널 UI | `dsh --profile tui`(미내장, 플러그인 설치 필요) |

profile 디렉터리는 이렇게 생겼습니다(`~/.dsh/profiles/<name>/`):

```text
profiles/web/
├── package.json        # 플러그인 의존성 + dsh.profile 매니페스트(bundles 순서)
├── cordis.patch.yml    # 당신의 패치 레이어: 플러그인 장착/오버라이드 선언
├── cordis.yml          # (생성된) 합성 설정
├── pnpm-workspace.yaml
└── node_modules/
```

**로딩 순서**(공식 문서 원문): 내장 bundle(`dsh-base` → `dsh-web-app`) → profile의 `cordis.patch.yml` → 사용자 레벨 `~/.dsh/cordis.patch.yml` → `--patch` 오버라이드 레이어.

### 내장 Agent 프리셋: 표준 / PTC / 미니멀 / 생성

profile은 "**어떤 형태로 시작할 것인가**"를 해결하고, Agent 프리셋은 "**어떤 방식으로 Agent를 실행할 것인가**"를 해결합니다. dsh는 네 가지 Agent 프리셋을 내장하고 있으며(소스는 공식 저장소 `apps/cli/config/agent-presets/`의 `agent.cordis.yml`), 각 프리셋마다 기본으로 로드하는 플러그인 조합이 다릅니다:

| 프리셋 | 포지셔닝 | 로드되는 내용 | 전형적인 시나리오 |
|---|---|---|---|
| **표준** | 기본 프리셋 | 완전한 도구 조합 | 일상적인 Agent 작업 |
| **PTC**(Code mode) | 코드 주도 도구 체인 | 프로그래매틱 도구 호출 —— 모델이 여러 라운드의 도구 호출을 조합하는 코드 조각을 생성 | 복잡한 다단계 도구 워크플로우 |
| **미니멀** | 벤치마크 테스트 | `bash` + 파일 편집 도구 하나만 | 최소 환경에서의 모델 벤치마크 테스트 |
| **생성** | 플러그인 개발용 | 현재 런타임 검사, 메모리 내에서 Cordis 플러그인 실험, 새 프리셋 조합 | 새로운 플러그인 조합의 구축과 테스트 |

**PTC 명칭 설명**([#1052](https://github.com/deepseek-ai/deepseek-harness/discussions/1052) 댓글의 커뮤니티 상세 설명): PTC = **Programmatic Tool Calling(프로그래매틱 도구 호출)**. 공식 중국어 사이트는 "PTC 모드"라는 마케팅 이름만 사용했고(공식 사이트 원문: "PTC 모드는 모델이 생성한 코드 조각을 통해 여러 라운드의 도구 호출을 조합합니다"), 공식 영문 페이지에서는 이에 대응하는 이름이 **Code mode**입니다; "Programmatic Tool Calling"이라는 완전한 풀네임은 커뮤니티의 상세 설명([cnblogs 가이드](https://www.cnblogs.com/sing1ee/p/22455466))에서 나온 것이며 소스 코드 구현과 완전히 일치합니다. 메커니즘과 이점은 제8장 8.3.1절 참고; 4가지 실행 모드 전체 개요는 제2장 2.4.1절 참고.

## 3.2 플러그인 하나 장착하기: 두 곳만 수정

커뮤니티 플러그인 `dsh-better-sidebar`를 장착하는 것을 예로 들어봅니다(실제 조작):

**① `package.json`에 의존성 추가**(`link:`는 로컬 소스 코드를 가리키거나, npm 패키지 이름을 사용):

```json
{
  "dependencies": {
    "dsh-better-sidebar": "link:C:\\path\\to\\DSH-better-sidebar"
  }
}
```

**② `cordis.patch.yml`에 장착 줄 추가**:

```yaml
- insert:
    - id: better-sidebar
      name: dsh-better-sidebar
```

**③ 설치하고 재시작**:

```bash
cd ~/.dsh/profiles/web && pnpm install
dsh web   # 재시작 후 적용됨
```

> 💡 플러그인은 기본적으로 **세션별로 격리**됩니다(`better-sidebar`의 레이아웃/탭은 세션 단위로 영속화됩니다) —— 장착은 profile 레벨이지만, 상태는 세션 레벨입니다.

## 3.3 host 반쪽과 client 반쪽: 하나의 패키지, 두 얼굴

**"반쪽"(half) = 플러그인 구현의 절반**. 플러그인 하나는 두 부분으로 나뉘어 서로 다른 환경에서 실행될 수 있습니다: **host 반쪽**은 Node 프로세스에서 실행되고(파일 시스템/명령/서비스 권한 보유), **client 반쪽**은 브라우저에서 실행됩니다(web UI, 인터페이스와 상호작용만 가능). 둘은 cordis 서비스로 연결됩니다(`ctx.provide`/`ctx.get`).

**플러그인은 host 반쪽만 있을 수도 있습니다** —— UI가 필요 없는 플러그인(도구류, 서비스류)은 host 반쪽만 있으면 됩니다: `dsh.client`를 선언하지 않고 `exports["./client"]`를 작성하지 않으면 됩니다(제4장의 템플릿이 바로 순수 host 플러그인입니다). 반대로 client 반쪽만 있을 수도 있지만(3.5절 Q4 참고), client 반쪽은 파일 시스템에 직접 접근할 수 없어서 host 반쪽의 연결이 필요합니다.

플러그인은 두 실행 반쪽을 동시에 가질 수 있습니다:

| 반쪽 | 실행 위치 | 역할 | 예시 |
|---|---|---|---|
| **host 반쪽** | Node 프로세스 | 도구, 서비스, 이벤트, 파일 시스템, 프로세스 | `apply(ctx)`로 도구/서비스 등록 |
| **client 반쪽** | 브라우저(web profile) | UI, 상호작용, DOM | `package.json`의 `dsh.client` 선언 + `src/client/` |

`package.json`에서 client 반쪽을 선언하는 방법:

```json
{
  "dsh": {
    "client": {
      "platform": "web",
      "inject": [],        // 주입이 필요한 client 서비스(보통 비어 있음; 공식 예시도 [])
      "immediately": true  // web 시작과 동시에 즉시 로드할지 여부
    }
  },
  "exports": {
    ".": { "types": "./lib/types/index.d.ts", "default": "./lib/index.js" },
    "./client": { "types": "./lib/types/client/index.d.ts", "default": "./lib/client.js" }
  }
}
```

- host 반쪽: `exports["."]` → `apply(ctx)`(cordis 플러그인 본체)
- client 반쪽: `exports["./client"]` → 브라우저 쪽의 `apply(ctx)`
- `inject`: 필요한 서비스 선언(cordis 의존성 주입)

## 3.4 확장 포인트: 동작을 바꾸려면 훅을 먼저 찾고, 코어를 fork하지 마세요

공식 원칙(CONTRIBUTING/AGENTS.md에 명시): **"Plugins, not loop changes: new behavior goes on documented extension points"**. 초보자가 가장 흔히 저지르는 실수는 코어 loop를 수정하는 것입니다 —— 올바른 방법은 확장 포인트를 사용하는 것입니다.

확인된 자주 쓰는 확장 포인트(이후 장에서 하나씩 실습):

| 확장 포인트 | 위치 | 용도 |
|---|---|---|
| `agent/request` waterfall | `agent-loop` | **매 모델 요청 전 설정 변경**(provider/model/reasoningEffort/tools) —— 속도 개선 플러그인 예제가 이것을 사용 |
| `agent/request-error` | `agent-loop` | 요청 실패 시 개입(공식 compaction 플러그인이 컨텍스트 오버플로우 복구에 이것을 사용) |
| `conversationEvents.register` | client runtime | 대화 이벤트 구독/주입(tool/call, turn/start 등) |
| `ctx.slots.inject` | client ui-slots | 인터페이스 슬롯에 UI 주입(예: turnTail에 산출물 파일 행 표시) |
| `settings` 서비스 | dsh-settings | 사용자가 설정 가능한 네임스페이스 등록(설정 페이지 자동 렌더링) |
| `ctx.provide` / `ctx.get` | cordis | 플러그인 간 서비스 제공 |

**사용법(`agent/request`를 예로)** —— 플러그인의 `apply(ctx)`에서 등록:

```ts
import type {} from '@deepseek-ai/dsh-agent'  // 타입 확장

export function apply(ctx: any) {
  ctx.on('agent/request', async (request, next) => {
    // 매 모델 요청 전에 트리거됨: provider / model / reasoningEffort / tools를 바꿀 수 있음
    request.reasoningEffort = request.tools?.length > 3 ? 'low' : request.reasoningEffort
    // 반드시 await next()를 하고 그 결과를 return해야 함(Promise이므로, 수정된 요청을 다음 단계로 넘겨줌)
    return await next(request)
  })
}
```

> 핵심: `agent/request`는 **waterfall**(체인)입니다 —— 모든 플러그인이 `await next()`를 하고 결과를 return해야 합니다; `await`를 빠뜨리거나 return을 하지 않으면 다운스트림이 당신의 수정 사항을 받지 못합니다(3.5절 흔한 트러블슈팅 참고).

## 3.5 흔한 트러블슈팅(실제 경험)

### 의존성 해석 트러블슈팅

| 트러블 | 증상 | 해결 | 스레드 |
|---|---|---|---|
| **rc.1 의존성 단절** | `pnpm install`이 `@deepseek-ai/dsh-type-meta@0.0.1-rc.1` 404 오류 | 공식 rc.1 시절 여러 패키지가 게시된 적이 없음; `^0.1.0-rc.6` 라인으로 업그레이드 | — |
| **전역 `core.hooksPath` 충돌** | 전역으로 `core.hooksPath`를 설정한 경우(예: Codex/Claude Code) → `pnpm install` 시 lefthook postinstall 실패 | 전역 hooksPath를 임시로 해제: `git config --global --unset core.hooksPath`, 설치 후 복원 | [#139](https://github.com/deepseek-ai/deepseek-harness/discussions/139) |
| **macOS 전역 설치가 플러그인을 해석하지 못함** | `pnpm add -g @deepseek-ai/dsh` 후 시작 시 약 88개 플러그인 패키지에 `ERR_MODULE_NOT_FOUND` 오류 | pnpm 전역 설치의 모듈 해석 전략이 npm과 다름; macOS에서는 `npm i -g`나 npx 권장 | [#204](https://github.com/deepseek-ai/deepseek-harness/discussions/204) |
| **koffi 사전 컴파일 손상(Windows)** | koffi 3.1.3/3.1.4의 `win32-x64` 사전 컴파일 바이너리가 손상됨 → 디렉터리 선택기 크래시/서비스가 조용히 멈춤 | koffi@3.1.2로 고정: `pnpm add koffi@3.1.2` 또는 `package.json`의 resolution 수정 | [#293](https://github.com/deepseek-ai/deepseek-harness/discussions/293) |
| **koffi 네이티브 크래시(Windows)** | 선택기 worker 크래시(`0xC0000005` 액세스 위반) 또는 시작 후 바로 멈춤 | 위와 동일하게 3.1.2로 고정; 그래도 크래시하면 STA 스레드 CoUninitialize 세그폴트인지 확인(커뮤니티 패치 존재) | [#197](https://github.com/deepseek-ai/deepseek-harness/discussions/197) |

### 플러그인 설치 형식 트러블슈팅

| 형식 | 작성법 | 주의사항 | 스레드 |
|---|---|---|---|
| **npm 패키지명** | `dsh plugin add dsh-better-sidebar` | npm에 이미 게시되어 있어야 함; rc 단계에서는 많은 커뮤니티 플러그인이 아직 게시되지 않음 | — |
| **GitHub 저장소** | `dsh plugin add github:작성자/저장소` | **의존성만 추가할 뿐, `cordis.patch.yml`에 자동으로 append되지 않음**, 장착 줄을 수동으로 추가해야 함 | [#656](https://github.com/deepseek-ai/deepseek-harness/discussions/656) |
| **로컬 경로** | `link:/path/to/plugin`(macOS/Linux)<br>`link:C:\path\to\plugin`(Windows) | 개발 디버깅용; 경로 슬래시는 플랫폼마다 다름 | — |
| **공백 포함 경로(Windows)** | `dsh plugin <...>`에 공백 포함 경로 전달 | Windows에서 공백 포함 경로가 내부 전달 시 끊어짐: `apps/cli/src/plugin.ts`의 `runPlugin`이 `spawnSync("pnpm", args, { shell: true })`를 사용 —— Node가 공백으로 인자를 이어붙이고 이스케이프하지 않음(Node 22+부터 DEP0190 경고), 호출자의 따옴표가 전달 레이어까지 도달하지 못함 | 수정 방향: `shell: false`(인자 배열을 직접 전달)이나 전달 전 직접 이스케이프 | [#1420](https://github.com/deepseek-ai/deepseek-harness/discussions/1420) |

> 커뮤니티 트러블슈팅 요약: `github:` 형식으로 플러그인을 설치한 후에는 `cordis.patch.yml`에 `- insert:` 장착 줄을 수동으로 추가하는 것을 잊지 마세요. 그렇지 않으면 플러그인 의존성은 설치되었지만 dsh가 로드하지 않습니다.

### 그 외 흔한 트러블슈팅

| 트러블 | 증상 | 해결 |
|---|---|---|
| 플러그인에 `main`이 없음 | `dsh: No "exports" main defined` | host 플러그인의 `package.json`은 `.` 엔트리를 노출해야 함(`"main": "src/index.ts"`는 tsx로 직접 로드 가능) |
| 이벤트 핸들러가 `await next()`를 빠뜨림 | 요청에서 provider/model이 유실되어 오류 발생 | `agent/request`의 `next()`는 **Promise**를 반환하므로, 반드시 await 후 spread해야 함 |
| `'agent/request' is not assignable to keyof Events` 타입 오류 | npm 타입이 공식 타입 확장을 re-export하지 않음 | 느슨한 시그니처 사용(`ctx.on as unknown as ...`)으로 경계에서 변환 |
| client 패키지 산출물이 `window.__ModuleLoader__`에 의존 | jsdom으로 client 테스트를 직접 실행 불가 | 컴포넌트 레벨 테스트에는 dsh web 런타임이 필요(공식 CI에서 실행) |

---

> **프리셋 장착 싱글턴 충돌(#1415/#1827 확인)**: Cordis toolset을 가진 프리셋을 장착할 때마다, `tool-cordis`는 프로세스 싱글턴인 Host inspect 레지스트리에 4개의 provider ID(`Service`/`Event`/`Builtin`/`Tool`)를 중복 등록합니다 —— 두 번째 장착 시 바로 `already registered`를 던지며, 세션 복원/여러 프리셋을 겹칠 때 이 문제가 발생합니다. 점검: 동일 프로세스 안에서 tool-cordis가 단 한 번만 장착되는지 확인하세요.

## 실습 문제(진짜로 이해했는지 검증)

1. **이해 문제**: 원문을 보지 않고 `profiles/web/` 디렉터리 아래의 파일 구조를 그리고, 각 파일의 역할을 말해보세요
   > 정답 확인: 본 장 3.1절 "profile 디렉터리는 이렇게 생겼습니다" 참고
2. **이해 문제**: "host 반쪽"과 "client 반쪽"이 각각 어디서 실행되고 무엇을 담당하는지 설명하세요. 플러그인이 host 반쪽만 가질 수 있나요?
   > 정답 확인: 본 장 3.3절 표 참고
3. **실습 문제**: 여러분의 `~/.dsh/profiles/web/package.json`을 열어 `dsh.profile.bundles` 필드를 찾고, bundle의 로딩 순서를 말해보세요
   > 정답 확인: 본 장 3.1절 "로딩 순서" 단락 참고
4. **실습 문제**: `dsh-my-widget`이라는 이름의 플러그인을 장착한다고 가정하고, `package.json`과 `cordis.patch.yml`에 각각 무엇을 추가해야 하는지 작성해보세요
   > 정답 확인: 본 장 3.2절의 두 곳 수정 예시 참고
5. **실습 문제**: `cordis.patch.yml`에 잘못된 장착 줄을 하나 추가하고(예: 플러그인 이름 오타), dsh web을 재시작해 오류 메시지를 관찰하고 기록하세요
   > 정답 확인: 본 장 3.5절 "흔한 트러블슈팅" 표와 비교
6. **사고 문제**: 공식 원칙은 "Plugins, not loop changes"라고 말합니다. dsh의 "모델 응답 후 자동 요약" 동작을 바꾸고 싶다면 어떤 확장 포인트를 써야 할까요? 왜 agent-loop 소스 코드를 직접 수정하면 안 될까요?
   > 정답 확인: 본 장 3.4절 확장 포인트 표 + "동작을 바꾸려면 훅을 먼저 찾을 것" 원칙 참고

## 자주 묻는 질문 FAQ

**Q1: profile과 bundle은 무엇이 다른가요?**
profile은 "실행 가능한 형태 하나"(예: web, headless)이고, bundle은 "미리 설정된 플러그인 집합 하나"입니다. profile은 `dsh.profile.bundles`로 어떤 bundle을 로드할지 지정한 다음, 자신의 patch를 덧붙입니다. 간단히 말하면: bundle은 블록 키트이고, profile은 조립된 완성품입니다.

**Q2: `cordis.patch.yml`의 `- insert:`는 무슨 의미인가요? 다른 작업도 있나요?**
`insert`는 "플러그인 체인에 새 플러그인을 삽입한다"는 뜻입니다. patch 파일은 cordis의 설정 병합 메커니즘을 기반으로 하며, insert(삽입), override(기존 플러그인 설정 덮어쓰기) 등의 작업을 지원합니다. 초보자는 insert만 익혀도 대부분의 플러그인을 장착할 수 있습니다.

**Q3: 플러그인 상태는 profile 단위로 격리되나요, 세션 단위로 격리되나요?**
장착은 profile 레벨입니다(한 번 설치하면 해당 profile의 모든 세션에서 사용 가능). 하지만 **상태는 세션 레벨**입니다. 예를 들어 `better-sidebar`의 탭 레이아웃은 세션별로 영속화되어, 세션끼리 서로 간섭하지 않습니다.

**Q4: client 반쪽만 있는(순수 UI) 플러그인을 작성하고 싶은데, 가능한가요?**
가능합니다. `package.json`에 `dsh.client` + `exports["./client"]`를 선언하고, `exports["."]`는 작성하지 않으면 됩니다. 단, client 반쪽은 파일 시스템에 직접 접근하거나 명령을 실행할 수 없으므로, host 반쪽의 서비스(`ctx.provide`/`ctx.get`)를 통해 연결해야 합니다.

**Q5: `agent/request` waterfall과 `conversationEvents.register`는 무엇이 다른가요? 어떤 것을 써야 하나요?**
`agent/request`는 "모델 요청 설정을 바꾸는" 훅입니다(provider/model/tools/reasoningEffort), 매 요청 전에 트리거되어 새 설정을 반환합니다. `conversationEvents.register`는 "대화 이벤트를 구독/주입하는" 훅입니다(tool/call, turn/start 등), 이벤트 스트림을 감시하거나 주입하는 데 사용합니다. 요청 파라미터를 바꾸고 싶다면 전자를, 대화 흐름을 감시하고 싶다면 후자를 사용하세요.

**Q6: 왜 제 플러그인이 설치됐는데 반응이 없나요? 어떻게 디버깅하나요?**
세 단계로 점검하세요: ① `pnpm install`에 오류가 없고 `node_modules`에 여러분의 플러그인이 있는지 확인; ② `cordis.patch.yml`의 id/name 철자가 정확한지 확인; ③ `dsh web`을 재시작한 후 프로세스 로그에 플러그인 로딩 정보가 있는지 확인. host 반쪽에 `console.log`가 있다면 로그에서 출력을 볼 수 있어야 합니다.

---

**다음 장**: [제 4 장: 플러그인 개발 실전](./04-plugin-dev.ko.md) —— 처음부터 첫 host 플러그인을 작성합니다.
