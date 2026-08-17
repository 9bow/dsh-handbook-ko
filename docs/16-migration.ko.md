# 제 16 장: Claude Code에서 이전하기

> 본 장의 목표: **쌓아온 것을 가지고 이사하는 것이지, 처음부터 다시 짓는 것이 아닙니다**. 각 유형의 자산마다 「DSH에서는 어떻게 되는지」에 대한 실측 결론을 제공하며, 모두 `0.1.0-rc.6` 소스 코드 기준으로 항목별로 검증했습니다.

## TL;DR(본 장 핵심, 30초 버전)

1. **절반은 공짜입니다**: 프로젝트 `CLAUDE.md`는 옮길 필요 없음(DSH가 네이티브로 읽음), `SKILL.md` 형식은 그대로 호환됨
2. **절반은 기계적입니다**: `.mcp.json`은 무손실 변환 가능(도구 이름 `mcp__server__tool`이 양쪽에서 완전히 동일), hooks는 공식 브릿지가 있음
3. **자동화**: `npx dsh-movein`으로 이사 목록 미리보기, `--apply`로 실제 적용; 대화 이력은 `dsh-chat-import` 사용
4. **이사 전에 먼저 계산해보기**: 스킬 카탈로그는 스킬 하나당 요청마다 약 28토큰 소비, 여러분이 쓰는 것만 옮기고 가진 것 전부를 옮기지 마세요

## 16.1 자산 대조표(실측)

| Claude Code 자산 | DSH 호환성 | 실제 상황 |
| --- | --- | --- |
| 프로젝트 `CLAUDE.md` | 네이티브, 수정 없음 | `instructionFileCandidates`가 기본으로 `CLAUDE.md`를 포함, 프로젝트 루트에서 cwd까지 자동으로 찾음 |
| 전역 `~/.claude/CLAUDE.md` | 심볼릭 링크 하나면 됨 | 전역 위치는 `$DSH_HOME/AGENTS.md`만 인식하므로, 링크만 걸면 됨 |
| 스킬(`SKILL.md`) | 형식 그대로 호환 | 프론트매터 메타데이터는 개방형 객체로 파싱, 알 수 없는 키(`allowed-tools` 등)는 무시됨. 주의: `.claude/skills`는 DSH의 기본 스킬 루트가 **아니므로**, `~/.dsh/skills`나 `<프로젝트>/.dsh/skills`에 두어야 함 |
| MCP(`.mcp.json`) | 무손실 기계 변환 | 서버마다 `dsh-mcp-client` 설정 한 줄(stdio와 streamable-http), 도구 이름이 양쪽에서 동일하므로 이를 참조하는 스킬은 수정할 필요 없음 |
| hooks | 공식 브릿지, 일부 이벤트만 | `dsh-hooks-claude-code`가 기존 설정을 그대로 실행, 30개 이벤트 중 7개만 매핑됨, command 타입만 지원. **트러블**: matcher가 DSH 도구 이름의 대소문자를 구분함, `Bash`는 소문자 `bash` 도구에 매칭되지 않아 보안 hook이 조용히 실패할 수 있음(공식 디스커션 #582), 수정 전에는 matcher를 소문자로 작성할 것 |
| 권한 규칙 | 네이티브 아님, 브릿지 가능 | DSH는 3단계의 대략적인 프리셋만 있음. `deny`/`ask`는 `tools/pre-execute`에서 강제 실행 가능(커뮤니티 플러그인), `allow`는 대응물이 없음 |
| 서브에이전트(`.claude/agents`) | 직접 가져올 수 없음 | DSH 프리셋은 markdown이 아니라 `agent.cordis.yml` 디렉터리이며, 현실적인 경로는 스킬로 변환하는 것(프론트매터 메타데이터가 거의 동일함) |
| 슬래시 명령(`.claude/commands`) | 파일 형태의 대응물 없음 | DSH 명령은 코드로 등록되며, 사용자가 호출 가능한 스킬이 파일 레벨의 대체재임 |
| 세션 | 가장 어려움, 직접 작성하지 말 것 | 세션 파일은 v0 형식 + zstd frame + 엄격한 이벤트 검증을 사용하며, 공식은 호환성을 명시적으로 보장하지 않음. 이전 세션은 [dsh-chat-import](https://github.com/Nwflower/dsh-chat-import)를 사용(13개 소스 지원, 역방향 내보내기 가능) |

## 16.2 자동화된 이사

```sh
npx dsh-movein            # 미리보기, 이사 목록 출력, 어떤 파일도 쓰지 않음
npx dsh-movein --apply    # 실제 적용
npx dsh-movein --reverse  # DSH에서 새로 만든 스킬을 Claude Code로 되돌리기(양쪽 사용)
```

플러그인으로 설치해 agent에게 대신 시킬 수도 있습니다: `dsh plugin --profile web add dsh-movein`, 그다음 대화에서 "내 Claude Code 설정을 옮겨줘"라고 말하면 됩니다.

권한 규칙은 **이전 차이 리포트**를 출력합니다(몇 개가 그대로 적용되는지, 몇 개가 매핑되지 않는지 항목별로 나열), 조용히 변환하지 않습니다. 이사할 때마다 `~/.dsh/movein-manifest.json`에 출처와 도착지를 기록합니다.

## 16.3 커뮤니티가 실제로 겪은 트러블(시작 관련)

1. **해석되지 않는 패키지를 patch에 써넣으면 dsh 시작이 바로 실패합니다**(plugin tree failed to load, 경고가 아니라 fatal입니다). 먼저 패키지를 설치하고, 설치가 성공한 후에 설정 줄을 작성하세요.
2. **주변 패키지의 npm latest 태그가 코어보다 뒤처질 수 있습니다**(hooks 브릿지가 rc.5였는데 코어는 이미 rc.6이었던 적 있음), 설치는 호스트 dsh 버전을 기준으로 고정하세요.
3. **`dsh-hook-protocol`은 hooks 브릿지의 peer 의존성입니다**, 호스트 설치에는 포함되지 않으므로 함께 설치해야 합니다.

## 16.4 이사 전에 먼저 계산해보기

스킬 카탈로그는 system-reminder로 매 요청마다 주입됩니다: 고정 래핑이 143토큰, 스킬 하나당 약 28토큰(96자 설명 기준). 스킬 129개의 설정은 요청마다 약 3.8k 토큰을 짊어지며, 캐시는 비용을 절약해주지만 컨텍스트 윈도우는 절약해주지 못합니다. **여러분이 쓰는 스킬을 옮기세요, 가진 스킬 전부가 아니라** —— 설명은 짧게 작성하세요. 재현 스크립트와 방법은 [dsh-movein의 토큰 청구서 실측](https://github.com/sjh9714/dsh-movein/blob/main/docs/token-bill.md) 참고.
