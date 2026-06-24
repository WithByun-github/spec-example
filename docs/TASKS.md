# TASKS — K-LLM Firewall 개발 작업 목록

> Claude Code가 따라갈 단계별 체크리스트. 완료하면 `[ ]` → `[x]`로 갱신.
> 위에서부터 순서대로 진행하면 항상 "돌아가는 상태"를 유지하도록 설계됨.

---

## Phase 0. 프로젝트 셋업
- [ ] 디렉토리 구조 생성 (CLAUDE.md의 4번 참조)
- [ ] `requirements.txt` 작성 (fastapi, uvicorn, litellm, presidio-analyzer, presidio-anonymizer, llm-guard, streamlit, pyyaml, pydantic)
- [ ] `.env.example` 작성 (ARCHITECTURE.md 6번 참조)
- [ ] `config.py` — env 로드 + 기본값
- [ ] `README.md` — 실행 방법 (게이트웨이 서버 / 플랫폼 화면)

## Phase 1. 게이트웨이 골격 (가장 먼저 "돌아가는 것" 만들기)
- [ ] `main.py` — FastAPI 앱, `POST /v1/chat/completions` (OpenAI 호환 스키마)
- [ ] `llm_client.py` — LiteLLM로 보호 대상 LLM(SaaS 기본) 호출 (검사 없이 단순 패스스루부터)
- [ ] `gateway.py` — 패스스루 흐름 (입력 → LLM → 출력) 먼저 완성
- [ ] **검증**: curl로 요청 → 응답 정상 수신 확인

## Phase 2. 1차 룰 기반 입력 검사
- [ ] `rules/injection_ko.yaml` — 한국어 인젝션/탈옥 패턴 초안 (10~20개)
- [ ] `guards/rules.py` — yaml 로드 + 정규식 매칭 → verdict 반환
- [ ] `guards/decoder.py` — Base64 등 복원 후 재검사
- [ ] `guards/input_guard.py`에 1차 검사 연결
- [ ] gateway에 입력 검사 연결 (명백한 공격 즉시 차단)
- [ ] **검증**: "이전 지시 무시하고..." → 차단 / 정상 질문 → 통과

## Phase 3. 한국형 PII 마스킹 (입출력 양방향)
- [ ] `guards/pii_ko.py` — Presidio + 한국형 패턴(주민번호·휴대폰·카드·사업자번호·계좌·여권)
- [ ] 체크섬 검증으로 오탐 감소 (주민번호 등)
- [ ] 입력단 마스킹 (LLM 도달 전) + 출력단 마스킹 (사용자 노출 전)
- [ ] gateway 입력·출력 검사에 연결
- [ ] **검증**: 주민번호 포함 입력 → 마스킹 / 응답에 주민번호 → 마스킹

## Phase 4. 2차 로컬 LLM 검사
- [ ] `guards/llm_judge.py` — Ollama 호출, 의도 분석 프롬프트 (공격/정상 + 사유 JSON)
- [ ] 1차에서 애매한 경우만 2차로 라우팅하는 로직
- [ ] `FAIL_SAFE` 정책 적용 (LLM 오류 시 차단/통과)
- [ ] **검증**: 우회적으로 표현된 공격 → 2차에서 차단

## Phase 5. 출력 필터 + 정책 엔진
- [ ] `guards/output_guard.py` — 응답 PII 마스킹 + 사내 기밀/RAG 누출 패턴 + 세션 반향 탐지
- [ ] `guards/policy.py` — 역할 이탈 / off-topic / 컴플라이언스 (정책은 config/yaml로)
- [ ] gateway 출력 검사에 연결
- [ ] **검증**: 응답에 기밀 패턴 주입 시 마스킹

## Phase 6. 로그 + 탐지 대시보드(플랫폼 모듈)
- [ ] `store.py` — SQLite `detections` 테이블 (ARCHITECTURE 4번 스키마)
- [ ] 모든 검사 판정을 로깅 (마스킹된 형태로만)
- [ ] `platform/modules/firewall.py` — Streamlit: 탐지 추이/유형 분포/차단·통과 비율/최근 목록
- [ ] 알림 stub (`ALERT_WEBHOOK_URL` 비면 콘솔)
- [ ] **검증**: 공격 몇 건 발생 → Firewall 모듈 화면에 반영 확인

## Phase 6.5. 내재화 플랫폼 셸
- [ ] `platform/app.py` — 좌측 내비 + 모듈 라우팅 셸
- [ ] `platform/modules/solutions.py` — 솔루션 목록/통합 현황 (Firewall=실제, 후속=목업)
- [ ] `platform/modules/settings.py` — 관리 설정 (검사 동작·룰·정책·알림·LLM 설정)
- [ ] `platform/modules/dev_status.py` — 개발 현황/내재화 진척 + AI Agent 산출물 (일부 목업)
- [ ] `platform/mock/` — 후속 솔루션 목업 데이터
- [ ] **검증**: 플랫폼에서 Firewall 모듈은 실데이터, 나머지는 목업으로 표시

## Phase 7. AI Agent 자동화 (가설 실증 — 데모 하이라이트)
- [ ] `tests/attack_scenarios/` — 공격 시나리오 자동 생성 스크립트 (자동 테스트 Agent)
- [ ] 탐지율·오탐율 자동 리포팅 (콘솔/마크다운 리포트)
- [ ] (선택) 룰 작성 Agent: 미탐 공격 → 룰 yaml 초안 자동 생성 데모
- [ ] **검증**: 시나리오 일괄 실행 → 탐지율 리포트 출력

## Phase 8. 데모 준비
- [ ] `docs/DEMO.md` 시나리오대로 리허설
- [ ] 합성 샘플 데이터 정리 (실제 개인정보 금지)
- [ ] 한 번에 실행되는 데모 스크립트 / 실행 가이드

---

## 진행 원칙
1. **항상 돌아가는 상태 유지**: Phase 1에서 패스스루부터 완성하고 검사를 점진 추가.
2. **각 Phase 끝의 "검증"을 통과해야 다음으로**.
3. 룰·정책은 코드가 아닌 yaml/config로.
4. 막히면 `docs/PRD.md`의 성공 기준으로 우선순위 판단.
