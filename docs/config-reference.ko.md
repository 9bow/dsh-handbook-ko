# 설정 참조 총정리(Configuration Reference)

> dsh를 깊게 설정해야 하는 사용자를 위한 문서: settings.yaml 전체 필드, profile 구조, cordis.patch.yml 문법. **rc 단계에서는 필드가 바뀔 수 있으니 공식 changelog를 기준으로 삼으세요**; 이 표는 0.1.0-rc.6 실측과 공식 문서를 기반으로 합니다.

## 1. settings.yaml(전역 설정)

`~/.dsh/settings.yaml`

| 필드 | 예시 | 설명 |
|---|---|---|
| `agent-default-model.model` | `deepseek-v4-flash` | 기본 모델(`deepseek-v4-pro`도 가능) |
| `agent-default-model.reasoningEffort` | `high` | 사고 강도: `off`(사고 끄기/최고 속도) / `high`(기본값) / `max`(최고 성능). 참고: 공식 DeepSeek 어댑터는 이 세 등급뿐이며, `low` 등은 pi-ai(opencode-go) 게이트웨이의 강도입니다 |
| (그 외 네임스페이스) | — | 각 플러그인의 설정 네임스페이스(예: 사이드바 설정) |

> 전체 필드는 공식 settings 서비스 스키마를 기준으로 삼으세요; `dsh --dump-config`로 현재 적용 중인 합성 설정을 볼 수 있습니다.

## 2. profile 구조

```
~/.dsh/profiles/<name>/
├── package.json          # 플러그인 의존성 + profile 매니페스트(dsh.profile)
├── cordis.patch.yml      # 패치 레이어(플러그인 장착/오버라이드)
├── cordis.yml            # (생성된) 합성 설정
├── pnpm-workspace.yaml
└── node_modules/
```

### package.json의 dsh.profile 매니페스트

```json
{
  "dsh": {
    "profile": {
      "bundles": [
        "@deepseek-ai/dsh-base",
        "@deepseek-ai/dsh-web-app"
      ]
    }
  }
}
```

`bundles`: 순서대로 로드되는 공식/서드파티 플러그인 그룹입니다(web은 dsh-web-app, headless는 dsh-headless를 사용).

## 3. cordis.patch.yml 문법

`cordis.patch.yml`은 **최상위 YAML 배열** 형태의 패치 항목이며, 자주 쓰는 세 가지는 다음과 같습니다:

### insert(플러그인 장착)

```yaml
- insert:
    - id: better-sidebar      # 플러그인 인스턴스 id(고유해야 함)
      name: dsh-better-sidebar # npm 패키지 이름
```

### override(플러그인 설정 오버라이드)

```yaml
- override:
    - id: speed-plugin
      config:
        baseline: low         # 플러그인 기본 설정을 오버라이드
```

### disable(플러그인 비활성화)

```yaml
- disable:
    - id: some-plugin
```

> 주의: 최상위는 반드시 배열이어야 하며(`- insert:`로 시작), 먼저 `[]`를 쓴 다음 항목을 쓰면 안 됩니다(YAML 문법 오류).

## 4. 자주 쓰는 설정 시나리오

### 기본 모델 바꾸기

```yaml
agent-default-model:
  model: deepseek-v4-pro
  reasoningEffort: high
```

### 로컬에서 개발 중인 플러그인 장착하기

```json
// package.json dependencies
"my-plugin": "link:C:\\path\\to\\my-plugin"
```

```yaml
# cordis.patch.yml
- insert:
    - id: my-plugin
      name: my-plugin
```

```bash
cd ~/.dsh/profiles/web && pnpm install && dsh web
```

### 현재 적용 중인 설정 확인하기

```bash
dsh --dump-config        # 사용자 레이어를 포함한 합성 트리
dsh --dump-default-config  # 사용자 레이어/패치 미포함
```

## 5. 자주 발생하는 설정 문제

| 문제 | 원인 | 해법 |
|---|---|---|
| 플러그인이 적용되지 않음 | cordis.patch.yml에 장착되지 않음 / 의존성 미설치 | insert 줄 확인 + `pnpm install` |
| settings를 바꿨는데 반응이 없음 | 재시작하지 않음 | `dsh web` 재시작 |
| 의존성 404 | rc.1 라인 단절 | `^0.1.0-rc.6` 사용 |
| YAML 파싱 실패 | 최상위에서 `[]`와 블록 항목을 섞어 씀 | 블록 배열로 통일 |
