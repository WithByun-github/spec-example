# 01 — 프로젝트 셋업 & 스캐폴딩

Status: ready-for-agent

> 출처: `docs/PRD.md`, `docs/TASKS.md` (Phase 0). prefactor — 이후 모든 슬라이스의 기반.

## What to build

이후 슬라이스들이 바로 코드를 얹을 수 있도록 프로젝트 골격을 세운다. `CLAUDE.md` 4번의 디렉토리 구조를 만들고, 의존성·환경설정·실행 문서를 갖춘다. 동작하는 기능은 아직 없지만, `config.py`로 env와 기본값(임계값·룰 경로·FAIL_SAFE 등)을 한 곳에서 읽을 수 있어야 한다.

## Acceptance criteria

- [ ] `CLAUDE.md` 4번 기준 디렉토리 구조 생성 (`app/`, `app/guards/`, `rules/`, `platform/modules/`, `platform/mock/`, `tests/attack_scenarios/`)
- [ ] `requirements.txt` 작성 (fastapi, uvicorn, litellm, presidio-analyzer, presidio-anonymizer, llm-guard, streamlit, pyyaml, pydantic)
- [ ] `.env.example` 작성 (`docs/ARCHITECTURE.md` 6번 참조 — 보호 대상 LLM, 2차 검사 Ollama, FAIL_SAFE, 알림 웹훅 등)
- [ ] `app/config.py` — env 로드 + 기본값 제공 (임계값·룰 경로·FAIL_SAFE 정책)
- [ ] `README.md` — 게이트웨이 서버 / 플랫폼 화면 실행 방법
- [ ] `pip install -r requirements.txt` 후 `import` 에러 없이 `config` 로드 확인

## Blocked by

None - can start immediately
