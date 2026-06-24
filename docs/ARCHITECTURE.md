# ARCHITECTURE — K-LLM Firewall

> 시스템 구조와 데이터 흐름. Claude Code가 모듈 간 책임을 헷갈리지 않도록 정의한다.

---

## 1. 기본 컨셉

기존 LLM 서비스 코드를 **수정하지 않고**, 사용자와 LLM 사이에 **보안 검사 관문(Security Gateway)** 을 끼운다.

```
                     ┌─────────────────────────────────────────┐
                     │          K-LLM Firewall (Gateway)         │
   [사용자/챗봇앱]    │                                           │  [보호 대상 LLM]
        │            │   ① 입력검사 → ② LLM호출 → ③ 출력검사       │  (주로 SaaS,
        │  요청       │                                           │   예: ChatGPT)
        └───────────▶│  (OpenAI 호환 /v1/chat/completions)        │───────▶│
        ◀───────────│                          ▲ 마스킹된 입력만  │◀───────│
           응답       │              ↓ 모든 판정 로깅              │
                     │   ↑ 2차 검사는 사내 로컬 LLM(Ollama)만      │
                     └──────────────────┬───────────────────────┘
                                        │
                       [SQLite] ─┬─ [내재화 플랫폼 (Streamlit)]
                                 └─ Firewall 모듈 = 탐지 대시보드
```

게이트웨이는 **OpenAI Chat Completions API와 동일한 인터페이스**를 노출한다.
→ 기존 챗봇은 `base_url`만 게이트웨이로 바꾸면 끝 (드롭인 방식).
→ 보호 대상 LLM은 대개 외부 SaaS다. 사내 데이터가 SaaS로 새는 것을 막는 것이 목적이므로, ②로는 ①에서 마스킹·검사를 통과한 입력만 보낸다. 반면 마스킹 전 원문을 다루는 **2차 검사는 사내 로컬 LLM(Ollama)만** 사용한다.

## 2. 요청 처리 흐름 (상세)

```
[사용자 입력 수신]
      │
      ▼
┌──────────────── ① 입력 검사 (input_guard) ────────────────┐
│ 1. decoder: Base64 등 인코딩 변형 복원                      │
│ 2. rules (1차): 공격 키워드/패턴 사전 매칭                   │
│      ├─ 명백한 공격  → 즉시 차단 (로그 + 알림)              │
│      └─ 애매함       → 2차로                                │
│ 3. llm_judge (2차): Ollama 로컬 LLM이 의도 정밀 분석         │
│      └─ 공격 판정    → 차단                                 │
│ 4. pii_ko: 한국형 개인정보 탐지 → 마스킹 (LLM 도달 전)       │
│ 5. policy: 역할 이탈 / off-topic / 컴플라이언스 위반 검사    │
└────────────────────────────────────────────────────────────┘
      │ (통과 / 마스킹된 안전한 입력)
      ▼
┌──────────────── ② LLM 호출 (llm_client) ──────────────────┐
│ LiteLLM 통해 보호 대상 LLM 호출 (주로 외부 SaaS)            │
│ → 여기로는 마스킹·검사를 통과한 안전한 입력만 전달          │
└────────────────────────────────────────────────────────────┘
      │ (LLM 응답)
      ▼
┌──────────────── ③ 출력 검사 (output_guard) ───────────────┐
│ 1. pii_ko: 응답 내 한국형 개인정보 마스킹 (사용자 노출 전)   │
│ 2. 사내 기밀/RAG 누출 패턴 탐지 → 마스킹/차단               │
│ 3. 세션 반향(다른 사용자 입력 노출) 탐지                     │
└────────────────────────────────────────────────────────────┘
      │
      ▼
[최종 응답 반환]  + 전 과정 SQLite 로깅
```

## 3. 모듈 책임 (단일 책임 원칙)

| 모듈 | 책임 | 입력 → 출력 |
| --- | --- | --- |
| `main.py` | FastAPI 진입점, OpenAI 호환 엔드포인트 노출 | HTTP 요청 → HTTP 응답 |
| `gateway.py` | 검사 순서 오케스트레이션 (①②③ 조율) | 요청 → 검사·호출·로깅 조율 |
| `guards/decoder.py` | 인코딩 우회 복원 | 텍스트 → 복원 텍스트 + 탐지 플래그 |
| `guards/rules.py` | 1차 룰 기반 검사 | 텍스트 → {verdict, matched_rule, confidence} |
| `guards/llm_judge.py` | 2차 로컬 LLM 의도 분석 | 텍스트 → {verdict, reason} |
| `guards/pii_ko.py` | 한국형 PII 탐지·마스킹 (Presidio 확장) | 텍스트 → 마스킹 텍스트 + 탐지 항목 |
| `guards/input_guard.py` | 입력 검사 파이프라인 묶음 | 입력 → {allowed, masked_text, reasons} |
| `guards/output_guard.py` | 출력 검사 파이프라인 묶음 | 응답 → {masked_response, reasons} |
| `guards/policy.py` | 정책 엔진 (역할/off-topic/컴플라이언스) | 텍스트+정책 → {verdict, reason} |
| `llm_client.py` | LiteLLM로 보호 대상 LLM(SaaS 기본)·2차 검사 로컬 LLM 호출 | 메시지 → LLM 응답 |
| `store.py` | SQLite 로그 CRUD | 판정 이벤트 → 저장/조회 |
| `config.py` | 설정 로드 (env, 임계값, 룰 경로) | — |
| `platform/app.py` | 내재화 플랫폼 셸 (좌측 내비 + 모듈 라우팅) | — |
| `platform/modules/firewall.py` | Firewall 탐지 대시보드 모듈 | SQLite → 화면 |
| `platform/modules/{solutions,settings,dev_status}.py` | 솔루션 목록·관리 설정·개발 현황 (Firewall 외 일부 목업) | — / 설정값 |

## 4. 검사 결과 데이터 모델 (로그 스키마)

`store.py`가 관리하는 SQLite `detections` 테이블(예시):

| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| id | INTEGER PK | |
| ts | TEXT | ISO8601 타임스탬프 |
| direction | TEXT | input / output |
| stage | TEXT | rule(1차) / llm(2차) / pii / policy |
| verdict | TEXT | allow / block / mask |
| category | TEXT | injection / jailbreak / encoding / pii / off_topic ... |
| reason | TEXT | 판정 사유 (룰명 또는 LLM 분석 요약) |
| matched | TEXT | 매칭된 패턴/룰 ID |
| sample | TEXT | **마스킹된** 입력/응답 일부 (평문 PII 저장 금지) |
| latency_ms | INTEGER | 처리 시간 |
| session_id | TEXT | 세션 식별 (반향 탐지용) |

## 5. 룰 파일 구조 (`rules/injection_ko.yaml`)

룰은 코드가 아닌 데이터. AI Agent가 초안을 자동 생성하고 담당자가 승인하는 컨셉.

```yaml
rules:
  - id: ko-inj-001
    category: injection
    description: "이전 지시 무시 유형"
    patterns:
      - "이전\\s*(지시|명령|프롬프트).*(무시|잊)"
      - "ignore.*(previous|above).*instruction"
    severity: high
    action: block
  - id: ko-jb-001
    category: jailbreak
    description: "역할극 탈옥"
    patterns:
      - "너는\\s*이제.*(제한|규칙).*(없|해제)"
    severity: high
    action: block
```

## 6. 설정 (`.env`) 핵심 항목

```
# 보호 대상 LLM (LiteLLM 형식) — 주로 외부 SaaS, 사내 로컬도 가능
TARGET_LLM_BASE_URL=https://api.openai.com/v1   # 데모: Ollama로 대체 가능
TARGET_LLM_MODEL=...

# 2차 검사용 로컬 LLM (Ollama)
JUDGE_LLM_BASE_URL=http://localhost:11434/v1
JUDGE_LLM_MODEL=...

# 검사 동작
FAIL_SAFE=block            # 검사기 오류 시 기본 동작 (block | allow)
ENABLE_SECOND_PASS=true    # 2차 검사 사용 여부
RULES_PATH=rules/injection_ko.yaml
DB_PATH=detections.db
ALERT_WEBHOOK_URL=         # 비우면 콘솔 알림
```

## 7. 확장 포인트 (후속 내재화로 이어지는 설계)

- 검사기는 모두 동일한 인터페이스(`verdict`, `reason`)를 반환 → 새 검사기 추가가 쉬움.
- 로그 스키마가 통일 → 후속 "로그분석 시스템"이 이 데이터를 그대로 소비 가능.
- 룰을 데이터로 분리 → 룰 작성 Agent가 자동으로 갱신 가능.
