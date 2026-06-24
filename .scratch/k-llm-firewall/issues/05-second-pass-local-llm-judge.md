# 05 — 2차 로컬 LLM 검사 (Ollama)

Status: ready-for-agent

> 출처: `docs/PRD.md` (F4-2~F4-3, 성공기준 ④), `docs/TASKS.md` (Phase 4).

## What to build

1차 룰에서 명확히 판정되지 않은 애매한 입력만 사내 로컬 LLM(Ollama)으로 넘겨 자연어 의도를 정밀 분석한다. 2차는 마스킹 전 원문을 다루므로 절대 외부로 보내지 않고 로컬에서만 수행한다. 검사기 오류 시에는 `FAIL_SAFE` 정책(기본 차단 또는 명시적 통과)을 따른다.

## Acceptance criteria

- [ ] `app/guards/llm_judge.py` — Ollama 호출, 의도 분석 프롬프트 → 판정(공격/정상) + 사유 JSON 반환
- [ ] 1차에서 애매한 경우만 2차로 라우팅하는 로직 (명백한 통과/차단은 2차 생략)
- [ ] `FAIL_SAFE` 정책 적용 (LLM 오류·타임아웃 시 config 기준 차단/통과, 에러 삼키지 않음)
- [ ] 2차 검사는 로컬 Ollama만 사용 — 마스킹 전 원문 외부 전송 없음(코드·테스트로 보장)
- [ ] **검증**: 우회적으로 표현된 공격(1차 미탐) → 2차에서 차단 / 정상 애매 입력 → 통과
- [ ] Ollama 모델명은 `.env`로 설정

## Blocked by

- 03 — 1차 룰 기반 입력 차단
