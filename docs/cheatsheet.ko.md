# 한 페이지 치트시트(Cheatsheet)

> 이 카드를 인쇄/저장해두면 일상적인 dsh 사용에 책을 뒤질 필요가 없습니다.

## 설치와 시작

```bash
npx -y @deepseek-ai/dsh --version      # 설치 없이 실행
npm i -g @deepseek-ai/dsh              # 전역 설치
dsh web                                # Web UI → http://127.0.0.1:3080
dsh --profile headless "작업"          # 일회성 작업(스크립트/CI)
```

## 설정(~/.dsh/settings.yaml)

```yaml
agent-default-model:
  model: deepseek-v4-flash     # 또는 deepseek-v4-pro
  reasoningEffort: high        # off(사고 끄기/최고 속도) / high(기본값) / max(최고 성능)
```

## 추론 강도

| 강도 | 용도 | 비고 |
|---|---|---|
| `low` | 단순/배치/도구 체인의 저렴한 라운드(가장 빠름) | 실측 게이트웨이(pi-ai/opencode-go)만 지원 |
| `high` | 일상 기본값 | 공식 어댑터 기본값 |
| `max` | 복잡한 추론/긴 체인 계획 | 공식 어댑터 지원 |
| `off` | 사고 끄기/가장 빠름 | **DeepSeek 공식 어댑터 강도**(`low`를 대신함) |

> 도구 체인 작업 시간의 90%가 사고에 쓰입니다 —— 강도를 낮추는 것이 가장 빠른 속도 개선책입니다.

## 핵심 명령어

| 명령어 | 용도 |
|---|---|
| `dsh web` | Web UI |
| `dsh --profile headless "작업"` | 일회성 작업 |
| `dsh --dump-config` | 합성 설정 확인 |
| `dsh plugin --profile <n> add <pkg>` | 플러그인 설치 |

## 플러그인 장착(두 단계)

```yaml
# package.json에 의존성 추가
"my-plugin": "link:C:\\path\\to\\my-plugin"
# cordis.patch.yml에 장착 추가
- insert:
    - id: my-plugin
      name: my-plugin
```

```bash
cd ~/.dsh/profiles/web && pnpm install && dsh web
```

## 프롬프트 황금률

1. **검수 기준을 쓰기**: `"실행 검증 통과"` > `"스크립트 하나 작성해줘"`
2. **컨텍스트를 주기**: 파일이 어디 있는지, 데이터가 어떻게 생겼는지, 독자가 누구인지
3. **한 번에 하나의 작업**: 작은 폐루프가 거대한 작업보다 신뢰할 수 있음

## 캐시로 비용 절약

- 긴 작업은 세션 연속성 유지(자주 새로 만들지 않기)
- prompt 접두사 안정화(설정을 자주 바꾸지 않기)
- Web UI 하단의 "캐시 적중 %" 확인(실측 97%까지 달성)

## 트러블슈팅 빠른 참조

| 증상 | 해법 |
|---|---|
| 포트 점유 | `netstat -ano \| findstr 3080` → kill |
| 모델 무응답 | settings.yaml + API key 확인 |
| 플러그인 설치 실패 404 | 의존성은 `^0.1.0-rc.6` 라인 사용 |
| 긴 작업 크래시 | 전역 설치(npx 우회) + 추론 강도 낮추기 |

## 용어(요약)

`profile` 형태 · `bundle` 플러그인 그룹 · `host/client 반쪽` 서버/인터페이스 · `확장 포인트` 공식 훅 · `waterfall` 요청 체인 · `compaction` 컨텍스트 압축 · `headless` 일회성 CLI · `locations` 산출물 경로

---

전체 튜토리얼은 [dsh-handbook 백서](https://github.com/9bow/dsh-handbook-ko) · [설정 참조](./config-reference.ko.md) · [용어집과 명령어 빠른 참조](./appendix-glossary.ko.md) 참고
