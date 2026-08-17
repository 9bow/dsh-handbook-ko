# FAQ: 자주 묻는 질문 빠른 참조

> 각 챕터의 FAQ + 전역 고빈도 질문을 모아 한 페이지로 정리했습니다. 답을 못 찾으셨나요? [공식 Discussions](https://github.com/deepseek-ai/deepseek-harness/discussions)에 질문해보세요.

## 입문

**Q: dsh는 모델인가요?**
아닙니다. dsh는 런타임/프레임워크이며, 모델은 `llm` 플러그인을 통해 연결됩니다(공식은 DeepSeek V4 계열을 지원하며, OpenAI 호환 모델도 연결 가능).

**Q: dsh와 Claude Code는 무엇이 다른가요?**
Claude Code는 "완성차"(바로 사용 가능, 폐쇄적)이고, dsh는 "레고 베이스플레이트"(커스터마이즈 가능, 오픈소스)입니다. [제1장 비교](./01-intro.ko.md) 참고.

**Q: TypeScript를 써본 적 없어도 할 수 있나요?**
사용에는 전혀 필요 없습니다; 플러그인 작성에는 기초 TS가 필요하며, 백서에 완전한 코드가 제공됩니다.

**Q: 비용이 드나요?**
dsh 자체는 무료 오픈소스입니다; 대화하려면 DeepSeek API key가 필요합니다(사용량 기반 과금, Flash는 매우 저렴함 —— 오프피크 미적중 $0.22/M, 캐시 적중 $0.007/M, 적중 시 약 97% 할인; 2026-08-16부터 피크/오프피크 요금제 시행, 피크는 2배, [제14장](./14-cost.ko.md) 참고).

## 설치와 실행

**Q: `npx`가 너무 느린가요?**
처음 다운로드하는 패키지 용량이 큽니다(40+개 플러그인 모듈). `npm i -g @deepseek-ai/dsh` 설치 후에는 더 빠릅니다.

**Q: 브라우저에서 3080이 안 열리나요?**
포트가 점유되어 있습니다: `netstat -ano | findstr 3080` → PID kill.

**Q: `dsh --profile tui`가 오류를 내나요?**
tui profile은 플러그인으로 만들어야 합니다(공식 미내장), `dsh plugin --profile tui add <pkg>`.

**Q: 첫 `npx`가 8분 동안 아무 반응이 없나요?**
정상입니다 —— 처음 설치할 때 500+개의 의존성 패키지를 다운로드해야 합니다(Windows에서 특히 느림, [#176](https://github.com/deepseek-ai/deepseek-harness/discussions/176)). `npm i -g @deepseek-ai/dsh`로 전역 설치하면 이후 npx 다운로드가 필요 없습니다.

**Q: 시작 시 `--expose-internals is required for HMR service` 오류가 나나요?**
macOS arm64 / NixOS / 일부 Linux에서 cordis HMR loader가 Node 내부 모듈을 감지하지 못해 발생합니다([#113](https://github.com/deepseek-ai/deepseek-harness/discussions/113) [#269](https://github.com/deepseek-ai/deepseek-harness/discussions/269) [#690](https://github.com/deepseek-ai/deepseek-harness/discussions/690)). 임시 방안: `node --expose-internals <bin> web`으로 시작; 공식 수정을 기다리세요.

**Q: 포트 3080이 EACCES 오류를 내는데, 점유한 프로세스가 없나요?**
Windows에서 3080이 Hyper-V/WSL2/Docker Desktop의 예약 포트 구간에 들어갈 수 있습니다([#589](https://github.com/deepseek-ai/deepseek-harness/discussions/589)). `netstat -ano | findstr 3080`에 결과가 없으면 먼저 `netsh interface ipv4 show excludedportrange protocol=tcp`를 확인하거나, 바로 `dsh web --port 13080`으로 포트를 바꾸세요.

## 모델과 성능

**Q: 추론 강도는 어떻게 선택하나요?**
> ⚠️ 강도 지원은 어댑터마다 다릅니다: `deepseek-official` 어댑터의 능력 표는 `off`/`high`/`max`입니다(`low`는 공식 API가 실제로 지원하지만 어댑터가 노출하지 않음); opencode-go/pi-ai 게이트웨이는 `low`를 지원합니다. 다운그레이드 전에 현재 provider가 지원하는지 먼저 확인하세요(`does not support reasoning effort` 오류 = 어댑터 공백이니, 가장 가까운 사용 가능한 강도로 매핑하면 됩니다).

`low`(단순/배치/도구 라운드) / `high`(일상) / `max`(복잡). 도구 체인 시간의 90%가 사고에 쓰입니다 —— 강도를 낮추는 것이 가장 빠른 속도 개선책입니다.

**Q: 왜 제 작업이 느린가요?**
먼저 사고 강도가 높은지 + 콜드 스타트인지 확인하세요. 긴 작업은 `low` + 세션 연속성(캐시 적중)을 권장합니다.

**Q: 캐시 적중률은 어떻게 높이나요?**
세션 연속성 유지, prompt 접두사 안정화, 배치는 같은 세션에서. 실측 97%까지 달성합니다([제5장](./05-cases.ko.md) 참고); 커뮤니티의 긴 실행 실측은 99.7%까지 달성합니다([#560](https://github.com/deepseek-ai/deepseek-harness/discussions/560)).

**Q: 서드파티 모델/커스텀 게이트웨이에 "추론 강도" 옵션이 없나요?**
rc.6의 llm-pi-ai는 `thinkingFormat`/`supportsReasoningEffort`만 노출하며, 직접 작성한 provider의 reasoningEfforts는 settings.yaml에서 수동으로 선언해야 합니다([#122](https://github.com/deepseek-ai/deepseek-harness/discussions/122) [#302](https://github.com/deepseek-ai/deepseek-harness/discussions/302) [#736](https://github.com/deepseek-ai/deepseek-harness/discussions/736)). 선언 후 `400 unknown variant developer` 오류가 나면 게이트웨이가 developer role을 인식하지 못하는 것입니다 —— `compat.supportsDeveloperRole: false`를 설정해야 합니다([#280](https://github.com/deepseek-ai/deepseek-harness/discussions/280) [#614](https://github.com/deepseek-ai/deepseek-harness/discussions/614) [#636](https://github.com/deepseek-ai/deepseek-harness/discussions/636)). 커뮤니티에는 게이트웨이 방언을 자동으로 감지하는 플러그인이 있습니다([#559](https://github.com/deepseek-ai/deepseek-harness/discussions/559)).

**Q: 모든 도구 호출에서 `Error: unknown tool ""` 오류가 나나요?**
rc.6 스트리밍 파싱 버그입니다: SSE 청크가 덮어쓰기 할당을 해서 도구 이름/ID를 빈 문자열로 지워버립니다([#725](https://github.com/deepseek-ai/deepseek-harness/discussions/725) 근본 원인 + 수정; [#161](https://github.com/deepseek-ai/deepseek-harness/discussions/161) 같은 계열). **두 가지 유형의 근본 원인**:
1. **덮어쓰기 할당**: `translate.ts`가 `block.name`/`block.callId`를 청크마다 누적이 아닌 덮어쓰기로 처리해서, 이후 청크에 빈 `function.name`이 오면 이미 파싱된 도구 이름을 지워버립니다(게시물 내 주요 수정: `(block.name ?? '') + ...`로 누적하도록 변경);
2. **null이 암묵적으로 문자열로 변환됨**: 일부 모델(예: hy3, longcat-2.0)이 스트리밍 delta에서 `id`/`name`에 `null`을 채웁니다 —— `null !== undefined`가 항상 참이라서, 기존 판단 로직이 `null`을 암묵적으로 문자열로 변환해 연결시켜, 파괴적인 도구 이름을 만들어냅니다(예: `"Glob" + null → "Globnull"`). 더 근본적인 수정은 **엄격한 타입 검증**입니다: `typeof call.id === 'string'` / `typeof call.function?.name === 'string'`일 때만 누적하도록(#725 댓글의 보충 방안).

공식이 현재 Issue/PR 제출을 닫아둔 상태라서(#725 댓글에서 확인), **직접 수정해야 합니다** `packages/llm/llm-deepseek/src/translate.ts`(해당 함수를 청크별 누적 + 엄격한 타입 검증으로 변경)를 수정한 후 `pnpm run build`로 재빌드하고 dsh를 재시작하세요. 공식 수정 전에는 다운그레이드하거나 버전을 기다릴 수도 있습니다; 모델이 반복적으로 재시도할 수 있으니 제때 중단하는 데 유의하세요.

**Q: 초장문 세션이 열리지 않고 `Maximum call stack size exceeded` 오류가 나나요?**
초장문 응답(20만+ 토큰)의 `sourceEventSeqs` 배열이 함수 인자로 펼쳐지면서 V8의 인자 개수 상한을 초과합니다([#317](https://github.com/deepseek-ai/deepseek-harness/discussions/317) [#370](https://github.com/deepseek-ai/deepseek-harness/discussions/370) [#508](https://github.com/deepseek-ai/deepseek-harness/discussions/508)). 세션 파일 자체는 손상되지 않았습니다; rc 단계의 알려진 결함이니 수정을 기다리거나 커뮤니티 패치를 찾아보세요.

## 플러그인 개발

**Q: 플러그인이 설치되지 않나요(404)?**
rc.1 의존성 단절입니다 —— `^0.1.0-rc.6` 라인을 사용하는지 확인하세요(제3장 트러블 #1).

**Q: 첫 플러그인을 작성할 때 가장 흔한 트러블은 무엇인가요?(커뮤니티 6가지 트러블)**
출처: 공식 디스커션 [#380](https://github.com/deepseek-ai/deepseek-harness/discussions/380) 「첫 dsh 플러그인을 작성하며 겪은 여섯 가지 트러블」(작성자 codeAnqiang-ma 수록 허가 받음, dsh `0.1.0-rc.6` 로컬 재확인, @codeAnqiang-ma 님께 감사드립니다). 충실히 요약합니다:
1. **`@deepseek-ai/*`를 import할 수 있는지는 플러그인이 어디에 설치되었는지에 달려 있습니다**: 개발용 `link`로 profile에 연결하면, Node가 심볼릭 링크의 실제 경로를 따라 해석하므로 폴백 디렉터리 `~/.dsh/profiles/node_modules`에 도달하지 못합니다(registry로 설치한 경우에만 이 폴백을 만납니다) → `ERR_MODULE_NOT_FOUND` 오류. 해결: 플러그인이 `@deepseek-ai/*`를 import하지 않고 전부 `ctx`에서 가져오게 하세요; schemastery로 config schema를 작성해야 한다면 peer dependency로(registry 형태에서는 통함).
2. **`inject`는 문자열 배열만 가능합니다**: `{ required: [...], optional: [...] }`로 작성하면 `required`/`optional`을 두 개의 서비스 이름으로 취급해서, 시작이 `pending (waiting for services: required, optional)`에서 멈춥니다. 공식 패키지는 모두 배열 형태입니다(`["skills"]`, `["systemPrompt"]`, `["agents","tools","skills"]`).
3. **하나의 profile 레이어가 되려면 반드시 `dsh.bundle`을 선언해야 합니다**: `package.json`에 `"dsh": { "bundle": { "patch": "./cordis.patch.yml" } }`를 추가하지 않으면, `dsh plugin add`로 설치해도 그냥 일반 의존성일 뿐 플러그인은 전혀 작동하지 않습니다(경고가 pnpm 출력에 섞여 있어 놓치기 쉬움). 패키지 이름을 바꿀 때는 patch의 `name`도 동기화해야 하며, 그렇지 않으면 `plugin(s) failed to load` 오류가 발생합니다.
4. **prompt section은 압축 후에도 남아 있습니다(좋은 소식)**: `ctx.systemPrompt.section()`으로 등록한 내용은 컨텍스트 압축 후에도 남아 있습니다 —— 매 단계마다 레지스트리에서 다시 조립되므로, 대화 이벤트를 감시해 반복 주입하거나 중복 방지 가드를 작성할 필요가 없습니다. `order`에는 관례적인 구간이 있습니다(`-100` harness identity / `0` persona / `100-199` 도구 가이드), 같은 order는 플러그인 로드 순서대로 정렬됩니다(불안정하니 겹치지 않는 숫자를 직접 고르세요).
5. **Web 쪽은 preset을 바꿀 필요 없지만, persona는 예외입니다**: 스킬 레지스트리와 prompt section 모두 「전역 레이어 + scope 체인」으로 병합해서 읽으므로, 전역 레이어에 등록된 모든 agent가 접근할 수 있습니다; 하지만 preset이 자체 장착한 `persona`(`deployment:persona`)는 scope 레이어에서 같은 이름으로 전역을 가려버려서, profile patch의 persona를 바꿔도 Web 기본 세션에는 적용되지 않으며 preset을 바꿔야 합니다. 플러그인 section 이름에 자신만의 접두사를 붙이면 가려지지 않습니다.
6. **npm 게시의 두 가지 트러블**: registry가 국내 미러(읽기 전용)를 가리킬 때는 `npm login`/`npm publish` 모두 명시적으로 `--registry=https://registry.npmjs.org`를 지정해야 합니다; `npm publish`는 `--otp`만 인식하고(`--auth-type` 옵션 없음), 2FA를 passkey/Touch ID로만 연결했다면 OTP를 꺼낼 수 없으니 TOTP 인증 코드나 복구 코드를 써야 합니다.

> 작성자 저장소: [dsh-superpowers](https://github.com/codeAnqiang-ma/dsh-superpowers)(Superpowers 방법론 플러그인). rc 반복이 빠르니 항목이 유효하지 않으면 알려주세요.

**Q: `agent/request`의 `next()`는 await해야 하나요?**
**반드시** 그래야 합니다. await하지 않으면 provider/model이 유실되어 오류가 발생합니다(제4장 트러블).

**Q: 플러그인을 가장 빨리 작성하려면?**
[플러그인 템플릿](../examples/plugin-template/)을 클론하고, 순수 함수 로직을 수정한 다음 장착하면 됩니다. waterfall 동작을 검증하고 싶은데 API key가 없나요? 커뮤니티에 무비용 방안이 있습니다: 저장소에 내장된 mock llm + headless + 감사 dump([#462](https://github.com/deepseek-ai/deepseek-harness/discussions/462)).

**Q: Code 모드에서 run_code/bash가 계속 `missing required property "description"` 오류를 내나요?**
rc.6 도구 파라미터 트러블입니다: run_code와 bash의 description 필드가 같은 이름이면서 required로 표시되어, 모델이 종종 내부 bash.description을 이미 전달한 것으로 착각해 외부 필드가 빠짐 → 무한 재시도 루프([#558](https://github.com/deepseek-ai/deepseek-harness/discussions/558) [#581](https://github.com/deepseek-ai/deepseek-harness/discussions/581) [#689](https://github.com/deepseek-ai/deepseek-harness/discussions/689)). 발생하면 외부 description을 수동으로 보충하거나 표준 모드로 전환하세요.

**Q: 분기(fork)된 세션에서 edit가 계속 "edit requires reading ... first"라고 하나요?**
rc.6 버그입니다: fork로 새 세션을 만들면 부모 세션의 "이미 읽은 파일" 관찰 상태를 상속받지 못해, 이력에서 분명히 읽었는데도 정책은 읽지 않았다고 판단합니다([#275](https://github.com/deepseek-ai/deepseek-harness/discussions/275) [#450](https://github.com/deepseek-ai/deepseek-harness/discussions/450)). 파일을 한 번 다시 읽으면 우회할 수 있습니다.

**Q: 어떤 플러그인을 설치한 후 dsh 시작 시 `Invalid schema for function ...` 오류가 나나요?**
플러그인 schema가 잘못 작성되었거나 agent가 `cordis.patch.yml`을 잘못 수정해서, dsh 전체가 시작되지 않습니다([#297](https://github.com/deepseek-ai/deepseek-harness/discussions/297) [#447](https://github.com/deepseek-ai/deepseek-harness/discussions/447) [#708](https://github.com/deepseek-ai/deepseek-harness/discussions/708)). 복구: `~/.dsh/profiles/web/cordis.patch.yml`의 해당 줄을 삭제/수정하거나 백업에서 복원하세요. agent가 스스로 플러그인을 설치해 설정을 바꾸게 하기 전에는 미리 백업하세요.

**Q: 도구 호출이 `Cannot read properties of undefined (reading 'prepare')` 오류를 내나요?**
`@deepseek-ai/dsh-tools`가 프로세스 안에 두 개(전역 + profile) 존재해서, 모듈 레벨 Symbol이 분열되어 스케줄러가 찾지 못합니다([#572](https://github.com/deepseek-ai/deepseek-harness/discussions/572) [#783](https://github.com/deepseek-ai/deepseek-harness/discussions/783)). profile의 중복된 dsh-tools 의존성을 정리하면 됩니다.

## 보안과 프로덕션

**Q: dsh는 안전한가요?**
도구 실행에는 샌드박스 + 승인 레이어가 있습니다. 의료/법률 등 고위험 출력은 사람의 검토가 필요합니다. 하지만 방심하지 마세요: 커뮤니티 감사에서 샌드박스에 실제 탈출 경로가 있음을 발견했습니다 —— `node:vm`은 보안 경계가 아닙니다(workflow/동적 플러그인이 탈출 가능, [#243](https://github.com/deepseek-ai/deepseek-harness/discussions/243) [#451](https://github.com/deepseek-ai/deepseek-harness/discussions/451) [#774](https://github.com/deepseek-ai/deepseek-harness/discussions/774)), `workspace-write`에서 작업공간 전체를 재귀 삭제할 수 있습니다([#149](https://github.com/deepseek-ai/deepseek-harness/discussions/149)), 모델이 approval 순환 참조를 통해 full access를 스스로 승인할 수 있습니다([#250](https://github.com/deepseek-ai/deepseek-harness/discussions/250)), localhost Web이 iframe 클릭재킹에 노출될 수 있습니다([#381](https://github.com/deepseek-ai/deepseek-harness/discussions/381)). 전체 서드파티 감사는 [#454](https://github.com/deepseek-ai/deepseek-harness/discussions/454) 참고.

**Q: 프로덕션에 쓸 수 있나요?**
rc 단계라 파괴적 변경이 있습니다; 핵심 의존성은 `0.1.0` 정식 버전을 기다리고, 생태계 탐색은 지금 시작할 수 있습니다. 프로덕션 스케줄링 측면에서 주의할 점: headless는 승인 요청을 처리할 UI가 없어 동작이 정의되지 않습니다([#291](https://github.com/deepseek-ai/deepseek-harness/discussions/291)); 서브에이전트는 제한 없이 파생되어 서비스를 죽일 수 있습니다([#131](https://github.com/deepseek-ai/deepseek-harness/discussions/131)); 큰 작업에서 heap OOM이 보고된 적 있습니다([#754](https://github.com/deepseek-ai/deepseek-harness/discussions/754)).

**Q: 왜 `--host 0.0.0.0`이 금지되나요? 우회할 수 있나요?**
공식이 안전을 위해 능동적으로 거부한 것입니다 —— 네트워크에 노출하는 것은 RCE/자격 증명/컨텍스트 유출을 신뢰할 수 없는 네트워크에 넘기는 것과 같습니다([#76](https://github.com/deepseek-ai/deepseek-harness/discussions/76) [#130](https://github.com/deepseek-ai/deepseek-harness/discussions/130)). 우회의 대가는 큽니다(설정 변경/리버스 프록시), 원격 인증이 완비되기 전에는 **권장하지 않습니다**. 정말 원격이 필요하다면: SSH 터널 + Host/Origin 변경으로 가능하지만 절차가 번거롭습니다([#242](https://github.com/deepseek-ai/deepseek-harness/discussions/242)).

**Q: 긴 작업이 크래시할 수 있나요?**
전역 설치(npx 우회) + 추론 강도 낮추기 + 메모리 관찰을 권장합니다(실측 50단계 작업에서 메모리가 뚜렷하게 증가). Windows에서 Temp 디렉터리가 정리되면 샌드박스가 영구적으로 크래시되어 자체 복구되지 않을 수 있습니다([#758](https://github.com/deepseek-ai/deepseek-harness/discussions/758)); 강제 kill 후 재시작하면, 작성 중이던 세션에 닫히지 않은 turn이 남을 수 있습니다([#466](https://github.com/deepseek-ai/deepseek-harness/discussions/466)).

## Windows 호환성

**Q: 워크스페이스 선택 시 한자 경로가 잘리나요(`开`/`一`/`需` 등 글자 뒤가 전부 사라짐)?**
rc.6의 유명한 버그입니다: Windows 네이티브 디렉터리 선택기의 readUtf16이 UTF-16 하위 바이트만 확인해서, 하위 바이트가 0x00인 한자(开 U+5F00, 一 U+4E00, 需 U+9700 등)를 만나면 조기에 잘라버립니다([#107](https://github.com/deepseek-ai/deepseek-harness/discussions/107) [#151](https://github.com/deepseek-ai/deepseek-harness/discussions/151) [#563](https://github.com/deepseek-ai/deepseek-harness/discussions/563)). 17+개 스레드에서 재현됨([#244](https://github.com/deepseek-ai/deepseek-harness/discussions/244) [#580](https://github.com/deepseek-ai/deepseek-harness/discussions/580) 등), 커뮤니티가 cherry-pick 가능한 수정을 제공합니다. 임시 방안: 워크스페이스 경로에 U+XX00 문자를 피하세요.

**Q: 디렉터리 선택창이 `win32 folder dialog worker exited before reporting a result` 오류를 내나요?**
koffi/네이티브 picker 계열 크래시입니다([#30](https://github.com/deepseek-ai/deepseek-harness/discussions/30) [#154](https://github.com/deepseek-ai/deepseek-harness/discussions/154) [#236](https://github.com/deepseek-ai/deepseek-harness/discussions/236)). koffi 3.1.3/3.1.4 사전 컴파일 손상은 3.1.2로 고정하면 됩니다([#293](https://github.com/deepseek-ai/deepseek-harness/discussions/293)); STA CoUninitialize 세그폴트라는 또 다른 근본 원인도 있습니다([#768](https://github.com/deepseek-ai/deepseek-harness/discussions/768)).

**Q: 브라우저에서 127.0.0.1:3080을 열었는데 API가 전부 403 오류를 내나요?**
Host/Origin 신뢰 검증 문제입니다: `http://localhost:3080`으로 접속하면 보통 정상입니다([#153](https://github.com/deepseek-ai/deepseek-harness/discussions/153) [#313](https://github.com/deepseek-ai/deepseek-harness/discussions/313) [#764](https://github.com/deepseek-ai/deepseek-harness/discussions/764)). 서버가 출력한 주소가 사용 불가능할 수 있으니 localhost로 바꿔서 시도해보세요.

**Q: LAN/휴대폰 접속 시 `crypto.randomUUID is not a function` 오류가 나나요?**
평문 HTTP는 loopback이 아니면 보안 컨텍스트가 아니라서, `crypto.randomUUID`를 쓸 수 없어 모든 RPC가 실패합니다([#221](https://github.com/deepseek-ai/deepseek-harness/discussions/221) [#367](https://github.com/deepseek-ai/deepseek-harness/discussions/367) [#514](https://github.com/deepseek-ai/deepseek-harness/discussions/514) [#755](https://github.com/deepseek-ai/deepseek-harness/discussions/755)). rc.6에는 저장소에서 이미 수정된 shim이 포함되어 있지 않습니다; 원격 접속에는 평문 IP를 쓰지 마세요.

**Q: Windows에서 pwsh/도구 호출이 `missing required property "command"` 오류를 내거나 멈추나요?**
pwsh 호출 실패 계열입니다([#121](https://github.com/deepseek-ai/deepseek-harness/discussions/121) [#225](https://github.com/deepseek-ai/deepseek-harness/discussions/225)), 가장 심할 때는 dsh 전체가 멈춥니다([#663](https://github.com/deepseek-ai/deepseek-harness/discussions/663)). 샌드박스와 PowerShell 조합에 관련된 알려진 문제이니, Windows 사용자는 먼저 bash로 다운그레이드하거나(가능하다면) 수정을 기다리세요.

## 생태계

**Q: 공식이 외부 PR을 받나요?**
현재 명확히 "아직 받지 않습니다"(CONTRIBUTING). Discussion 제안 + 커뮤니티 채널을 이용하세요(제7장 참고). 커뮤니티가 여러 차례 정식 버전에서 Issues/PR을 열어달라고 호소했습니다([#341](https://github.com/deepseek-ai/deepseek-harness/discussions/341) [#775](https://github.com/deepseek-ai/deepseek-harness/discussions/775)).

**Q: 제 플러그인은 어떻게 홍보하나요?**
`dsh-plugin` topic 추가 + npm 게시 + 공식 Discussion Show-and-tell + awesome 목록([#215](https://github.com/deepseek-ai/deepseek-harness/discussions/215) 중영 이중언어 선별 목록). 커뮤니티에는 원클릭 설치 프로그램인 DSH Plugin Marketplace([#442](https://github.com/deepseek-ai/deepseek-harness/discussions/442))와 통합 저장소([#688](https://github.com/deepseek-ai/deepseek-harness/discussions/688))도 있습니다.

**Q: CLI / TUI를 원하나요?**
공식은 아직 없지만, 커뮤니티가 이미 만들었습니다: TUI 플러그인([#391](https://github.com/deepseek-ai/deepseek-harness/discussions/391)), CLI([#405](https://github.com/deepseek-ai/deepseek-harness/discussions/405)), Pi 계열 CLI([#132](https://github.com/deepseek-ai/deepseek-harness/discussions/132)). headless 이어서 실행하기(session-id 출력 + `--resume`)는 커뮤니티의 고빈도 요청입니다([#167](https://github.com/deepseek-ai/deepseek-harness/discussions/167)).

**Q: 데스크톱 버전 / Node 설치 없이 쓰고 싶나요?**
커뮤니티에 다양한 패키징이 있습니다: mac DMG + Windows exe 설치 패키지([#414](https://github.com/deepseek-ai/deepseek-harness/discussions/414)), Windows 원클릭 패키지([#419](https://github.com/deepseek-ai/deepseek-harness/discussions/419)), 데스크톱 쉘([#767](https://github.com/deepseek-ai/deepseek-harness/discussions/767) 종합 비교). 모두 비공식 커뮤니티 유지입니다.

**Q: memory / 비전 능력을 원하나요?**
공식은 아직 내장하지 않았지만, 커뮤니티 방안이 체계를 갖췄습니다: memory([#192](https://github.com/deepseek-ai/deepseek-harness/discussions/192) [#484](https://github.com/deepseek-ai/deepseek-harness/discussions/484) [#525](https://github.com/deepseek-ai/deepseek-harness/discussions/525)), 비전 브리지([#456](https://github.com/deepseek-ai/deepseek-harness/discussions/456) [#495](https://github.com/deepseek-ai/deepseek-harness/discussions/495) [#733](https://github.com/deepseek-ai/deepseek-harness/discussions/733)). 다른 도구에서 이전하는 브리지 플러그인도 있습니다([#272](https://github.com/deepseek-ai/deepseek-harness/discussions/272) [#531](https://github.com/deepseek-ai/deepseek-harness/discussions/531)).

**Q: 기업용 메신저(企业微信) 채널이 추가되지 않나요?**
공식 기업용 메신저 채널이 텐센트의 리스크 관리/포화 상태에 걸려 있어([#25](https://github.com/deepseek-ai/deepseek-harness/discussions/25) [#270](https://github.com/deepseek-ai/deepseek-harness/discussions/270) [#591](https://github.com/deepseek-ai/deepseek-harness/discussions/591)), 공식 수정을 기다리는 중입니다; 커뮤니티가 자발적으로 위챗 그룹/QQ 그룹을 만들었습니다([#705](https://github.com/deepseek-ai/deepseek-harness/discussions/705) [#730](https://github.com/deepseek-ai/deepseek-harness/discussions/730)).

---

**더 보기**: 용어집은 [부록 A](./appendix-glossary.ko.md), 명령어 빠른 참조는 [한 페이지 카드](./cheatsheet.ko.md) 참고
