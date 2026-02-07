# Agent Teams 핵심 개념

## 아키텍처

Agent Teams는 4가지 핵심 컴포넌트로 구성됩니다.

### 1. Team Lead (팀 리더)

메인 Claude Code 세션으로, 다음 역할을 수행합니다:
- 팀 생성 및 Teammate 소환
- 작업 조율 및 방향 설정
- 전체 워크플로우 관리

### 2. Teammates (팀원)

독립적인 Claude Code 인스턴스로, 각각 자신만의 context window를 가집니다:
- 할당된 태스크 독립 실행
- 다른 팀원과 직접 메시지 교환
- Task List에서 작업 선택 및 완료 처리

### 3. Task List (태스크 목록)

공유 상태 관리 시스템:

**상태 전이:**
```
pending → in_progress → completed
```

**의존성 관리:**
- `blocked`: 다른 태스크 완료 대기
- `blockedBy`: 어떤 태스크가 차단하는지 명시

**Race Condition 방지:**
- 파일 잠금(file locking)으로 동시 수정 방지
- 원자적(atomic) 상태 변경 보장

### 4. Mailbox (메일박스)

메시징 시스템:
- **message**: 1:1 직접 메시지
- **broadcast**: 1:all 전체 공지
- 자동 배달(automatic delivery)

### 아키텍처 다이어그램

```
┌──────────────────────────────────────────────────────────────┐
│                        Team Lead                             │
│                   (Main Claude Code)                         │
│                                                              │
│  • 팀 생성 및 조율                                              │
│  • 워크플로우 관리                                              │
│  • Teammate 소환                                              │
└────────┬─────────────────────────────────────────┬───────────┘
         │                                         │
         │ creates/coordinates                     │ reads/writes
         ▼                                         ▼
┌─────────────────────┐              ┌──────────────────────────┐
│    Task List        │◄─────────────┤      Mailbox             │
│                     │              │                          │
│ • pending           │              │ • message (1:1)          │
│ • in_progress       │              │ • broadcast (1:all)      │
│ • completed         │              │ • auto delivery          │
│                     │              │                          │
│ Dependencies:       │              └──────────┬───────────────┘
│ • blocked           │                         │
│ • blockedBy         │                         │ exchange
│                     │                         │ messages
│ File locking        │                         │
└────┬────────┬───────┘                         │
     │        │                                 │
     │ pick   │ update                          │
     │ tasks  │ status                          │
     │        │                                 │
┌────▼────────▼──────┐    ┌────────────────┐   │
│   Teammate 1       │◄───┤   Teammate 2   │◄──┘
│                    │    │                │
│ • Own context      │◄──►│ • Own context  │
│ • Independent      │    │ • Independent  │
│   execution        │    │   execution    │
└────────────────────┘    └────────────────┘
         ▲                        ▲
         │                        │
         └────────┬───────────────┘
                  │
            Direct messaging
           (peer-to-peer)
```

---

## Subagents vs Agent Teams 비교

| 측면 | Subagents | Agent Teams |
|------|-----------|-------------|
| **Context** | 독립 context, 결과만 호출자에게 반환 | 독립 context, 완전 자율 |
| **Communication** | 메인 에이전트에게만 보고 | 팀원 간 직접 메시지 교환 |
| **Coordination** | 메인 에이전트가 모든 작업 관리 | 공유 Task List, 자율 조율 |
| **Best for** | 집중된 태스크, 결과만 중요 | 논의 필요한 복잡한 작업 |
| **Token cost** | 낮음 (결과 요약됨) | 높음 (각각 독립 인스턴스) |
| **Session** | 단일 세션 내 | 여러 독립 세션 |
| **병렬 실행** | 순차적 (메인이 하나씩 호출) | 진정한 병렬 (동시 작업) |
| **컨텍스트 오염** | 메인 세션에 영향 가능 | 완전 격리 |
| **실행 시점** | 메인이 명시적으로 호출 | 자율적으로 Task 선택 |
| **의존성 처리** | 메인이 순서 결정 | Task List의 blocked/blockedBy |

### 선택 기준

**Subagents를 사용하세요:**
- 결과만 필요하고 과정은 중요하지 않을 때
- 순차적 작업 흐름
- 토큰 비용 최소화가 중요할 때
- 단순/반복 작업

**Agent Teams를 사용하세요:**
- 팀원 간 논의/협업이 필요할 때
- 진정한 병렬 작업이 필요할 때
- 컨텍스트 격리가 중요할 때
- 복잡한 의존성 관리가 필요할 때

---

## 적합한 시나리오

### 1. Research & Review (병렬 조사)

**예시**: 새 라이브러리 평가
```
Teammate 1: React Query 조사
Teammate 2: SWR 조사
Teammate 3: Apollo Client 조사

→ 각자 독립 조사 후 Mailbox로 결과 공유
→ Lead가 최종 의사결정
```

**장점**: 각 팀원이 독립 컨텍스트에서 깊이 있게 조사

### 2. New Modules/Features (독립 모듈)

**예시**: 새 기능 개발
```
Teammate 1: Backend API 개발
Teammate 2: Frontend UI 개발
Teammate 3: 테스트 코드 작성

Task dependencies:
- UI는 API 완료 후 (blocked by API task)
- 테스트는 둘 다 완료 후
```

**장점**: 병렬 개발, 자동 의존성 관리

### 3. Debugging with Competing Hypotheses (가설 검증)

**예시**: 성능 저하 원인 분석
```
Teammate 1: 데이터베이스 쿼리 조사
Teammate 2: 네트워크 레이턴시 조사
Teammate 3: 메모리 누수 조사

→ 각자 가설 검증 후 결과 공유
→ 가장 유력한 원인 도출
```

**장점**: 여러 가설을 동시에 검증

### 4. Cross-Layer Coordination (계층 간 협업)

**예시**: Full-stack 기능 구현
```
Teammate 1 (Backend): API endpoint 개발
Teammate 2 (Frontend): UI 컴포넌트 개발
Teammate 3 (DevOps): 배포 스크립트 작성

→ Mailbox로 API 스펙 협의
→ Task List로 완료 순서 조율
```

**장점**: 전문 영역별 독립 작업, 필요 시 협의

---

## 부적합한 시나리오

### 1. Sequential Tasks with Dependencies (순차적 의존 작업)

**예시**: 데이터베이스 마이그레이션
```
Step 1: 스키마 설계
Step 2: 마이그레이션 스크립트 작성 (Step 1 필요)
Step 3: 롤백 스크립트 작성 (Step 2 필요)
Step 4: 테스트 작성 (Step 3 필요)
```

**문제점**: 순차 의존성이 강해 병렬화 불가능
**대안**: 단일 Agent 또는 Subagent로 순차 실행

### 2. Same-File Edits (동일 파일 수정)

**예시**: 한 파일에 여러 기능 추가
```
Teammate 1: utils.ts에 formatDate 추가
Teammate 2: utils.ts에 parseJSON 추가
```

**문제점**: 파일 덮어쓰기로 충돌 발생
**대안**: 단일 Agent가 한 번에 처리

### 3. Simple/Routine Tasks (단순 반복 작업)

**예시**: 로그 메시지 번역
```
100개 파일의 console.log를 한국어로 번역
```

**문제점**: 오버헤드 > 실제 이득
**대안**: 단일 Agent의 반복문 또는 스크립트

### 4. Real-time Interactive Work (실시간 대화 작업)

**예시**: 사용자와 요구사항 정리
```
User: "로그인 기능이 필요해요"
Agent: "OAuth? 자체 인증?"
User: "OAuth로..."
```

**문제점**: 팀원들이 개별적으로 사용자와 대화하면 혼란
**대안**: Lead가 요구사항 정리 후 Team 소집

---

## 토큰 비용 고려사항

### 비용 계산

```
총 비용 ≈ (팀원 수 + 1) × 기본 비용

예시:
- Lead만: 1× 비용
- Lead + 3 teammates: 4× 비용
```

### 비용 발생 시점

- **팀 활성화 시**: 각 Teammate가 독립 context window 생성
- **작업 진행 중**: 각자 독립적으로 토큰 소비
- **메시지 교환**: Mailbox 읽기/쓰기도 토큰 소비

### 비용 최적화 전략

#### 1. 팀원 수 최소화

```
✅ 좋은 예:
3-4명 팀원 (각자 명확한 역할)

❌ 나쁜 예:
10명 팀원 (역할 중복, 조율 오버헤드)
```

#### 2. 작업 범위 명확화

```
✅ 좋은 예:
Task: "UserService 단위 테스트 작성"

❌ 나쁜 예:
Task: "뭔가 테스트 관련 일 해봐"
```

#### 3. Environment Variable Toggle

```bash
# Agent Teams 비활성화 (비용 0)
export CLAUDE_AGENT_TEAMS_ENABLED=false

# 필요 시에만 활성화
export CLAUDE_AGENT_TEAMS_ENABLED=true
```

**효과**: 비활성화 시 Agent Teams 기능 완전 차단 → 추가 비용 0

#### 4. 작업 종료 후 팀 해체

```
TodoWrite task:
- title: "팀 작업 완료 후 해체"
  status: in_progress

→ 완료 시 Teammate 세션 종료 → 비용 절감
```

### ROI (투자 대비 효과) 판단

**높은 ROI**:
```
• 병렬 작업으로 시간 대폭 단축
• 컨텍스트 격리로 품질 향상
• 복잡한 협업 필요

→ 비용 증가 < 시간/품질 개선
```

**낮은 ROI**:
```
• 순차 작업 (병렬화 불가)
• 단순 반복 작업
• 협업 불필요

→ 비용 증가 > 실제 이득
```

---

## 결론

Agent Teams는 **진정한 병렬 협업**이 필요한 복잡한 프로젝트에서 빛을 발합니다.

**핵심 원칙**:
1. 병렬 작업 가능 여부 확인
2. 팀원 간 협업 필요성 판단
3. 토큰 비용 vs 시간/품질 이득 평가
4. 간단한 작업은 단일 Agent/Subagent 사용

**언제 사용할까?**
- 복잡도 > 단순 작업
- 병렬성 > 순차성
- 협업 필요 > 독립 작업
- ROI > 비용 증가
