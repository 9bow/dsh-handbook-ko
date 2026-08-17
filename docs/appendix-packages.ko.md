# 부록 B: 공식 패키지 빠른 참조 총정리(@deepseek-ai/*)

> **이 목록은 「알려진 패키지」 목록입니다**: 백서 본문에서 실측/언급되었고 npm registry에서 확인 가능한 공식 패키지(`@deepseek-ai/dsh-*`와 `@deepseek-ai/cordis`)를 수록합니다. 패키지 설명은 `npm view` 실측 결과에서 가져왔습니다. dsh는 `0.1.0-rc` 빠른 반복 시기이므로 **패키지 이름과 설명은 버전에 따라 업데이트됩니다** —— 공식 저장소 `packages/AGENTS.md`나 npm 최신 설명과 다르다면 공식을 기준으로 삼으세요. 확인하지 못한 패키지는 「(추후 보완)」으로 표시했으며, 패키지 이름을 지어내지 않았습니다.
>
> **패키지 수 기준**: 이 목록은 npm으로 확인된 **핵심 패키지 33개**를 수록합니다(CLI 직접 의존성 53개 + 계열 전체 게시 수 221개, [제8장 8.1절](./08-tools-context.ko.md) 설명 참고); 백서/공식이 말하는 「60+ 능력 패키지」는 저장소 `packages/` 아래의 모든 하위 디렉터리를 가리키며, 전체 목록은 공식 저장소 `packages/README.md`(47개 그룹 공식 표)에서 확인할 수 있습니다.
>
> 패키지 이름과 저장소 레이아웃의 대응 관계: npm 패키지 `@deepseek-ai/dsh-xxx` ≈ 공식 저장소 `packages/<group>/xxx`(예: `dsh-tool-fs` ↔ `fs/tool-fs`).
>
> 검증 방법: `npm view @deepseek-ai/<name> description`(2026-08 스냅샷).

**능력 영역 값**: `도구`(모델이 호출 가능한 도구/실행 파이프라인) · `컨텍스트`(요청 컨텍스트 조립과 압축) · `세션`(세션 영속화/제목/텔레메트리) · `서브에이전트`(하위 작업 위임) · `MCP`(외부 도구 서버 연동) · `워크플로우`(다단계 오케스트레이션) · `보안`(샌드박스/승인/루프 위생) · `LLM`(모델 연동과 전략) · `UI`(브라우저 인터페이스와 클라이언트 서비스) · `핵심`(CLI/골격 bundle, 위 능력 영역에 속하지 않음).

---

## 핵심(Core): CLI와 골격 bundle

dsh의 "기반": `@deepseek-ai/dsh` 하나만 설치하면 바로 실행되며, profile은 내장 bundle 스택으로 쌓입니다(`dsh-base` → `dsh-headless` / `dsh-web-app`).

| 패키지 이름 | 용도 | 능력 영역 |
|---|---|---|
| `@deepseek-ai/dsh` | dsh CLI: profile 시작, 플러그인 관리, 브라우저 UI 별칭(`npx -y @deepseek-ai/dsh web`) | 핵심 |
| `@deepseek-ai/dsh-base` | 공유 dsh 코어 profile bundle: 각 profile의 첫 번째 패치 레이어로, 빈 profile 루트에 기본 플러그인 줄을 삽입 | 핵심 |
| `@deepseek-ai/dsh-headless` | headless 일회성 bundle: Host/HTTP/브라우저 레이어 없는 직접적인 core Agent/Session 실행기 | 핵심 |
| `@deepseek-ai/dsh-agent` | Agent 인터페이스, 레지스트리, 발신자 스코프, 이벤트 어휘(플러그인 개발이 직접 의존하는 경우 많음) | 핵심 |
| `@deepseek-ai/dsh-web-app` | dsh 브라우저 쪽 bundle: web patch 레이어(프론트엔드 dist 서빙, web 쪽 프롬프트, bash 런타임 변수, URL 행) | 핵심 |
| `@deepseek-ai/cordis` | 하부 메타 프레임워크: 현대적 JavaScript 앱을 위한 의존성 주입/이벤트/생명주기 컨테이너(dsh의 플러그인 기반) | 핵심 |

## 도구(Tools): 모델이 호출 가능한 능력

모델이 실제로 보는 도구 이름은 **짧은 동사**입니다(`read`/`write`/`grep`/`glob`/`edit`/`bash`/`web_search`/`todo_write`…), 하부는 이 그룹의 패키지들이 구현을 제공합니다.

| 패키지 이름 | 용도 | 능력 영역 |
|---|---|---|
| `@deepseek-ai/dsh-tools` | 도구 레지스트리와 실행 파이프라인(tool registry + execution pipeline) | 도구 |
| `@deepseek-ai/dsh-fs` | 추상 파일 시스템 능력 seam(`ctx.fs`): 어휘 타입, FileSystem 서비스(텍스트 IO + 선택적 버전 가드 원자적 쓰기), `fs/*` 정책 이벤트 어휘 | 도구 |
| `@deepseek-ai/dsh-tool-fs` | 모델 쪽 파일 시스템 도구(`read`, `write`, `edit`), `ctx.fs` 기반 | 도구 |
| `@deepseek-ai/dsh-tool-fs-search` | 모델 쪽 파일 탐색 도구(`glob`, `grep`), 번들된 ripgrep 바이너리 내장 | 도구 |
| `@deepseek-ai/dsh-tool-str-replace-editor` | 모델 쪽 편집 도구: 보기, 생성, 리터럴 교체, 줄 삽입(도구 결과에 `locations` 포함 → 산출물 추적) | 도구 |
| `@deepseek-ai/dsh-shell` | 추상 bash 실행기 seam(`ctx.shell`) | 도구 |
| `@deepseek-ai/dsh-tool-bash` | 모델 쪽 bash 도구, 선택적으로 범용 백그라운드 작업과 샌드박스 업그레이드 지원 | 도구 |
| `@deepseek-ai/dsh-web` | 추상 web 접근 능력 seam(`ctx.web`): search/fetch 프로바이더 레지스트리, 등록 순서와 무관한 선택, 요청/결과 어휘, WebError 분류 | 도구 |
| `@deepseek-ai/dsh-tool-web` | 모델 쪽 web 도구(`web_search`, `web_fetch`), `ctx.web` 기반 | 도구 |
| `@deepseek-ai/dsh-tool-todo` | 모델 쪽 할 일 도구 `todo_write`, 이벤트 소싱 세션 로그 기반 | 도구 |
| `@deepseek-ai/dsh-tool-skill` | 모델 쪽 스킬 로딩 도구(skill 호출 엔트리) | 도구 |

## 컨텍스트(Context): 컨텍스트 조립과 압축

"무엇을 담을지"(시스템 프롬프트 + 스킬 카탈로그 + 이력 + 도구 스키마)와 "담을 수 없을 때 어떻게 압축할지".

| 패키지 이름 | 용도 | 능력 영역 |
|---|---|---|
| `@deepseek-ai/dsh-system-prompt` | 시스템 프롬프트 조립 레지스트리(공식 플러그인이 `systemPrompt.section()`으로 프롬프트 조각을 등록) | 컨텍스트 |
| `@deepseek-ai/dsh-compaction` | 추상 압축 서비스 seam(`ctx.compaction`) | 컨텍스트 |
| `@deepseek-ai/dsh-compaction-basic` | 토큰 계량 기반 압축 전략 + LLM 요약 백엔드(오버플로우 감지 → 다듬기 → 요약) | 컨텍스트 |
| `@deepseek-ai/dsh-skill` | Agent 스킬 프로바이더 레지스트리(스킬 카탈로그가 컨텍스트에 주입되어 모델이 필요에 따라 호출) | 컨텍스트 |

## 세션(Session): 영속화, 제목, 텔레메트리

| 패키지 이름 | 용도 | 능력 영역 |
|---|---|---|
| `@deepseek-ai/dsh-session` | 이벤트 소싱 세션 저장소(event-sourced session store) | 세션 |
| `@deepseek-ai/dsh-session-title` | 로그 기반 세션 제목 서비스와 프로바이더 레지스트리 | 세션 |
| `@deepseek-ai/dsh-session-telemetry` | 텔레메트리 seam: 세션 이벤트 캡처, 프로젝션, 마스킹, 리포팅 백엔드로 전달 | 세션 |

## 서브에이전트(Subagent): 하위 작업 위임

| 패키지 이름 | 용도 | 능력 영역 |
|---|---|---|
| `@deepseek-ai/dsh-subagent` | 추상 서브에이전트 seam(`ctx.subagents`): 서브에이전트를 위임하는 명명된 프로바이더 레지스트리 | 서브에이전트 |

## MCP: 외부 도구 서버 연동

| 패키지 이름 | 용도 | 능력 영역 |
|---|---|---|
| `@deepseek-ai/dsh-mcp-client` | MCP 클라이언트 브릿지: MCP 서버에 연결해 그 도구를 `ctx.tools`에 등록 | MCP |

## 워크플로우(Workflow): 다단계 결정론적 오케스트레이션

| 패키지 이름 | 용도 | 능력 영역 |
|---|---|---|
| `@deepseek-ai/dsh-workflow` | 워크플로우 능력 seam: `ctx.workflows` 서비스, run 어휘, `workflow/*` 이벤트 | 워크플로우 |

## 보안(Safety): 샌드박스, 승인, 루프 위생

| 패키지 이름 | 용도 | 능력 영역 |
|---|---|---|
| `@deepseek-ai/dsh-sandbox` | 추상 프로세스 샌드박스 seam(`ctx.sandbox`): 동일 세계 격리 어휘 + SandboxProvider 계약 | 보안 |
| `guard/*` | 루프 위생과 도구 타임아웃(백서 8.5절 언급; npm의 구체적 패키지 이름은 추후 보완) | 보안 |
| `interaction/*` | 권한/승인(위험한 작업 확인; npm의 구체적 패키지 이름은 추후 보완) | 보안 |

## LLM: 모델 연동과 전략

| 패키지 이름 | 용도 | 능력 영역 |
|---|---|---|
| `@deepseek-ai/dsh-llm` | Provider 독립적인 LLM 서비스 인터페이스 | LLM |
| `@deepseek-ai/dsh-llm-deepseek` | DeepSeek chat-completions 어댑터(기본 `provider=deepseek-official`, off/high/max 강도 수용) | LLM |
| `@deepseek-ai/dsh-llm-retry` | Provider 라우팅되는 LLM 요청 재시도 전략 | LLM |

## UI: 브라우저 인터페이스와 클라이언트 서비스

| 패키지 이름 | 용도 | 능력 영역 |
|---|---|---|
| `@deepseek-ai/dsh-client-runtime` | 클라이언트 코어 서비스: SlotsService, SessionsService(스코프 트리 + 객체 레이어) | UI |
| `@deepseek-ai/dsh-client-locale` | 언어팩 플러그인: Host 쪽 zh/en 선호도, 브라우저 쪽 폴백, 언어 스냅샷, 타입이 있는 네임스페이스 딕셔너리 | UI |
| `ui-conversation` / `ui-tool` 등 | Web UI 각 구성요소(백서 8.1절에서 `client/*` 언급; npm의 구체적 패키지 이름은 추후 보완) | UI |

---

## 이름은 알려졌지만 npm에 게시되지 않은 패키지(과거 404 패키지)

아래 패키지 이름은 백서의 트러블슈팅 기록에만 존재하며, **npm에서 설치할 수 없습니다**(rc.1 시대 의존성 단절의 근본 원인), 404를 만나는 것이 정상입니다:

| 패키지 이름 | 설명 |
|---|---|
| `@deepseek-ai/dsh-pty` | rc.1 시대 의존성, 게시된 적 없음(`pnpm dlx` 404의 근본 원인, 제2장 FAQ 참고) |
| `@deepseek-ai/dsh-type-meta` | rc.1 시대 의존성, 게시된 적 없음(rc.1 의존성 단절의 근본 원인, 제3장 3.5절 참고) |

> 회피법: 의존성은 통일해서 `^0.1.0-rc.6` 라인을 사용하세요.

---

**관련 챕터**: [제8장 · 도구와 컨텍스트 시스템](./08-tools-context.ko.md)(60+ 능력 패키지 지도) · [제9장 · MCP·서브에이전트·워크플로우](./09-mcp-subagent-workflow.ko.md) · [용어집과 명령어 빠른 참조](./appendix-glossary.ko.md)
