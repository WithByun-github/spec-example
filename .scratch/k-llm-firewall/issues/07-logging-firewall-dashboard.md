# 07 — 로그 + Firewall 탐지 대시보드 모듈

Status: ready-for-agent

> 출처: `docs/PRD.md` (F6, 성공기준 ⑤), `docs/TASKS.md` (Phase 6).

## What to build

모든 검사 판정을 SQLite에 기록하고, 그 데이터를 플랫폼의 Firewall 모듈(Streamlit) 한 화면으로 시각화한다. 로그에는 마스킹된 형태만 남긴다. 보안 담당자가 "어떤 공격이 언제 들어왔는지"를 한눈에 볼 수 있어야 한다. 위협 탐지 시 알림은 해커톤 범위에선 stub으로 충분하다.

## Acceptance criteria

- [ ] `app/store.py` — SQLite `detections` 테이블 (`docs/ARCHITECTURE.md` 4번 스키마: 타임스탬프·판정·사유·검사단계·처리시간·마스킹된 입력 일부)
- [ ] 모든 검사 판정(차단/통과)을 마스킹된 형태로만 로깅
- [ ] `platform/modules/firewall.py` — Streamlit: 탐지 추이 / 공격 유형 분포 / 차단·통과 비율 / 최근 탐지 목록 (실데이터 연동)
- [ ] 알림 stub (`ALERT_WEBHOOK_URL` 미설정 시 콘솔 출력)
- [ ] **검증**: 공격 몇 건 발생 → Firewall 모듈 화면에 반영 확인

## Blocked by

- 03 — 1차 룰 기반 입력 차단
