# 부록 A: 용어집과 명령어 빠른 참조

## 용어집

| 용어 | 한 줄 설명 |
|---|---|
| **Harness** | 모델 바깥을 감싸는 엔지니어링 레이어: 세션, 도구, 컨텍스트, 루프 제어 |
| **dsh** | DeepSeek Harness의 명령줄 이름(`dsh` 명령어) |
| **Profile** | 실행 가능한 형태 하나(web / headless / 커스텀) = bundle 스택 + patch 레이어 |
| **Bundle** | 플러그인 집합 하나(공식: dsh-base, dsh-web-app, dsh-headless) |
| **Cordis** | dsh 하부의 플러그인 컨테이너(의존성 주입, 이벤트, 생명주기) |
| **플러그인(Plugin)** | cordis 플러그인; host 반쪽(Node)과 client 반쪽(브라우저)을 동시에 가질 수 있음 |
| **host 반쪽 / client 반쪽** | 플러그인의 Node 쪽(도구/서비스/이벤트)과 브라우저 쪽(UI) 두 얼굴 |
| **확장 포인트** | 공식이 제공하는 훅(agent/request, settings, conversationEvents, slots) |
| **agent/request waterfall** | 매 단계 모델 요청 전에 설정을 바꿀 수 있는 이벤트 체인(플러그인이 여기서 reasoningEffort 등을 주입) |
| **reasoning_effort** | 사고 강도(공식 어댑터 off/high/max; `low`는 실측 게이트웨이 강도) —— 속도와 품질의 트레이드오프 다이얼 |
| **headless** | 일회성 CLI 작업 모드(`dsh --profile headless "작업"`) |
| **compaction** | 긴 대화 컨텍스트 압축 |
| **subagent** | 서브에이전트(작업 병렬 위임) |
| **MCP** | Model Context Protocol(외부 도구 서버 연동) |
| **workflow** | 다단계 결정론적 워크플로우 오케스트레이션 |
| **locations** | 도구가 반환하는 파일 경로(산출물 추적을 구동) |
| **extension point** | 공식 훅(agent/request 등), 플러그인 연결 지점 |
| **waterfall** | 이벤트 체인: 리스너가 설정을 바꿔 다음 리스너에게 전달(agent/request가 이를 사용해 주입) |
| **context cache** | DeepSeek 프롬프트 캐시(반복 입력은 할인 가격으로 과금) |
| **캐시 적중률** | 입력 토큰이 캐시 가격으로 처리되는 비율(dsh 실측 97%까지 달성) |
| **TUI** | 터미널 UI(공식 미내장, 플러그인 필요) |
| **turn** | 대화의 한 라운드(사용자+어시스턴트+도구 호출) |
| **step** | turn 안의 한 추론 단계 |
| **guard** | 루프 위생/도구 타임아웃 플러그인 |
| **skill** | 스킬(skill-catalog가 컨텍스트에 주입, 모델이 필요에 따라 호출) |
| **inject** | cordis 의존성 주입(플러그인이 필요한 서비스를 선언) |
| **산출물 파일** | 모델이 생성/수정한 파일(대화 끝의 열람 가능한 칩) |
| **sandbox** | 명령 실행의 격리 샌드박스 |
| **rc** | 프리릴리스 버전(0.1.0-rc.6), 반복이 빠르고 파괴적 변경이 있을 수 있음 |

## 명령어 빠른 참조

### dsh 핵심

```bash
dsh web                                        # Web UI 시작
dsh --profile headless "작업"                  # 일회성 작업(스크립트/CI)
dsh --version                                  # 버전
dsh --dump-config                              # 합성 설정 트리
dsh plugin --profile <name> add <pkg>          # 플러그인 설치
```

### 환경

```bash
node --version                                 # ≥ 22 필요
npm install -g @deepseek-ai/dsh                 # 전역 설치
npx -y @deepseek-ai/dsh web                     # 설치 없이 실행
```

### 트러블슈팅

```bash
netstat -ano | findstr 3080                    # 포트 점유(Windows)
taskkill /PID <pid> /F                          # 프로세스 종료
cat ~/.dsh/settings.yaml                        # 전역 설정(모델/추론 강도)
```

### 플러그인 개발

```bash
# profile에 장착(web을 예로)
# ① package.json에 의존성 추가: "pkg": "link:C:\\path\\to\\pkg"
# ② cordis.patch.yml에 추가:
#    - insert:
#        - id: <플러그인id>
#          name: <npm패키지명>
cd ~/.dsh/profiles/web && pnpm install          # 설치
```

## 설정 참조(settings.yaml)

```yaml
agent-default-model:
  model: deepseek-v4-flash     # 또는 deepseek-v4-pro
  reasoningEffort: high        # off(사고 끄기/최고 속도) / high(기본값) / max(최고 성능)
```

---

**부록 B**: [공식 패키지 빠른 참조 총정리](./appendix-packages.ko.md) · **부록 C**: [동일 모델 다중 Agent 실측](./benchmark.ko.md)
