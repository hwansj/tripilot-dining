# triPilot — 요식(Dining) 파트

SK네트웍스 Family AI 캠프 32기 최종 프로젝트 · 6인 팀 · 2026.08 ~ 2026.10 · 요식 에이전트 설계·구현

- 팀 저장소: [SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM)
- 내 커밋: [main 브랜치의 hwansj 커밋](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/commits/main?author=hwansj) · 내 PR: [author:hwansj](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/pulls?q=is%3Apr+author%3Ahwansj)

이 저장소는 코드 사본이 아니라, 팀 저장소에 있는 제 작업을 **사례별로 찾아볼 수 있게 정리한 색인**입니다. 아래 표의 링크는 모두 팀 저장소의 실제 커밋, 코드, 평가 보고서로 연결됩니다.

사용 기술: Python · FastAPI · PostgreSQL · MCP(FastMCP) · OpenAI API · Ollama · 카카오·구글 장소 API · pytest

## 무엇을 만들었나

여행 일정 속 식당이 그 시각에 실제로 이용 가능한지(영업·휴무·결제·동반 조건) 판정하고, 안 되면 동선 안에서 대체 식당을 제안하는 에이전트입니다. 식당 데이터는 약 1,900곳입니다.

기준으로 삼은 원칙은 하나입니다. **LLM은 읽고 판단하고, 코드는 계산하고, 승인은 고객이 합니다.**

```
고객 문장 ──▶ LLM: 확인할 항목 고르기 · 도구 부르기
                     │
                     ▼
            MCP 도구 (규칙 6 · 조회 3)
            영업시간 · 휴무 · 결제 · 동반 · 장소 찾기 …
                     │
                     ▼
            서버 근거 검증 ── 판정이 그 판정을 낼 수 있는 도구의 결과를 인용했는가?
                     │            아니면 "확인 필요"
                     ▼
              접수 확인 화면 (승인은 고객)
```

| 구성 | 코드 |
|---|---|
| MCP 도구 (FastMCP 서버 `build_mcp`) | [agent_tools.py](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/blob/main/final_project_cs/app/domains/travel_ops/instances/dining/agent_tools.py) |
| 에이전트 루프 · 근거 검증 | [agent.py](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/blob/main/final_project_cs/app/domains/travel_ops/instances/dining/agent.py) |
| 모델 연결 (OpenAI · Ollama 공통 인터페이스) | [tool_chat.py](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/blob/main/final_project_cs/app/infrastructure/llm/tool_chat.py) |
| 평가 실행기 | [dining_agent.py](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/blob/main/final_project_cs/eval/runners/dining_agent.py) · [dining_alternatives.py](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/blob/main/final_project_cs/eval/runners/dining_alternatives.py) |
| 설계 문서 | [LLM 에이전트 · MCP 판정 설계](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/blob/main/final_project_cs/wiki/records/plans/2026-10-13_1130_%EC%9A%94%EC%8B%9D_LLM%EC%97%90%EC%9D%B4%EC%A0%84%ED%8A%B8_MCP%ED%8C%90%EC%A0%95_%EC%84%A4%EA%B3%84.md) |

## 사례별 근거

| 사례 | 상황 → 판단 → 결과 | 근거 |
|---|---|---|
| 해석과 계산 나누기 | LLM이 직접 계산하면 영업 판정의 51%만 맞음, 규칙 코드는 자유문장 조건을 못 읽음 → LLM은 확인 항목만 고르고 계산은 규칙 도구가 맡음 → 같은 평가 세트에서 지금 코드 34/41, LLM 직접 98/123, 에이전트 + 규칙 도구 123/123(홀드아웃 66/66) | [커밋](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/commit/51cf1432f5301e0770df2ce55b5ef88d5f923ad2) · [비교 보고서](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/blob/main/final_project_cs/eval/reports/2026-10-13_dining_agent.md) |
| 판정 근거 검증 | '가게를 찾았다'는 결과를 근거로 든 '영업 가능'이, 같은 날 휴무 도구가 '휴무'를 낸 상태에서도 통과 → 체크마다 근거로 인정하는 도구를 정한 표(`CHECK_TOOLS`), 시작·끝 시각을 모두 물은 영업 조회만 인정 | [커밋](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/commit/5495e5f966f4f2ba0007a9337049a57de83a18d5) · [코드](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/blob/5495e5f966f4f2ba0007a9337049a57de83a18d5/final_project_cs/app/domains/travel_ops/instances/dining/agent.py#L38-L50) · ['모름'만 인용한 판정 거절](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/commit/c491a3aed672e6c093da64b0e1ac74a2d0283618) |
| 끝나지 않던 실제 접수 | 실제 접수(식당 4곳)가 10턴, 12턴 상한에서도 판정 0건 → 턴이 아니라 같은 가게를 다시 찾는 행동이 원인 → 재검색 차단 · 식사 수 맞춤 턴 상한 · 마지막 답 턴 → 7턴, 이후 17.1초 · 실패 0 | [커밋](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/commit/5495e5f966f4f2ba0007a9337049a57de83a18d5) |
| 대체 식당 순위 | 가격 정보가 비어 사실상 장소 ID 순 → 다음 일정에 늦는 곳은 빼고 동선 · 비슷함 순으로, 장소 ID를 5가지로 바꿔 모든 판에서 같을 때만 정답 인정 → 안정적인 정답 5/17 → 17/17, 동선 위반 2 → 0 | [평가 세트 커밋](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/commit/64713901176e321ed19520886a1460722ae877c8) · [수정 커밋](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/commit/8d08f1ba0a2ed48d1c84dd2f6790c62d1331d9b8) · [기준선 보고서](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/blob/main/final_project_cs/eval/reports/2026-10-07_dining_alternatives_v0.md) |
| 취향에 맞는 대체 식당 | 규칙만으로는 취향 사례 3/12 → 규칙이 거른 상위 3곳 안에서만 LLM이 고르고 서버가 답을 검증, 어기면 규칙 1등 → 10/12, 홀드아웃 6/6, 동선 위반 0 | [커밋](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/commit/51cf1432f5301e0770df2ce55b5ef88d5f923ad2) · [보고서](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/blob/main/final_project_cs/eval/reports/2026-10-13_dining_meal_pick.md) |
| 필수 조건을 선호로 읽음 | 독립 평가 110건에서 필수 조건 210개 중 16개를 선호로 읽음 → 사실만 말한 동반 · 결제 · 종교 조건도 필수로 읽도록 수정 | [측정 커밋](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/commit/c09784e5ae5e9291c3f94fabe7b958d5e834305c) · [수정 커밋](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/commit/9612ac71620815086750c897144cf03204e1aab6) |
| 같은 이름 다른 지점 | 좌표 차이로 같은 가게를 놓치거나 다른 지점에 잇던 문제 → 지점 표시를 뗀 이름이 같고 150m 안에 하나뿐일 때만 연결 | [커밋](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/commit/75fbd6924b28363ee123e36968b1c48aa97451bc) |
| 알리지 않던 실패 | 재시도 대상이 아닌 실패를 '운영자 몫'이라 알리지 않았지만 운영자가 없는 구조 → 같은 항목 · 원인 · 코드당 한 번 고객에게 알림 | [커밋](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/commit/3991cf7dec8d732012003b6940c3a6061610cbae) |
| 팀 DB를 되돌리던 마이그레이션 | 모든 SQL을 한 트랜잭션으로 실행해 요식 파일의 재실행 오류가 Core까지 롤백 → 생성문 · 키 충돌 · 함수 중복 정의 세 곳 수정 → 빈 DB 3회 연속 통과 | [커밋](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/commit/9bbd352ab867f345cc6cdfe68b934398573c2ab5) |
| 실시간 경로의 모델 | 같은 도구 인터페이스에서 gpt-4o-mini 평균 7.0초, 로컬 CPU의 Gemma 152초 → 로컬 모델은 실시간 경로에서 제외 → 이후 기본 모델을 gpt-6-luna로 바꾸며 도구 호출 설정(reasoning_effort)과 구조화 응답(strict json_schema)을 고치고 다시 측정(대체 식당 고르기 12/12), 7턴 실제 접수 시험도 이 모델로 진행 | [비교 보고서](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/blob/main/final_project_cs/eval/reports/2026-10-13_dining_agent.md) · [모델 교체 커밋](https://github.com/SKNETWORKS-FAMILY-AICAMP/SKN32-FINAL-6TEAM/commit/536f453939768c41d32199119ec4b8123091a161) |

## 통합 평가 (2026-10-20)

코드 사본을 고정하고 한 번에 실행한 평가입니다. 기준은 개발에 쓰지 않은 검증용 일정 40개의 확인 항목 111개 × 3회 = 333회 판정이고, 채점은 코드로 했습니다.

| 구성 | 정확도 | macro-F1 |
|---|---|---|
| 규칙만 | 79.3% (88/111) | 0.603 |
| 에이전트 + 규칙 도구 | **98.5%** (328/333) | **0.977** |

- 규칙 대비 McNemar p = 0.0001 · 닫힌 식당을 허용한 판정 0/66
- 자유문 조건 F1 0.974 (독립 작성 96과제, 규칙만은 재현율 0)
- 평가 데이터는 프롬프트를 고친 세션과 분리해 만들고, 측정 전에 해시로 고정했습니다.

이 통합 평가의 원문 보고서는 공개 저장소에 포함되어 있지 않습니다. 공개된 근거는 위 '사례별 근거' 표의 커밋과 보고서에서 확인할 수 있습니다.

## 남은 과제

사람과 지도로 대조한 정답에서 식당 원장 자체가 틀린 36과제는 에이전트도 0/36이었습니다. 원장이 자신 있게 틀린 경우를 잡으려면 실시간 조회 조건이나 원장 갱신 주기를 다시 설계해야 합니다.

## AI 코딩 도구 사용

코드 작성에는 Claude Code를 함께 썼습니다(커밋의 `Co-Authored-By`). 구조의 기준("LLM은 읽고 판단하고, 코드는 계산하고, 승인은 고객이")과 무엇을 평가할지는 제가 정했고, 커밋 메시지에 시험과 평가 결과를 함께 남겼습니다.
