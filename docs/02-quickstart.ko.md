# 제 2 장: 5분 빠른 시작

> 본 장의 목표: **따라 하면서 실행해보기**. 모든 명령어마다 예상 출력과 흔한 오류 해결법을 제공합니다. 터미널을 열어놓고 함께 따라 해보시길 권장합니다.

## TL;DR(본 장 핵심, 30초 버전)

1. **설치**: `npx -y @deepseek-ai/dsh web` → http://127.0.0.1:3080
2. **자주 쓰는 두 가지 모드**: web(대화 UI) / headless(`dsh --profile headless "작업"`, CI 친화적) —— 공식은 총 4가지 실행 모드를 지원(2.4.1절 개요 참고)
3. **추론 강도**: off / low(사고 끄기 또는 약한 사고/최고 속도, 공식 provider는 `off`, 게이트웨이는 `low` 사용) / high(기본값) / max(최고 성능) —— **도구 체인 작업 시간의 90%가 사고에 쓰이므로, 강도를 낮추는 것이 가장 빠른 속도 개선책**입니다
4. **모델**: `deepseek-v4-flash`(기본값, 가성비) 또는 `deepseek-v4-pro`(플래그십)
5. **설정**: `~/.dsh/settings.yaml`(모델 + 추론 강도)

<details><summary>본 장 목차</summary>
- [2.1 준비 작업(30초 점검)](#21-준비-작업30초-점검)
  - [Node 버전 레드라인(≥22.19)](#node-버전-레드라인2219)
- [2.2 설치(세 가지 방법)](#22-설치세-가지-방법)
  - [설치 트러블슈팅(커뮤니티 실제 경험)](#설치-트러블슈팅커뮤니티-실제-경험)
- [2.3 모드 1: Web UI(`dsh web`)](#23-모드-1-web-uidsh-web)
- [2.4 모드 2: Headless(일회성 작업, 스크립트/CI에 적합)](#24-모드-2-headless일회성-작업-스크립트ci에-적합)
  - [2.4.1 공식 4가지 실행 모드 개요](#241-공식-4가지-실행-모드-개요)
- [2.5 첫 번째 플러그인: web에 Git 패널 추가하기](#25-첫-번째-플러그인-web에-git-패널-추가하기)
- [2.6 설정과 디렉터리 빠른 참조](#26-설정과-디렉터리-빠른-참조)
- [2.7 명령어 빠른 참조](#27-명령어-빠른-참조)
- [2.8 트러블슈팅 빠른 참조](#28-트러블슈팅-빠른-참조)
</details>

## 2.1 준비 작업(30초 점검)

| 필요 사항 | 확인 명령어 | 통과 기준 |
|---|---|---|
| **Node.js ≥ 22.19** | `node --version` | `v22.19.0` 또는 그 이상(**레드라인**, 아래 참고) |
| npm(Node에 포함) | `npm --version` | 버전 번호만 있으면 OK |
| 네트워크 | npm registry 접근 가능 | 패키지 설치 가능 |
| (선택) DeepSeek API Key | https://platform.deepseek.com | 실제 대화용 |

> API Key가 없어도 dsh는 시작할 수 있습니다(화면은 열림), 하지만 대화하려면 Key가 필요합니다. 본 백서의 예시는 이미 설정되어 있다고 가정합니다.

### Node 버전 레드라인(≥22.19)

커뮤니티 실측 결과 **Node < 22.19에서는 치명적인 두 가지 API 누락**이 발생합니다:

| 누락된 API | 오류 예시 | 관련 스레드 |
|---|---|---|
| `node:zlib`에 `createZstdDecompress` 없음 | `TypeError: zlib.createZstdDecompress is not a function` | [#100](https://github.com/deepseek-ai/deepseek-harness/discussions/100) |
| `AbortSignal.timeout` 미구현 | `AbortSignal.timeout is not a function` | [#311](https://github.com/deepseek-ai/deepseek-harness/discussions/311) |

**해결법**: nvm/volta/fnm으로 `22.19.0+`로 전환하세요. Node 24 초기 버전(예: v24.15)도 `install-lefthook failed`를 유발할 수 있습니다 —— 이 오류를 만나면 `22.19.0`으로 고정하는 것이 가장 안정적입니다([#748](https://github.com/deepseek-ai/deepseek-harness/discussions/748)).

## 2.2 설치(세 가지 방법)

**방법 1: 직접 실행(초보자 추천)**

```bash
npx -y @deepseek-ai/dsh --version
```

처음 실행하면 dsh를 다운로드합니다(패키지 용량이 커서 40+개 플러그인 모듈 포함, 약 1-3분 소요). 버전 번호가 보이면 성공입니다:

```text
0.1.0-rc.6
```

**방법 2: 전역 설치(자주 사용할 경우 추천)**

```bash
npm install -g @deepseek-ai/dsh
dsh --version
```

**방법 3: Node 설치가 필요 없는 설치 패키지(초보자 선택 가능)**

Node를 설치하고 싶지 않으신가요? 커뮤니티 개발자 codeAnqiang-ma([#380](https://github.com/deepseek-ai/deepseek-harness/discussions/380) 플러그인 트러블슈팅 스레드 작성자, 수록 허가 받음)가 **Node 설치가 필요 없는 설치 패키지**를 제공합니다 —— mac DMG / Windows exe, 하부는 공식 dsh를 그대로 패키징하고 Node 런타임을 내장해 사전 의존성이 전혀 없습니다:

> [dsh-installers](https://github.com/codeAnqiang-ma/dsh-installers)(비공식, 공식 rc 버전에 맞춰 해당 설치 패키지 배포, SHA256 검증 포함)

"Node 환경을 건드리고 싶지 않고 더블클릭으로 바로 쓰고 싶은" 체험 사용자에게 적합합니다; 플러그인 개발/잦은 업그레이드가 필요하다면 방법 1이나 2를 권장합니다.

### 설치 트러블슈팅(커뮤니티 실제 경험)

| 문제 | 증상 | 해결 | 스레드 |
|---|---|---|---|
| **최초 npx 극도로 느림(Windows)** | `npx dsh web`이 Windows에서 처음 실행될 때 8분+ 동안 아무 반응 없음, npm이 500+ 패키지를 다운로드해야 함 | 인내심을 갖고 기다리거나, `npm i -g @deepseek-ai/dsh`로 전환하면 이후 즉시 시작됨 | [#176](https://github.com/deepseek-ai/deepseek-harness/discussions/176) |
| **pnpm dlx 404** | `pnpm dlx @deepseek-ai/dsh web`이 `@deepseek-ai/dsh-pty@0.0.1-rc.2`가 게시되지 않았다는 오류 | `pnpm dlx` 대신 `npx`나 `npm i -g` 사용 | [#369](https://github.com/deepseek-ai/deepseek-harness/discussions/369) |
| **pnpm 전역 설치 후 플러그인을 찾지 못함** | `pnpm add -g @deepseek-ai/dsh` 후 시작 시 `Cannot find package 'cordis-plugin-timer'` 오류 | pnpm 전역 설치의 의존성 해석 전략이 npm과 다름; `npm i -g`나 npx 권장 | [#55](https://github.com/deepseek-ai/deepseek-harness/discussions/55) |
| **WSL2에서 npx 설치 실패** | WSL2(Ubuntu 등 배포판)에서 `npx -y @deepseek-ai/dsh` 설치 실패 | 먼저 npm 프록시 설정 확인; 그래도 안 되면 **소스 설치로 전환**(커뮤니티 실측 성공) | [#118](https://github.com/deepseek-ai/deepseek-harness/discussions/118) |

> **WSL2 설치 참고사항**: 커뮤니티는 [#118](https://github.com/deepseek-ai/deepseek-harness/discussions/118)에서 WSL2에서 `npx` 설치가 실패할 수 있다고 보고했습니다(Windows 네이티브 쪽은 정상, 해당 스레드의 "윈도우에서는 이미 다 설치하고 놀고 있음" 대조 참고). 점검 순서: ① npm 프록시/registry 설정 확인; ② `npx`가 여전히 안 되면 **먼저 소스 설치를 시도**(공식 저장소를 clone한 후 `pnpm install`, 커뮤니티 실측 가능). 자세한 내용은 해당 스레드 댓글 참고.
>
> 전역 설치 방식 비교: **`npm i -g`**가 가장 안정적(커뮤니티 검증이 가장 많음); **`npx`**는 체험용으로 적합; **`pnpm dlx`**는 아직 권장하지 않음(rc 단계에서 pty 패키지가 pnpm에서 보이는 registry에 게시되지 않음).

## 2.3 모드 1: Web UI(`dsh web`)

### 시작

```bash
dsh web
```

예상 출력:

```text
dsh web: http://127.0.0.1:3080
```

브라우저에서 http://127.0.0.1:3080 을 엽니다.

### 화면 구성 이해하기(스크린샷 대조)

![dsh Web UI 대화](./assets/demo-web-chat.png)

| 영역 | 내용 |
|---|---|
| 왼쪽 열 | 세션 목록 / 워크스페이스 전환 / 새 세션 |
| 가운데 열 | 대화 영역: 입력창, 모델 선택(`DeepSeek V4 Flash`), 추론 등급(`High`) |
| 오른쪽/하단 | 플러그인 사이드바(기본적으로 비어 있음; 커뮤니티 플러그인 설치 후 표시됨) |
| 우측 상단 | Session log(세션 로그) / 트레이스(도구 호출 트레이스) |

### 첫 번째 대화

1. "새 세션" 클릭
2. 입력창에 입력: `안녕, 한 문장으로 자기소개 해줘`
3. Enter로 전송

예상 답변은 다음과 비슷합니다:

> 안녕하세요! 저는 DeepSeek 기반 AI 코딩 어시스턴트입니다. 코드 작성, 디버깅, 파일 처리, 자료 검색은 물론 다양한 개발 및 업무 작업을 도와드릴 수 있습니다.

### 모델과 추론 강도

입력창 옆의 "모델 선택"을 클릭하면 모델과 추론 강도 선택기가 열립니다:

![dsh 모델 선택과 추론 강도](./assets/demo-model-selector.png)

| 모델 | 포지셔닝 |
|---|---|
| `deepseek-v4-flash`(기본값) | 가성비: 빠르고 저렴, 일상 사용에 충분 |
| `deepseek-v4-pro` | 플래그십: 더 강력하지만 더 비싸고 느림 |

**추론 등급**(사고 모드 3단계, 2026-08-13부터 지원):

| 강도 | 속도 | 품질 | 권장 시나리오 |
|---|---|---|---|
| `low` | 가장 빠름 | 충분함 | 단순/결정론적 작업, 배치, 도구 체인의 저렴한 라운드 |
| `high`(기본값) | 중간 | 좋음 | 일상적인 Agent 작업 |
| `max` | 가장 느림 | 최고 | 복잡한 추론, 긴 체인 계획 |

> 위 표의 강도 등급은 본 백서가 실측한 환경(pi-ai / opencode-go 게이트웨이)이 지원하는 것입니다. DeepSeek 공식 어댑터(기본 provider=deepseek-official, llm-deepseek 플러그인)는 `off`(사고 끄기, 최고 속도) / `high` / `max` 세 등급만 받으며, `low`를 넣으면 `UNSUPPORTED_REASONING_EFFORT`를 던집니다. settings.yaml에서 기본 provider를 쓸 때는 `off` / `high` / `max`를 사용하세요.

> 💡 **성능의 핵심 인지**: 모델은 **매 도구 호출 전마다 다시 사고**합니다. 실측 결과 "파일 생성" 작업 하나에서 사고가 벽시계 시간의 ~90%를 차지했고, 50단계 도구 체인 작업에서는 사고 누적 시간이 수 분에서 십수 분에 이를 수 있습니다. **추론 강도를 낮추는 것이 가성비가 가장 높은 속도 개선 수단**입니다(제6장 + 속도 개선 플러그인 예제 참고).

## 2.4 모드 2: Headless(일회성 작업, 스크립트/CI에 적합)

```bash
dsh --profile headless "안녕, 한 문장으로 자기소개 해줘"
```

예상 출력(결과를 출력한 후 프로세스 종료):

```text
안녕하세요! 저는 DeepSeek 기반 AI 코딩 어시스턴트입니다. 코드 작성, 디버깅, 파일 처리, 자료 검색은 물론 다양한 개발 및 업무 작업을 도와드릴 수 있습니다.
```

**Headless의 핵심 가치**:
- **자동화**: CI, 서버, cron에 넣을 수 있음
- **스크립트 친화적**: 0이 아닌 종료 코드 = 실패; 출력을 파이프로 처리 가능
- **세션 격리**: 매 호출마다 새로운 세션(`--resume`으로 복원 가능, `dsh --profile headless --help` 참고)

**실전**: 매일 한 번 "일일 리포트 생성"을 실행하는 스크립트 작성:

```bash
dsh --profile headless "워크스페이스의 오늘 git log를 읽고 한국어 일일 리포트 요약을 생성해줘" > daily-report.md
echo "exit=$?"
```

### 2.4.1 공식 4가지 실행 모드 개요

> 출처: 공식 사이트 https://deepseek.com/harness/ 및 공식 저장소 `docs/`(2026-08 스냅샷). 앞의 두 절(web / headless)은 본 백서가 직접 실측하며 상세히 다룬 내용입니다; 나머지 두 모드는 공식이 발표했지만 본 백서에서는 아직 항목별로 실측하지 않았습니다.

| 모드 | 한 줄 설명 | 용도 | 백서 커버리지 |
|---|---|---|---|
| **Standard mode**(표준) | 완전한 도구 세트 + 브라우저 인터페이스 | 일상 대화/개발(`dsh web`) | ✅ 2.3절 실측 |
| **Minimal mode**(미니멀) | `bash`와 `str_replace_editor` 두 도구만 사용 | 벤치마크 테스트, 최소 공격 표면 | ⚠️ 미실측, 공식 docs 참고 |
| **Code mode**(코드) | 모델이 TypeScript를 작성해, 단일 프로그램 안에서 여러 라운드의 도구 호출을 오케스트레이션 | 긴 체인 자동화, 결정론적 실행 | ⚠️ 미실측, 공식 docs 참고 |
| **Creator mode**(생성) | 런타임 자체 점검 + 메모리 내 Cordis 플러그인 테스트 + 새 모드 조합 | 플러그인 개발, 빠른 프로토타이핑 | ⚠️ 미실측, 공식 docs 참고 |

> 초보자에게: 일상에는 **Standard(web)**, 스크립트에는 **Headless**면 충분합니다; Code/Creator 모드는 고급 기능이니 공식 문서가 안정화되면 더 깊이 파보세요. 공식 문서 사이트(VitePress): https://deepseek-harness.github.io/deepseek-harness/

## 2.5 첫 번째 플러그인: web에 Git 패널 추가하기

dsh의 사이드바는 기본적으로 비어 있습니다 —— 커뮤니티 플러그인 `dsh-better-sidebar`를 설치해 "모든 것이 플러그인"을 체험해봅시다(자세한 원리는 제3장 참고, 여기서는 먼저 실행부터 해봅니다):

```bash
# 1. 당신의 web profile을 찾으세요
#    Windows: %USERPROFILE%\.dsh\profiles\web
#    macOS/Linux: ~/.dsh/profiles/web

# 2. package.json의 dependencies에 한 줄 추가(link:가 플러그인 소스 코드를 가리킴)
#    "dsh-better-sidebar": "link:C:\\path\\to\\DSH-better-sidebar"

# 3. cordis.patch.yml에 장착 줄 추가
#    - insert:
#        - id: better-sidebar
#          name: dsh-better-sidebar

# 4. 설치 후 재시작
cd ~/.dsh/profiles/web && pnpm install
#    (Windows cmd에서는 `~`가 전개되지 않으니 다음을 사용하세요: cd %USERPROFILE%\.dsh\profiles\web)
dsh web
```

재시작 후 오른쪽에 파일 관리 / 터미널 / **Git 패널** / 브라우저 등의 탭이 나타납니다:

![dsh Git 패널(better-sidebar 플러그인)](./assets/demo-git-panel.png)

> 그림 속 "원격 가져오기 / 가져와서 병합 / 푸시" 버튼은 커뮤니티 PR로 구현된 것입니다(제5장 사례 참고) —— **이것이 바로 "플러그인 생태계"가 작동하는 방식입니다**.

## 2.6 설정과 디렉터리 빠른 참조

첫 실행 후 생성되는 디렉터리:

```text
~/.dsh/
├── settings.yaml          # 전역 설정(모델, 추론 강도)
├── profiles/              # profile 디렉터리
│   └── web/
│       ├── package.json      # 플러그인 의존성 + 매니페스트
│       └── cordis.patch.yml  # 패치 레이어(플러그인 장착)
├── sessions/              # 세션 데이터
└── storages/              # 영속 저장소
```

`settings.yaml` 예시:

```yaml
agent-default-model:
  model: deepseek-v4-flash
  reasoningEffort: high
```

## 2.7 명령어 빠른 참조

| 명령어 | 용도 |
|---|---|
| `dsh web` | Web UI 시작(=`dsh --profile web`) |
| `dsh --profile headless "작업"` | 일회성 작업, 결과 출력 후 종료 |
| `dsh plugin --profile <name> add <pkg>` | profile에 플러그인 설치 |
| `dsh --dump-config` | 합성된 설정 트리 출력 |
| `dsh --profile tui` | TUI 모드(tui 플러그인을 먼저 설치해야 함, 공식 미내장) |
| `dsh --version` | 버전 |

## 2.8 트러블슈팅 빠른 참조

| 증상 | 원인과 해법 |
|---|---|
| `dsh: profile "tui" does not exist` | tui profile은 플러그인으로 만들어야 함(`dsh plugin --profile tui add <pkg>`) |
| `npx`가 극도로 느림 | 첫 다운로드 용량이 큼; `npm i -g` 사용 시 더 빠름 |
| 브라우저에서 3080 포트가 안 열림 | 포트 점유 상태: `netstat -ano \| findstr 3080` → PID kill |
| 모델이 응답하지 않음 | `~/.dsh/settings.yaml`의 모델 설정 + API Key 확인 |
| 플러그인 설치 실패(404) | **rc.1 의존성 단절**: 의존성이 `^0.1.0-rc.6` 라인을 사용하는지 확인(제3장 흔한 트러블슈팅 #1) |
| 업그레이드 후 동작이 바뀜 | rc 단계의 파괴적 변경은 정상이니 공식 changelog 확인 |

---

**다음 장**: [제 3 장: profile과 플러그인 시스템](./03-profiles.ko.md) —— 커스터마이즈 가능한 골격을 이해합니다.

---

> **소스 코드 방식으로 시작이 느린 근본 원인(#1424 커뮤니티 실측 확인)**: `pnpm dsh web`은 매번 시작할 때 tsx/esbuild로 **전체 TS 소스 그래프를 그 자리에서 트랜스파일**합니다(전체 빌드가 아님), 여기에 기계식 디스크나 파일 수가 많으면 콜드 스타트가 수 분에 이를 수 있습니다. **실측 시간**: tsx 핫 캐시 ~40초 / 콜드 캐시 ~5분 / 컴파일 산출물 `lib/bin.js` 버전 ~12초(페이지 응답 5ms). 해결: ① 시작 명령을 `node apps\cli\lib\bin.js web`로 바꿔 컴파일 산출물을 사용 ② 또는 먼저 `pnpm build`로 전체 빌드를 한 번 실행 ③ 또는 그냥 `npx @deepseek-ai/dsh web`(배포 버전) 사용.
> **Windows 공통 트러블슈팅 1 —— 미니멀 모드가 실행되지 않음**(#1889 실측): `terminal inspection is unsupported on platform win32` 오류가 나면 **node-pty와는 무관**합니다 —— 호출 순서상 플랫폼 inspector를 먼저 파싱한 후 spawn하는데, inspector가 앞에 있기 때문입니다. node-pty 재설치/Node 교체/VS Build Tools 설치 모두 소용없습니며; dsh-win32 계열 패치가 필요합니다.
>
> **Windows 공통 트러블슈팅 2 —— 당일 배포된 플러그인 버전이 설치되지 않음**: profile의 `pnpm-workspace.yaml`에서 `minimumReleaseAgeExclude`는 연결 시점의 버전만 기록하므로, 플러그인이 새 버전을 배포한 후 약 하루 동안은 `dsh plugin add <pkg>@latest`가 `Already up to date`를 반환합니다. 즉시 설치하려면: `--config.minimumReleaseAge=0`.

## 실습 문제(10분 이내 완료)

1. **설치**: `npx -y @deepseek-ai/dsh --version`으로 버전 확인
2. **Web 대화**: `dsh web` 시작, 새 세션에서 "안녕" 전송, 답변과 화면 레이아웃 관찰
3. **Headless**: `dsh --profile headless "1+1은 몇이야"`, 결과 출력 후 종료되는지 확인
4. **추론 강도 실험**: settings.yaml의 `reasoningEffort`를 `off`(공식 provider)나 `low`(서드파티 게이트웨이)로 바꾸고 단순 작업을 다시 실행해 속도 차이를 체감
5. **트러블슈팅 연습**: "포트 점유" 상황을 재현하고(먼저 3080을 점유하는 서비스를 하나 실행), `netstat`으로 원인을 찾아보기

> 전부 통과했다면 [제 3 장](./03-profiles.ko.md)으로 가서 "왜 이렇게 바꿀 수 있는지" 이해해봅시다.
