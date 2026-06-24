# 06 — 출력 필터 + 정책 엔진

Status: ready-for-agent

> 출처: `docs/PRD.md` (F2-1~F2-3, F3), `docs/TASKS.md` (Phase 5).

## What to build

LLM 응답을 사용자에게 돌려주기 전에 검사한다. RAG·도구 호출로 주입된 사내 데이터 노출, 다른 세션 입력의 반향, 권한 없는 사용자에게 도달하는 민감정보를 마스킹/차단한다. 또한 정책 엔진으로 챗봇이 정의된 역할/도메인을 벗어나거나(off-topic), 컴플라이언스를 위반하는지 탐지한다. 정책·패턴은 코드가 아니라 yaml/config로 관리한다.

## Acceptance criteria

- [ ] `app/guards/output_guard.py` — 응답 내 사내 기밀/RAG 누출 패턴 + 세션 반향 탐지 + PII 마스킹 연계
- [ ] `app/guards/policy.py` — 역할 이탈 / off-topic / 컴플라이언스 위반 탐지 (정책은 config/yaml)
- [ ] gateway 출력단에 연결
- [ ] **검증**: 응답에 기밀 패턴 주입 시 마스킹 / 역할 벗어난 요청·응답 탐지
- [ ] 정책·패턴 하드코딩 금지 (yaml/config로만 조정, 탐지를 임의로 끄지 않음)

## Blocked by

- 04 — 한국형 PII 마스킹 (입·출력 양방향)
