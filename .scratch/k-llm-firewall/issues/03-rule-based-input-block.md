# 03 — 1차 룰 기반 입력 차단 (+ 인코딩 복원)

Status: ready-for-agent

> 출처: `docs/PRD.md` (F1-1~F1-3, F4-1, 성공기준 ①②), `docs/TASKS.md` (Phase 2).

## What to build

사용자 입력이 LLM에 도달하기 전, 알려진 한국어 공격 패턴을 룰로 즉시 차단한다. Base64 등으로 변형된 입력은 원문 복원 후 재검사한다. 명백한 공격은 빠르게(<50ms 목표) 막고, 정상 입력은 그대로 통과시켜 게이트웨이의 LLM 호출로 넘긴다. 룰은 코드가 아니라 yaml에서 로드해 코드 수정 없이 확장 가능해야 한다.

## Acceptance criteria

- [ ] `rules/injection_ko.yaml` — 한국어 인젝션/탈옥 패턴 초안 10~20개 (예: "이전 지시 무시하고…", "시스템 프롬프트 알려줘", DAN류 역할극)
- [ ] `app/guards/rules.py` — yaml 로드 + 정규식/키워드 매칭 → verdict(차단/통과 + 사유) 반환
- [ ] `app/guards/decoder.py` — Base64 등 복원 후 재검사
- [ ] `app/guards/input_guard.py`에 1차 검사 연결, gateway 입력단에 연결
- [ ] **검증**: "이전 지시 무시하고…" → 차단 / 정상 질문 → 통과 / Base64로 감싼 공격 → 복원 후 차단
- [ ] 룰은 하드코딩하지 않고 전부 `rules/*.yaml`에서 로드

## Blocked by

- 02 — 게이트웨이 패스스루
