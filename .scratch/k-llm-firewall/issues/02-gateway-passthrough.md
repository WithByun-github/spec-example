# 02 — 게이트웨이 패스스루

Status: ready-for-agent

> 출처: `docs/PRD.md` (F4 기반, 비기능 호환성), `docs/TASKS.md` (Phase 1). 첫 트레이서 불릿 — "항상 돌아가는 상태"의 출발점.

## What to build

검사 로직 없이 입력 → 보호 대상 LLM → 출력이 end-to-end로 흐르는 OpenAI 호환 게이트웨이를 만든다. 기존 챗봇이 `base_url`만 바꾸면 붙을 수 있는 드롭인 프록시의 골격이다. 이후 모든 검사 슬라이스는 이 흐름의 입력단·출력단에 끼워 넣는다.

## Acceptance criteria

- [ ] `app/main.py` — FastAPI 앱, `POST /v1/chat/completions` (OpenAI Chat Completions 스키마 호환)
- [ ] `app/llm_client.py` — LiteLLM로 보호 대상 LLM(SaaS 기본, 사내 로컬도 지원) 호출
- [ ] `app/gateway.py` — 패스스루 오케스트레이션 (입력 → LLM 호출 → 출력 반환)
- [ ] **검증**: `curl`로 `/v1/chat/completions` 요청 → 정상 응답 수신
- [ ] 보호 대상 엔드포인트는 `.env`로 설정, 미설정 시 mock 또는 로컬 모델로 동작

## Blocked by

- 01 — 프로젝트 셋업 & 스캐폴딩
