# 제 7 장: 생태계와 리소스

> 본 장의 목표: "dsh 생태계에 합류하는" 지도를 드립니다 —— 공식 채널, 커뮤니티 현황, 플러그인 실습, 그리고 참여 방법.

## TL;DR(본 장 핵심, 30초 버전)

1. **공식 채널**: GitHub 저장소(소스 + issue), 공식 문서 사이트(VitePress), API 문서, Discord, Discussions —— 현재 기여 채널은 Discussions가 중심입니다
2. **외부 PR은 아직 받지 않음**(2026-08-13 시점), 하지만 공식은 dsh-plugin 생태계 프로젝트 만들기, 튜토리얼/블로그 작성을 장려합니다 —— 커뮤니티가 이미 자발적으로 "보고-재현-근본원인-수정 브랜치" 기여 템플릿을 만들었습니다(7.2절 참고)
3. **플러그인 생태계 스냅샷**: `DSH-better-sidebar`(가장 완성도 높은 커뮤니티 플러그인), 예제 속도 개선 플러그인(제4장에서 완전 분석), 본 백서 —— 플러그인을 찾으려면 `topic:dsh-plugin` 검색
4. **초보자 참여 경로**: 사용해보기 → 작은 개선(커뮤니티 플러그인에 PR 제출) → 플러그인 공개 → 콘텐츠 작성
5. **생태계 초일 = 선점 우위**: 모든 초기 생태계에는 "가장 먼저 게를 먹어본 사람"의 보너스가 있으며, 지금이 딱 진입할 때입니다

<details><summary>본 장 목차</summary>
- [7.1 공식 채널](#71-공식-채널)
- [7.2 커뮤니티 기여 방식](#72-커뮤니티-기여-방식)
- [7.3 플러그인 생태계(2026-08-13 스냅샷)](#73-플러그인-생태계2026-08-13-스냅샷)
- [7.4 생태계에 참여하는 방법(초보자 경로)](#74-생태계에-참여하는-방법초보자-경로)
- [7.5 추천 읽기 순서](#75-추천-읽기-순서)
- [7.6 실제 플러그인 설치 실습](#76-실제-플러그인-설치-실습)
- [7.7 플러그인을 찾는 방법](#77-플러그인을-찾는-방법)
- [7.8 생태계 참여의 전체 흐름](#78-생태계-참여의-전체-흐름)
- [7.9 생태계 마일스톤 타임라인](#79-생태계-마일스톤-타임라인)
- [7.10 공식 디스커션과의 연계](#710-공식-디스커션과의-연계)
</details>

## 7.1 공식 채널

| 리소스 | 주소 | 용도 |
|---|---|---|
| 공식 저장소 | https://github.com/deepseek-ai/deepseek-harness | 소스 코드, 아키텍처 문서, issue |
| 공식 문서 사이트 | https://deepseek-harness.github.io/deepseek-harness/ | 공식 VitePress 문서 사이트(guide/providers/development/reference, 2026-08 오픈) |
| 제품 사이트 | https://deepseek.com/harness/ | 공식 제품 소개(4가지 실행 모드, 트레이스 뷰 등) |
| API 문서 | https://api-docs.deepseek.com | 모델, 가격, API 가이드 |
| Discord | 공식 README 내 링크 | 커뮤니티 토론 |
| Discussions | 공식 저장소 Discussions | 제안/도움 요청(공식이 현재 권장하는 기여 채널) |

**참고**: 공식 CONTRIBUTING은 "현재 외부 PR을 받지 않습니다"라고 명시합니다(2026-08-13 시점) —— 하지만 다음은 장려합니다:
- Discussions에 제안 작성(공식이 검토함)
- **dsh-plugin 생태계 프로젝트 만들기**(공식이 명시적으로 인정하는 방식)
- 튜토리얼/블로그 작성

> 📚 **공식 자체 문서의 보충**: 위의 채널 외에도, 공식 저장소는 체계적인 내부 문서를 유지하고 있습니다 —— `packages/README.md`(패키지 목록 공식 표), `docs/subsystems/`(서브시스템 설계), `docs/cookbook/`(9개 개발 가이드), `docs/tool-catalog.md` / `docs/config-catalog.md` / `docs/module-graph.md`(자동 생성 목록). 백서는 "초보자 관점의 보충"이고, 공식 문서는 "아키텍처의 권위"이니 둘을 함께 읽으세요.

## 7.2 커뮤니티 기여 방식

공식은 아직 외부 PR을 받지 않지만, 커뮤니티는 이미 자발적으로 성숙한 **"보고-재현-근본원인-수정 브랜치"** 기여 템플릿을 만들었습니다 —— [#341](https://github.com/deepseek-ai/deepseek-harness/discussions/341), [#371](https://github.com/deepseek-ai/deepseek-harness/discussions/371), [#775](https://github.com/deepseek-ai/deepseek-harness/discussions/775)에서 반복적으로 공식에 Issues/PR을 열어달라고 호소했지만 공식은 아직 응답하지 않았고, 그래서 기여자들이 수정 사항을 게시물에 직접 담기 시작했습니다.

**커뮤니티 기여 템플릿 4단계**:

1. **보고**: Discussion으로 게시, 제목에 【Bug】 표시, OS / Node / dsh 버전 첨부
2. **재현**: 최소 재현 단계 + 오류 로그 앞 20줄
3. **근본원인**: 구체적인 파일과 함수까지 특정(예: readUtf16이 하위 바이트 0x00만 판단)
4. **수정 브랜치**: fork 후 수정 사항 제출, 게시물에 `git cherry-pick <commit>` 명령 첨부

**cherry-pick 가능한 수정을 포함한 게시물 예시**:

| 스레드 | 문제 | 수정 형태 |
|---|---|---|
| [#244](https://github.com/deepseek-ai/deepseek-harness/discussions/244) | Windows 디렉터리 선택기가 "需求" 등 한자가 포함된 경로를 자름 | 수정 패치 첨부 |
| [#295](https://github.com/deepseek-ai/deepseek-harness/discussions/295) | 워크스페이스 생성 시 중국어 디렉터리 경로가 잘림 | 재현 + 수정 코드 조각 |
| [#580](https://github.com/deepseek-ai/deepseek-harness/discussions/580) | Win32 네이티브 디렉터리 선택기가 U+XX00에서 UTF-16 경로를 자름 | cherry-pick 가능한 수정 첨부 |

> 이런 게시물들의 공통점: **"공식이 고쳐주세요"가 아니라 "제가 고쳤으니 쓰세요"**입니다. 공식 Issues가 닫힌 단계에서, 이것이 dsh 커뮤니티의 가장 효율적인 협업 방식입니다 —— 공식의 부담을 나누는 동시에 자신의 커뮤니티 신뢰도도 쌓습니다.

## 7.3 플러그인 생태계(2026-08-13 스냅샷)

| 프로젝트 | 포지셔닝 | 상태 |
|---|---|---|
| `DSH-better-sidebar` | 파일 관리/터미널/Git/브라우저 사이드바 | 가장 완성도 높은 커뮤니티 플러그인 |
| 예제 속도 개선 플러그인 | 도구 호출 속도 개선(reasoning_effort 자동 조절) | 교육용 예제(제4장) |
| `dsh-handbook`(본 백서) | 초보자 튜토리얼 | 생태계 문서 |
| **DeepSeek Desktop** | Windows 데스크톱(x64 커뮤니티 설치 패키지, v0.2.0 오프라인 설치 프로그램) | [#872](https://github.com/deepseek-ai/deepseek-harness/discussions/872), 관련 [#529](https://github.com/deepseek-ai/deepseek-harness/discussions/529) [#446](https://github.com/deepseek-ai/deepseek-harness/discussions/446) |
| **turtle-ui** | 터미널 TUI 인터페이스 플러그인([turtle1999/turtle-ui](https://github.com/turtle1999/turtle-ui)) | 커뮤니티 TUI 솔루션, 설치 시도는 [#871](https://github.com/deepseek-ai/deepseek-harness/discussions/871) 참고 |
| **memory 플러그인 계열** | 세션 간 장기 기억(설계 제안 / 장기 기억 / MEMORY.md·USER.md 이식) | [#192](https://github.com/deepseek-ai/deepseek-harness/discussions/192) [#484](https://github.com/deepseek-ai/deepseek-harness/discussions/484) [#525](https://github.com/deepseek-ai/deepseek-harness/discussions/525) |
| **《기억체》제안** | 세션 간, 격리된, 사용자가 명시적으로 장착하는 기억 단위(하부 설계, memory 플러그인 계열과 연결 가능) | [#1822](https://github.com/deepseek-ai/deepseek-harness/discussions/1822) szx-a |
| **dsh-sgme** | 기억 엔진: 대화 이력과 장기 기억을 계층화, 추출 전 자동 정리(세션 내용 65-96% 절감), 시나리오별 기억 블록 주입 | [freehul/sgme](https://github.com/freehul/sgme)(npm 패키지 `dsh-sgme`), [#1052](https://github.com/deepseek-ai/deepseek-harness/discussions/1052) |
| **pi-quiet-tools** | 도구 출력 압축: 컨텍스트에 들어가기 전 큰 결과를 앞뒤 미리보기 + 로컬 artifact로 압축(12,000자 또는 240줄 초과 시 트리거) | [pi2dsh](https://github.com/weijiafu14/pi2dsh)를 통해 장착, [#1052](https://github.com/deepseek-ai/deepseek-harness/discussions/1052) |
| **dsh-win32** | Windows에서 공식이 빠뜨린 win32 ProcessInspector를 보완해 미니멀 모드의 지속적인 shell이 실행되도록 함; 샌드박스 안에서 쓸 수 있는 busybox 변형, ConPTY 포그라운드 명령 인식, GBK/UTF-16 읽기도 포함 | [sjh9714/dsh-win32](https://github.com/sjh9714/dsh-win32)(npm `dsh-win32`), 채택된 제안 [#1889](https://github.com/deepseek-ai/deepseek-harness/discussions/1889) |
| **dsh-installers** | Node 설치가 필요 없는 설치 패키지: mac DMG / Windows exe, Node 런타임 내장, 사전 의존성 전무 | [codeAnqiang-ma/dsh-installers](https://github.com/codeAnqiang-ma/dsh-installers)([#380](https://github.com/deepseek-ai/deepseek-harness/discussions/380) 작성자 codeAnqiang-ma 제공), 설치는 제2장 2.2절 방법 3 참고 |
| **kubemd** | Kubernetes 런타임 장애 진단 스킬(사례 기억 + CLI 이중 엔트리, 5개 시나리오 kind 실측 검증) | [guiyi-labs/kubemd](https://github.com/guiyi-labs/kubemd), `git clone`으로 바로 사용(DSH skill) |

**플러그인 찾기**: GitHub에서 `topic:dsh-plugin` 검색.
**플러그인 배포하기**: 여러분의 저장소에 `dsh-plugin` topic을 추가하고 npm에 게시.

## 7.4 생태계에 참여하는 방법(초보자 경로)

1. **사용해보기**: `dsh web` + 커뮤니티 플러그인 두 개 설치, 일상 사용에 익숙해지기
2. **작은 개선**: 커뮤니티 플러그인에 PR 제출(제5장의 세 사례를 읽어보세요, 완전한 PR 패턴입니다)
3. **플러그인 공개**: 제4장의 최소 host 플러그인부터 시작해 `dsh-plugin` topic 달기
4. **콘텐츠 작성**: 튜토리얼/리뷰/트러블슈팅 글(공식이 장려함), 본 백서와 상호 인용

## 7.5 추천 읽기 순서

| 목표 | 경로 |
|---|---|
| 빠르게 시작하기 | 제2장 → better-sidebar 설치 → 일상 사용 |
| 플러그인 개발 | 제3장 → 제4장 → 제5장 사례 따라하기 |
| 성능 튜닝 | 제6장 → 제4장 예제 소스 코드 |
| 심화 커스터마이징 | 공식 AGENTS.md(아키텍처) → docs/architecture.md → packages/ 소스 코드 |

## 7.6 실제 플러그인 설치 실습

아래는 대표적인 두 플러그인의 **완전한 장착 단계**입니다(문법은 제3장 3.2절과 동일).

### `DSH-better-sidebar` 설치

**① `package.json`에 의존성 추가**(`~/.dsh/profiles/web/package.json`):

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

**③ 설치와 검증**:

```bash
cd ~/.dsh/profiles/web && pnpm install && dsh web
```

재시작 후 왼쪽에 파일 관리/터미널/Git/브라우저 네 개의 탭이 나타나는지 관찰하세요.

### 예제 속도 개선 플러그인 설치

**① `package.json`에 의존성 추가**:

```json
{
  "dependencies": {
    "dsh-speed-plugin": "link:C:\\path\\to\\dsh-speed-plugin"
  }
}
```

**② `cordis.patch.yml`에 장착 줄 추가**:

```yaml
- insert:
    - id: speed-plugin
      name: dsh-speed-plugin
```

**③ 설치와 검증**:

```bash
cd ~/.dsh/profiles/web && pnpm install && dsh web
```

**④ 효과 검증**: "파일 3개 생성" 작업을 하나 보내고, 로그에 `[speed-plugin] calls=[...] => reasoningEffort=low`가 나타나는지 관찰하세요(제4장 4.5절 참고).

> ⚠️ 두 플러그인 모두 `@deepseek-ai/dsh-agent ^0.1.0-rc.6`을 요구합니다. `pnpm install`이 404를 반환하면 버전 라인을 확인하세요(제3장 3.5절 참고).

## 7.7 플러그인을 찾는 방법

| 채널 | 구체적인 방법 | 예상 결과 |
|---|---|---|
| **GitHub topic** | https://github.com/topics/dsh-plugin 또는 `topic:dsh-plugin` 검색 | topic이 달린 모든 저장소 |
| **GitHub 전역 검색** | `dsh-plugin` + 언어 필터 `TypeScript` | 키워드가 포함된 저장소와 코드 |
| **npm 검색** | `npm search dsh-plugin` 또는 https://www.npmjs.com/search?q=dsh-plugin | 이미 게시된 패키지(rc 단계에서는 GitHub이 중심) |
| **공식 디스커션 Show and tell** | https://github.com/deepseek-ai/deepseek-harness/discussions/categories/show-and-tell | 커뮤니티 자체 추천 프로젝트 |
| **본 백서의 인용 링크** | 제4/5/10장 사례 속 저장소 링크 | 실기 검증된 플러그인 |

**팁**: `topic:dsh-plugin`이 가장 정확한 필터링 방식입니다. 2026-08-13 시점 결과는 한 자릿수에 불과합니다 —— 지금이 바로 진입 기회입니다.

## 7.8 생태계 참여의 전체 흐름

7.4절의 "네 단계"를 **실행 가능한 의사결정 트리**로 펼쳐봅니다:

```text
사용해보기 → Issue로 피드백 → 플러그인 공개(npm publish) → 콘텐츠 작성
```

**npm publish 절차 요약**:

1. `package.json`에 `"name": "dsh-xxx"` + `"main": "src/index.ts"` + 올바른 `peerDependencies`가 있는지 확인
2. `npm login` → `npm publish --access public`
3. GitHub 저장소에 `dsh-plugin` topic 추가
4. 공식 Discussions의 Show and tell 카테고리에 게시

> CONTRIBUTING.md 참고: 사례에는 "시나리오 / 명령어 / 소요 시간 또는 산출물 / 검증 방식"이 포함되어야 합니다 —— 플러그인을 공개할 때 이 정보를 첨부하면 전환율이 더 높아집니다.

## 7.9 생태계 마일스톤 타임라인

| 시기 | 사건 | 의미 |
|---|---|---|
| **2026-08-13** | dsh 오픈소스 공개(초일) | MIT 라이선스로 발표, Agent 프로그래머블 시대의 출발점 |
| **2026-08-13** | 공식 패키지 60+ 동시 공개 | `packages/` 아래 도구/컨텍스트/세션/서브에이전트/MCP/워크플로우/보안/모델/인터페이스 전체 커버 |
| **2026-08-13 당일** | 커뮤니티 플러그인 폭발 | 공식 디스커션에서 하루에 30+ 게시물, 플러그인 트러블슈팅, TUI 예제, Windows 경로 버그 등 포함 |
| **2026-08-13 그 주** | 본 백서 공개 | 14개 챕터 중국어 튜토리얼 + Benchmark + 플러그인 템플릿, 초일 문서 공백을 채움 |
| **진행 중** | 공식 디스커션 지속 활발 | 매주 2-3개 게시물 응답, 플러그인/경로/TUI/vision 등 방향을 이미 커버(7.10절 참고) |
| **계획 중** | 플러그인 템플릿 확장 / 비디오 튜토리얼 / CI 검증 | ROADMAP.md P1 우선순위 |

**핵심 인지**: 2026-08-13은 "발표일"이 아니라 "생태계 초일"입니다 —— 공식 패키지, 커뮤니티 플러그인, 중국어 튜토리얼이 같은 날 준비되었습니다. 참여자 입장에서는, **지금 진입하는 것 = 공식 인프라와 함께 성장하는 것**입니다.

## 7.10 공식 디스커션과의 연계

본 백서는 이미 공식 Discussions에서 8개 게시물에 응답했습니다. 아래는 **효과적으로 참여한 사례**입니다:

| 게시물 | 주제 | 응답 방식 | 재사용 가능한 패턴 |
|---|---|---|---|
| [#380](https://github.com/deepseek-ai/deepseek-harness/discussions/380) | 플러그인 트러블슈팅 | 결론을 제3장 3.5절 "흔한 트러블슈팅"에 정리 | 문제 → 재현 → 튜토리얼 작성 → 댓글로 인용 |
| [#392](https://github.com/deepseek-ai/deepseek-harness/discussions/392) | TUI examples | docs/config-reference.md에 보충 | 공식에 예제 부족 → 실행 확인 → 레퍼런스 작성 → 링크 공유 |
| [#401](https://github.com/deepseek-ai/deepseek-harness/discussions/401) | Windows 경로 버그 | 제12장 크로스플랫폼 약점에 기록 | 플랫폼 버그 → 경계 기록 → 공식 부담 분산 |
| [#384](https://github.com/deepseek-ai/deepseek-harness/discussions/384) | visionDS | 능력 매트릭스의 vision 지원 상태 업데이트 | 새 능력 → 조사 → 표 업데이트 → 확인 |
| [#118](https://github.com/deepseek-ai/deepseek-harness/discussions/118) | 일반 토론 | FAQ에 반영(docs/faq.md 6개 카테고리 문답) | 고빈도 질문 → 카테고리별 정리 → 링크 제공 |
| [#655](https://github.com/deepseek-ai/deepseek-harness/discussions/655) | 커뮤니티 5개 프로젝트 | 생태계 전체 조망을 제7장 7.3절에 정리 | 흩어진 프로젝트 통합 → 생태계 지도 형성 |
| [#735](https://github.com/deepseek-ai/deepseek-harness/discussions/735) | token 비용 | 제6장 6.6절 비용 모델에 반영 | 비용 측정 → 공식화해서 정리 |
| [#781](https://github.com/deepseek-ai/deepseek-harness/discussions/781) | LSP 제안 | 제11장 미래 전망에 기록 | 선행 기능 → 아키텍처 진화 추적 |

**효과적인 참여의 3가지 원칙**: ① 먼저 직접 실행해본 후 댓글 달기(환경 정보와 오류 로그 포함); ② 결론을 인용 가능한 콘텐츠로 작성(댓글 하나는 묻히지만, 챕터/FAQ로 쓰면 지속적으로 가치를 만듦); ③ 링크로 반복을 대체(댓글에는 링크를 남기세요, 예: "제7장 7.6절 참고").

## 맺음말

dsh는 2026-08-13에야 오픈소스로 공개된 프로젝트입니다 —— **생태계의 모든 날이 "초기"입니다**. 백서는 dsh의 발전에 맞춰 지속적으로 업데이트될 것입니다. 어떤 장의 명령어가 작동하지 않는다면, 대부분 rc 버전 반복으로 인한 것이니 —— 공식 changelog를 기준으로 삼으세요.

이 새로운 생태계에서 여러분의 자리를 먼저 차지하시길 바랍니다. 🚀

---

## 실습 문제(진짜로 이해했는지 검증)

1. **이해 문제**: 원문을 보지 않고 dsh 생태계의 3가지 공식 채널(저장소/API 문서/커뮤니티)이 각각 무엇에 쓰이는지 말해보세요
   > 정답 확인: 본 장 7.1절 공식 채널 표 참고
2. **이해 문제**: 공식은 현재 "외부 PR을 받지 않지만", 어떤 3가지 참여 방식을 장려하나요? 왜 "dsh-plugin 생태계 프로젝트 만들기"가 공식이 명시적으로 인정하는 방식일까요?
   > 정답 확인: 본 장 7.1절 "하지만 다음은 장려합니다" 단락 참고
3. **실습 문제**: GitHub에서 `topic:dsh-plugin`을 검색해, 찾은 커뮤니티 플러그인 3개를 나열하고 각각의 포지셔닝을 말해보세요
   > 정답 확인: 본 장 7.3절 "플러그인 찾기" 단락 + 7.7절 검색 채널 표 참고
4. **실습 문제**: 본 장 7.6절 단계에 따라 `DSH-better-sidebar`를 완전히 장착하거나, 제4장 예제를 따라 직접 속도 개선 플러그인을 작성해, `pnpm install` 출력과 검증 결과를 기록하세요
   > 정답 확인: 본 장 7.6절 두 곳 수정의 전체 코드 조각 참고
5. **실습 문제**: `DSH-better-sidebar`나 직접 작성한 속도 개선 플러그인의 README에 개선 PR을 하나 제출해보세요(예: 설치 단계 추가, 오타 수정), 커뮤니티 PR 흐름을 체험해보세요
   > 정답 확인: 본 장 7.4절 2단계 + 제5장 사례의 PR 패턴 참고
6. **사고 문제**: 본 장 7.9절 타임라인에서 "공식 패키지 60+ 동시 공개"와 "커뮤니티 플러그인 폭발"이 같은 날 일어났습니다. 이것이 오픈소스 프로젝트의 생태계 전략에 어떤 시사점을 주나요?
   > 정답 확인: 본 장 7.9절 타임라인 표 + 7.10절 연계 패턴 참고

## 자주 묻는 질문 FAQ

**Q1: 공식이 "외부 PR을 받지 않는다"고 하는데, 제 플러그인 코드는 어디에 두어야 하나요?**
여러분 자신의 GitHub 저장소에 독립적인 dsh-plugin 생태계 프로젝트로 두세요. 공식은 이 방식을 장려합니다 —— 플러그인으로 능력을 확장하며, dsh 코어를 수정할 필요가 없습니다. 저장소에 `dsh-plugin` topic을 추가하고 npm에 게시하면 됩니다.

**Q2: Discord와 Discussions는 무엇이 다른가요?**
Discord는 실시간 토론에 적합하고; Discussions는 공식 제안, 기능 건의, 도움 요청에 적합합니다(기록으로 조회 가능). 공식이 현재 권장하는 기여 채널은 Discussions입니다.

**Q3: 한국어(중국어) 튜토리얼/블로그를 쓰고 싶은데, 형식 요구사항이 있나요?**
공식 형식 요구사항은 없습니다. 설치 단계, 코드 예시, 실기 스크린샷/로그를 포함할 것을 권장합니다. 본 백서의 구조(TL;DR + 단계별 설명 + 실습)를 참고할 수 있습니다.

**Q4: 생태계 초일이 무슨 뜻인가요? 왜 "선점 우위"라고 하나요?**
생태계 초일 = 생태계가 막 시작해서 콘텐츠/플러그인/튜토리얼이 거의 없는 상태. 선점 우위 = 초기 진입자가 "특정 방향의 표준"이 되기 쉬움. 비유: 2015년 React 튜토리얼을 쓴 사람, 2018년 VS Code 플러그인을 만든 사람.

**Q5: rc 단계는 반복이 빠른데, 제 플러그인이 다 만들자마자 구식이 되지 않을까요?**
그럴 수 있습니다. 리스크를 낮추는 법: ① 의존성은 `^0.1.0-rc.6` 라인 사용; ② 공식 changelog를 주시; ③ 로직을 순수 함수로 격리하고 연결 레이어를 얇게 유지해, 업그레이드 시 연결 레이어만 수정하면 되도록.

**Q6: dsh 생태계 프로젝트를 만들고 싶은데, 어디서부터 시작해야 하나요?**
"여러분 자신의 페인 포인트"에서 시작하세요. 예를 들어: ① TUI가 필요하다면 → tui profile 플러그인 만들기; ② 토큰 추적이 필요하다면 → cost-tracker 플러그인 만들기; ③ diff 뷰가 필요하다면 → diff 플러그인 만들기. 본 장 7.5절 읽기 순서를 참고하세요.

**Q7: 7.6절의 `link:` 경로는 Windows와 macOS에서 작성법이 같나요?**
다릅니다. Windows는 이중 백슬래시로 이스케이프하고(`link:C:\\path\\to\\plugin`), macOS/Linux는 정슬래시를 씁니다(`link:/path/to/plugin`). 커뮤니티 플러그인은 최종적으로 npm 게시로 가서 경로 차이를 없애는 것을 권장합니다.

**Q8: 공식 디스커션에 게시할 때 팁이 있나요? 공식이 더 빨리 응답하게 할 수 있나요?**
7.10절의 3가지 원칙을 참고하세요: 환경 정보 포함, 재현 단계 포함, 먼저 검색해 중복을 피하기. 기능 건의는 "어떻게 구현할지"보다 "무슨 문제를 해결하는지"를 설명하는 것이 더 중요합니다.
