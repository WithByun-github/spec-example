# 08 — 내재화 플랫폼 셸

Status: ready-for-agent

> 출처: `docs/PRD.md` (F8, 성공기준 ⑤), `docs/TASKS.md` (Phase 6.5).

## What to build

K-LLM Firewall을 품는 상위 통합 관리 화면을 만든다. 좌측 내비 + 모듈 라우팅 구조로, Firewall 모듈만 실데이터에 연결하고 나머지 영역(후속 솔루션·일부 개발 현황)은 목업으로 비전을 전달한다. "단일 솔루션이 아니라 내재화 플랫폼의 첫 조각"이라는 스토리를 화면으로 보여주는 것이 목적이다.

## Acceptance criteria

- [ ] `platform/app.py` — 좌측 내비 + 모듈 라우팅 셸
- [ ] `platform/modules/solutions.py` — 솔루션 목록/통합 현황 (K-LLM Firewall=실제, 후속=목업 카드)
- [ ] `platform/modules/settings.py` — 관리 설정 (검사 동작 2차 on/off·fail-safe, 룰 관리, 정책, 알림 채널, LLM 설정)
- [ ] `platform/modules/dev_status.py` — 개발 현황/내재화 진척 + AI Agent 산출물 (일부 목업)
- [ ] `platform/mock/` — 후속 솔루션 목업 데이터
- [ ] 07의 Firewall 모듈을 셸 안의 한 탭으로 임베드
- [ ] **검증**: 플랫폼에서 Firewall 모듈은 실데이터, 나머지는 목업으로 표시

## Blocked by

- 07 — 로그 + Firewall 탐지 대시보드 모듈
