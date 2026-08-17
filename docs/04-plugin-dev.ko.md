# 제 4 장: 플러그인 개발 실전

> 본 장의 목표: 처음부터 **실제로 사용 가능한 host 플러그인**을 작성합니다 —— `agent/request` 확장 포인트를 통해 추론 강도를 자동으로 조절합니다. 이것은 속도 개선 플러그인의 완전한 분석이며, 모든 코드는 실행 가능하고 테스트 가능합니다.

## TL;DR(본 장 핵심, 30초 버전)

1. **핵심 문제**: dsh는 매 도구 호출 전마다 모델이 다시 사고하며, 50단계 작업에서는 90%+ 시간이 사고에 쓰입니다 —— 강도를 낮추는 것이 가장 빠른 속도 개선책입니다
2. **3층 검증**: 순수 함수 단위 테스트(결정이 올바른가) → waterfall 계약 테스트(연결이 올바른가) → 실기 검증(실제 루프에서 작동하는가)
3. **`agent/request` waterfall**: 매 모델 요청 전에 트리거되고, 리스너의 반환값이 다음 리스너에게 전달되어 "원본 설정 유지 + 특정 필드 오버라이드"를 구현합니다
4. **`next()`는 Promise이므로 반드시 await**: await 없이 바로 spread하면 빈 객체를 얻게 되어 provider/model이 유실되고 오류가 발생합니다
5. **개발 원칙**: 확장 포인트를 먼저 찾을 것(90%의 동작에 공식 훅이 있음), 로직을 순수 함수로 분리할 것, 실기 검증은 생략 불가

<details><summary>본 장 목차</summary>
- [4.1 우리가 만들 것](#41-우리가-만들-것)
- [4.2 프로젝트 골격](#42-프로젝트-골격)
- [4.3 순수 함수: 결정 로직(의존성 없음, 단위 테스트 가능)](#43-순수-함수-결정-로직의존성-없음-단위-테스트-가능)
- [4.4 플러그인 본체: `agent/request` waterfall에 연결](#44-플러그인-본체-agentrequest-waterfall에-연결)
- [4.5 테스트](#45-테스트)
- [4.6 초보자를 위한 3가지 개발 원칙](#46-초보자를-위한-3가지-개발-원칙)
</details>

## 4.1 우리가 만들 것

**문제**: dsh는 매 도구 호출 전마다 모델이 다시 사고합니다(`reasoning_effort`). 50단계 도구 체인 작업에서는 "사고"가 벽시계 시간의 90%+ 를 차지합니다.

**해결책**: `agent/request` waterfall을 감시하는 host 플러그인을 만들어, 현재 단계의 최근 도구 호출을 기준으로 단순한 라운드의 `reasoning_effort`를 `high`에서 `low`로 낮춥니다.

## 4.2 프로젝트 골격

```text
dsh-speed-plugin/
├── package.json          # host 플러그인 선언
├── tsconfig.json
├── src/
│   ├── effort-decision.ts  # 순수 함수: 결정 로직(의존성 없음, 단위 테스트 가능)
│   └── index.ts            # apply(ctx): 확장 포인트에 연결
└── tests/
    └── effort-decision.spec.ts
```

`package.json`의 핵심 필드:

```json
{
  "name": "dsh-speed-plugin",
  "type": "module",
  "main": "src/index.ts",
  "exports": {
    ".": { "types": "./src/index.ts", "default": "./src/index.ts" }
  },
  "peerDependencies": {
    "@deepseek-ai/cordis": "^4.0.1",
    "@deepseek-ai/dsh-agent": "^0.1.0-rc.6"
  }
}
```

> ⚠️ 의존성 버전은 반드시 `^0.1.0-rc.6` 라인을 사용하세요 —— rc.1 라인의 npm 의존성 체인은 끊어져 있습니다(제3장 흔한 트러블슈팅 참고).

## 4.3 순수 함수: 결정 로직(의존성 없음, 단위 테스트 가능)

`src/effort-decision.ts`:

```ts
export type EffortId = 'low' | 'high' | 'max'

export interface ToolCallSample {
  name: string      // 도구 이름, 예: 'write', 'read', 'bash'
  argsSize: number  // 파라미터 크기(문자 수)
}

export interface EffortDecisionInput {
  recentCalls: readonly ToolCallSample[]
  selected: EffortId      // 사용자 기준 강도
  allowDowngrade: boolean
  allowUpgrade: boolean
}

const SIMPLE_TOOL_RE = /^(fs|bash|terminal|read|write|grep|glob|edit|ls|cat|rm|cp|touch|mkdir|pwd)/i
const HEAVY_ARGS = 800

export function decideEffort(input: EffortDecisionInput): EffortId {
  const { recentCalls, selected, allowDowngrade, allowUpgrade } = input
  if (recentCalls.length === 0) return selected   // 완전히 새 프롬프트: 기준값 유지

  const ratio = recentCalls.filter(c =>
    SIMPLE_TOOL_RE.test(c.name) && c.argsSize < HEAVY_ARGS,
  ).length / recentCalls.length
  const heaviest = recentCalls.reduce((m, c) => Math.max(m, c.argsSize), 0)

  if (ratio >= 0.75 && allowDowngrade) return 'low'
  if (heaviest >= HEAVY_ARGS * 4 && allowUpgrade) return 'max'
  if (ratio < 0.75) return allowUpgrade ? 'high' : selected
  return selected
}
```

**왜 순수 함수로 분리했는가**: 결정 로직을 dsh 런타임에서 분리했습니다 —— 단위 테스트는 의존성 없이 밀리초 단위로 실행되며 모든 분기를 커버할 수 있고, 실기에서는 "주입이 실제로 일어났는지"만 검증하면 됩니다.

## 4.4 플러그인 본체: `agent/request` waterfall에 연결

`src/index.ts`:

```ts
import type { Context } from '@deepseek-ai/cordis'
import { decideEffort, type ToolCallSample } from './effort-decision.ts'

export interface SpeedPluginConfig {
  enabled: boolean
  allowDowngrade: boolean
  allowUpgrade: boolean
  baseline: 'low' | 'high' | 'max'
}

export const DEFAULT_CONFIG: SpeedPluginConfig = {
  enabled: true, allowDowngrade: true, allowUpgrade: false, baseline: 'high',
}

const WINDOW = 8

function recentToolCalls(agent: unknown): ToolCallSample[] {
  const events = (agent as { session?: { events?: readonly unknown[] } }).session?.events ?? []
  const out: ToolCallSample[] = []
  for (let i = events.length - 1; i >= 0 && out.length < WINDOW; i--) {
    const e = events[i] as { type?: string; data?: { name?: string; arguments?: unknown } } | undefined
    if (e?.type !== 'tool/call') continue
    out.push({
      name: e.data?.name ?? 'tool',
      argsSize: typeof e.data?.arguments === 'string' ? e.data.arguments.length : 0,
    })
  }
  return out.reverse()
}

export function apply(ctx: Context, config: SpeedPluginConfig = DEFAULT_CONFIG): void {
  if (!config.enabled) return

  // 경계 어댑팅: npm 패키지가 공식 이벤트 타입 확장을 re-export하지 않아서, 여기서 시그니처를 느슨하게 처리(제3장 흔한 트러블슈팅)
  const on = ctx.on as unknown as (
    event: string,
    handler: (payload: Record<string, unknown>, next: () => unknown) => unknown | Promise<unknown>,
  ) => void

  on('agent/request', async (payload, next) => {
    const seed = await next() as { reasoningEffort?: unknown }   // ⚠️ 반드시 await!
    const calls = recentToolCalls(payload.agent)
    const effort = decideEffort({
      recentCalls: calls,
      selected: config.baseline,
      allowDowngrade: config.allowDowngrade,
      allowUpgrade: config.allowUpgrade,
    })
    console.log(`[speed-plugin] calls=${JSON.stringify(calls)} => reasoningEffort=${effort}`)
    return { ...seed, reasoningEffort: effort }
  })
}
```

**세 가지 핵심 포인트**(전부 실제로 겪은 트러블슈팅):

1. **`next()`는 Promise입니다**: `await next()`로 현재 설정을 가져옵니다; await 없이 바로 spread하면 빈 객체를 얻게 되어 → provider/model이 유실 → 오류가 발생합니다.
2. **waterfall 시맨틱**: 리스너의 **반환값**이 다음 리스너/최종 요청에 전달됩니다. `{...seed, reasoningEffort}`를 반환하는 것이 바로 "원본 설정 유지 + 추론 강도 오버라이드"입니다.
3. **`agent/request`는 매 단계마다 트리거됩니다**: `agent-loop`의 `buildRequest`는 매 단계마다 이 waterfall을 거칩니다 —— 그래서 동적 결정이 자연스럽게 단계별로 적용됩니다.

## 4.5 테스트

플러그인 테스트는 "모듈이 로드되는가"에서 멈추면 안 됩니다. 3층 검증을 권장합니다: 순수 함수 단위 테스트는 비즈니스 규칙을, waterfall 계약 테스트는 런타임 경계를, 실기 검증은 완전한 agent 루프를 담당합니다.

### 1층: 순수 함수 단위 테스트

순수 함수 테스트는 런타임 의존성이 없고 실행이 빠르며, 모든 결정 분기를 커버하기에 적합합니다:

```ts
import { describe, expect, it } from 'vitest'
import { decideEffort } from '../src/effort-decision.ts'

it('downgrades to low for simple tool chains', () => {
  expect(decideEffort({
    recentCalls: [{ name: 'write', argsSize: 40 }],
    selected: 'high', allowDowngrade: true, allowUpgrade: true,
  })).toBe('low')
})
// ... 더 많은 분기: 완전히 새 프롬프트는 기준값 유지 / 다운그레이드 비활성화 / 초대형 페이로드는 max로 업그레이드 / 혼합 도구는 high로 업그레이드
```

### 2층: waterfall 계약 테스트

순수 함수가 테스트를 통과했다고 해서 플러그인의 연결이 올바르다는 뜻은 아닙니다. 가장 흔한 숨은 오류는 `await next()`를 빠뜨리는 것으로, 이는 상위의 `provider`, `model`, `tools` 등 필드를 조용히 유실시킵니다. 계약 테스트는 최소한의 `Context` 대역으로 리스너를 캡처하며, API Key도 필요 없고 dsh를 시작할 필요도 없습니다:

```ts
it('awaits next, preserves the seed, and only overrides reasoning effort', async () => {
  const next = async () => ({
    provider: 'deepseek-official',
    model: 'deepseek-reasoner',
    reasoningEffort: 'max',
    tools: ['read', 'write'],
  })

  const result = await runRegisteredHandler({ agent: { session: { events: [] } } }, next)

  expect(result).toEqual({
    provider: 'deepseek-official',
    model: 'deepseek-reasoner',
    reasoningEffort: 'high',
    tools: ['read', 'write'],
  })
})
```

함께 제공되는 템플릿의 [`tests/plugin.spec.ts`](../examples/plugin-template/tests/plugin.spec.ts)는 다음도 커버합니다:

- `agent/request` 리스너를 단 하나만 등록하는지;
- 최근 8개의 `tool/call`만 읽고 다른 이벤트는 무시하는지;
- 다운스트림 waterfall이 오류를 던질 때 그대로 위로 전파되는지, 조용히 삼키지 않는지.

전체 자동 테스트 실행:

```bash
cd examples/plugin-template
npm install
npm test
npm run typecheck
```

### 3층: 실제 agent 루프 검증

계약 테스트는 플러그인의 경계가 예상대로 작동함을 증명하지만, 실제 런타임을 대체할 수는 없습니다. 플러그인을 장착하고(제3장 방법) → `dsh web` 재시작 → 파일 생성 작업 하나를 보내고 → dsh 프로세스 로그를 관찰합니다:

```text
[speed-plugin] agent/request: calls=[]                    => reasoningEffort=high
[speed-plugin] agent/request: calls=[{"name":"write",…}] => reasoningEffort=low
```

첫 번째 라운드는 도구 호출이 없어 → 기준값 `high`를 유지; `write` 도구가 감지되면 → 다음 라운드에서 `low`로 낮춰집니다. **주입 경로가 완전히 작동합니다.**

이 계층을 CI에 넣고 싶다면 공식 디스커션 [#462: API Key 없이 waterfall 동작 검증하기](https://github.com/deepseek-ai/deepseek-harness/discussions/462)를 참고하세요: mock LLM으로 고정된 도구 호출을 생성하고, headless profile을 통해 실제 agent loop를 실행한 다음, 감사 스냅샷으로 예상 waterfall과 `tools/result`를 확인합니다. 디스커션의 명령어는 특정 rc/master 소스 코드를 기반으로 하므로, 재사용할 때는 목표 버전의 스크립트와 이벤트 목록을 기준으로 삼아야 하며, 고정된 이벤트 개수를 버전 간 단언으로 삼지 마세요.

> 완전히 실행 가능한 코드: [`examples/plugin-template/`](../examples/plugin-template/) 참고.

> **⚠️ 강도 지원은 어댑터/모델마다 다릅니다(실제 트러블슈팅)**: `decideEffort`가 `low`를 반환한 후, 현재 provider의 어댑터 능력 표가 `low`를 지원하지 않으면(예: `deepseek-official` 어댑터는 `off`/`high`/`max`만 지원), 요청이 `does not support reasoning effort "low"` 오류를 던집니다 —— **이것은 어댑터 공백이지, 플러그인 버그가 아닙니다**(공식 API는 실제로 low를 지원합니다, api-docs.deepseek.com/guides/thinking_mode/ 참고; FAQ Q4에 강도 설명 있음).
>
> **모델 능력 표에 맞춰 어댑팅하는 작성법**: 다운그레이드 목표를 먼저 어댑터 지원 여부로 확인하고, 사용 가능한 강도로 매핑합니다:
> ```ts
> const SUPPORTED = ['off', 'high', 'max']  // 현재 어댑터 능력 표 기준
> const effort = decideEffort({...})        // 플러그인이 원하는 강도
> const final = SUPPORTED.includes(effort) ? effort : (effort === 'low' ? 'high' : effort)  // low가 지원되지 않으면 high로 폴백
> ```

## 4.6 초보자를 위한 3가지 개발 원칙

1. **확장 포인트를 먼저 찾을 것**: 바꾸고 싶은 동작의 90%는 공식 훅이 있습니다(`agent/request`, `settings`, `conversationEvents`, `slots`) —— 코어를 fork하지 마세요.
2. **로직을 순수 함수로 분리할 것**: 결정/계산 로직을 dsh에서 분리 → 단위 테스트가 밀리초 단위, 모든 분기 커버.
3. **경계와 실기 모두 검증할 것**: 계약 테스트는 `next()` 전달과 필드 보존을 증명하고, 실기 로그는 실제 agent loop가 작동함을 증명합니다.

> 📚 **공식 cookbook 추가 참고자료**(2026-08 공식 신규 추가, 공식 저장소 `docs/cookbook/`): `adding-a-package.md`(패키지 추가법), `adding-a-tool.md`(도구 추가법), `adding-a-conversation-node.md`(대화 노드 추가), `adding-an-llm-adapter.md`(LLM 어댑터 작성), `adding-a-vendored-package.md`(vendor 패키지), `extension-cookbook.md`(확장 포인트 모음). 본 장은 "최소 속도 개선 플러그인" 경로를 다뤘고, 공식 cookbook은 더 많은 확장 포인트 유형을 커버하니 심화 학습 시 대조하며 읽으세요.

---

## 실습 문제(진짜로 이해했는지 검증)

1. **이해 문제**: 원문을 보지 않고 이 속도 개선 플러그인의 3층 아키텍처(순수 함수 레이어 / 플러그인 본체 레이어 / 테스트 레이어)가 각각 무엇을 담당하는지 말해보세요
   > 정답 확인: 본 장 4.2-4.5절의 파일 구조 참고
2. **이해 문제**: `decideEffort`를 왜 `apply(ctx)` 안에 직접 로직을 쓰지 않고 순수 함수로 설계했는지 설명하세요. 결정 로직이 `ctx.session`에 의존했다면 여전히 단위 테스트가 가능할까요?
   > 정답 확인: 본 장 4.3절 "왜 순수 함수로 분리했는가" 단락 참고
3. **실습 문제**: `decideEffort`에 새 테스트 케이스를 작성해보세요: `recentCalls`에 단순 도구 3개 + 초대형 파라미터(`argsSize = 5000`) 1개가 있을 때 어떤 강도를 반환해야 할까요? 테스트 코드를 작성하고 실행해보세요
   > 정답 확인: 본 장 4.5절 단위 테스트 예시 참고. **주의**: 단순 3개 + 초대형 1개 = 총 4개 호출, ratio = 0.75로 **정확히 첫 번째 분기에 해당**(`ratio >= 0.75 && allowDowngrade`) → `low` 반환(`allowDowngrade`에 따라 달라지며, `allowUpgrade` 분기는 실행되지 않음)
4. **실습 문제**: `apply(ctx)` 안에서 `await next()`를 `const seed = next()`(await 없이)로 바꾸면 무슨 일이 일어날까요? 추론을 작성한 다음 실기에서 검증해보세요
   > 정답 확인: 본 장 4.4절 "세 가지 핵심 포인트"의 1번 참고
5. **실습 문제**: 비슷한 플러그인을 작성한다고 가정하되, "현재 세션의 도구 호출 총 횟수"에 따라 강도를 결정하도록 바꿔보세요(20회를 초과하면 자동으로 low로 낮춤). 순수 함수 시그니처와 핵심 로직을 작성해보세요
   > 정답 확인: 본 장 4.3절 순수 함수 설계 패턴 참고, 핵심은 입출력 타입 정의
6. **사고 문제**: `agent/request` waterfall에는 여러 리스너가 있을 수 있습니다. 속도 개선 플러그인과 다른 플러그인이 둘 다 `agent/request`를 감시한다면, 누가 먼저 실행될까요? 반환값은 어떻게 전달될까요?
   > 정답 확인: 본 장 4.4절 "waterfall 시맨틱" 단락 + 제3장 3.4절 확장 포인트 설명 참고

## 자주 묻는 질문 FAQ

**Q1: `next()`가 반환하는 seed에는 정확히 어떤 필드가 들어있나요?**
seed는 현재 waterfall 체인의 상류에서 누적된 요청 설정으로, 보통 `provider`, `model`, `reasoningEffort`, `tools` 등의 필드를 포함합니다. 여러분의 리스너가 seed를 받으면, spread로 바꾸고 싶은 필드를 오버라이드하고 나머지는 그대로 전달합니다. 구체적인 필드는 공식 `agent-loop` 패키지의 타입 정의를 기준으로 하세요(rc 단계에서 변경될 수 있음).

**Q2: 왜 npm 패키지의 타입 시그니처를 "느슨하게" 처리해야 하나요? 공식 타입을 바로 쓸 수 없나요?**
npm에 게시된 `@deepseek-ai/dsh-agent` 등의 패키지가 내부 이벤트 타입 확장을 re-export하지 않기 때문입니다(`Events` 인터페이스가 module augmentation으로 확장되지 않음). `ctx.on('agent/request', ...)`을 직접 쓰면 타입 오류가 발생합니다. 해결책은 경계에서 `as unknown as`로 변환하는 것이며, 이는 rc 단계의 임시 방편이고 정식 버전에서는 수정될 수 있습니다.

**Q3: 제 플러그인이 매 도구 호출 후마다 결정을 트리거해야 하는데, `agent/request`로 충분한가요?**
충분합니다. `agent/request`는 매 모델 요청 전에 트리거됩니다(도구 호출 후 다음 라운드 요청도 포함). 리스너 안에서 최근 도구 호출 이력을 읽기만 하면 됩니다(`payload.agent.session.events`에서 역순으로 `tool/call` 이벤트를 찾음), 이렇게 하면 단계별 결정이 가능합니다.

**Q4: 순수 함수 테스트도 통과했고 실기 검증도 통과했는데, 사용자가 "가끔 효과가 없다"고 피드백한다면 어떤 이유일 수 있나요?**
몇 가지 가능성이 있습니다: ① 사용자의 `settings.yaml`에서 `reasoningEffort`가 이미 `low`라서 다운그레이드 효과가 없음(이미 최저치); ② 사용자의 작업이 전부 단순한 도구라서 원래도 빨라서 차이를 체감하지 못함; ③ waterfall 안의 다른 플러그인이 여러분의 반환값을 덮어씀. 각 단계의 입력/출력 강도를 로그로 남길 것을 권장합니다.

**Q5: 플러그인에 사용자가 설정할 수 있는 스위치(설정 페이지에서 켜고 끌 수 있는)를 추가하고 싶은데, 어떻게 하나요?**
`settings` 서비스로 네임스페이스(예: `speed-plugin`)를 등록하고, 설정 항목(enabled, baseline 등)을 선언하세요. dsh의 설정 페이지가 자동으로 폼을 렌더링하고, 사용자가 수정하면 `ctx.get`으로 읽을 수 있습니다. 구체적인 API는 공식 `dsh-settings` 패키지 문서를 참고하세요.

**Q6: `recentToolCalls` 함수에서 왜 `reverse()`를 하나요?**
events 배열의 끝에서부터 앞으로 순회하기 때문입니다(최근 N개를 가져옴), 결과는 역순입니다(최신이 앞에 옴). reverse로 시간 정순을 복원하면(가장 오래된 것이 앞에 옴) 실제 호출 순서와 일치해서, 결정 로직이 "최근 윈도우"를 이해하기 쉬워집니다.

---

**다음 장**: [제 5 장: 실전 사례](./05-cases.ko.md) —— Git 패널, HTML 초안 미리보기, 속도 개선 플러그인.
