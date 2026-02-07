# 팀 구성 패턴

이 문서는 Agent Teams를 효과적으로 사용하기 위한 검증된 팀 구성 패턴을 제공합니다.

---

## 패턴 1: 코드 리뷰 팀

### 개요

여러 관점에서 동시에 리뷰하여 단일 리뷰어의 편향을 극복합니다.

**해결하는 문제:**
- 단일 리뷰어는 자신의 전문 영역에만 집중하는 경향
- 보안, 성능, 테스트 품질을 모두 보기 어려움
- 리뷰 누락 영역 발생

**핵심 아이디어:**
각 teammate가 서로 다른 품질 차원을 담당하여 전체 커버리지 향상

### 팀 구성

| 역할 | 책임 | 체크 항목 |
|------|------|-----------|
| **보안 리뷰어** | 보안 취약점 탐지 | SQL injection, XSS, token handling, session management, 권한 검증 |
| **성능 리뷰어** | 성능 영향 분석 | query optimization, N+1 문제, memory usage, algorithmic complexity |
| **테스트 리뷰어** | 테스트 품질 검증 | coverage gaps, edge cases, test quality, 회귀 테스트 누락 |

### 프롬프트 예제

```
Create an agent team to review PR #142. Spawn three reviewers:
- One focused on security implications (check for injection, XSS, token handling)
- One checking performance impact (query optimization, memory usage)
- One validating test coverage (coverage gaps, edge cases, test quality)

Have them each review the PR independently and report findings.
```

### 변형: 도메인별 리뷰

```
Review the payment integration PR with teammates specialized in:
- PCI compliance and security standards
- Financial transaction edge cases (refunds, partial payments)
- Integration testing with payment gateway mocks
```

### 사용 시기

- 중요 기능의 PR 리뷰
- 보안/성능에 민감한 코드 변경
- 여러 레이어를 아우르는 큰 PR

---

## 패턴 2: 경쟁 가설 디버깅

### 개요

Single agent tends to find one explanation and stop. Multiple agents challenge each other's theories.

**해결하는 문제:**
- 단일 에이전트는 첫 번째 가설에 고착되는 경향
- 복잡한 버그는 여러 원인이 복합적으로 작용
- 확증 편향으로 대안 가설 탐색 부족

**핵심 아이디어:**
경쟁하는 가설을 동시에 조사하고, 서로의 이론에 반론을 제기하도록 함

### 팀 구성

| 역할 | 책임 |
|------|------|
| **가설 A 조사자** | 특정 가설을 증명/반증하는 증거 수집 |
| **가설 B 조사자** | 대안 가설을 증명/반증하는 증거 수집 |
| **가설 C 조사자** | 또 다른 가설 탐색 |
| **반론 담당 (devil's advocate)** | 각 가설의 약점을 지적하고 반례 제시 |

### 프롬프트 예제

**기본 패턴:**
```
Users report the app exits after one message.
Spawn 5 agent teammates to investigate different hypotheses:
- Hypothesis A: Memory leak in message handling
- Hypothesis B: Unhandled exception in response parsing
- Hypothesis C: Race condition in WebSocket connection
- Hypothesis D: Session timeout issue
- One devil's advocate to challenge each theory

Have them talk to each other to try to disprove each other's theories.
Present me with the most likely root cause backed by evidence.
```

**실제 사례:**
```
API endpoint returns 500 intermittently (10% of requests).
Create a debugging team with teammates investigating:
- Database connection pool exhaustion
- Race condition in concurrent request handling
- Timeout in external service call
- Memory pressure triggering GC pauses

Each teammate should gather logs, metrics, and reproduce the issue.
Have them debate which hypothesis best explains the 10% failure rate.
```

### 사용 시기

- 재현이 어려운 간헐적 버그
- 여러 컴포넌트가 연관된 복잡한 문제
- 로그만으로 원인 파악이 어려운 경우

### 주의사항

- 가설이 너무 많으면 조정 비용 증가 (3-5개 권장)
- 각 teammate가 독립적으로 증거를 수집하도록 방향 제시
- Lead는 최종 판단 전 모든 증거를 종합

---

## 패턴 3: Cross-layer 기능 개발

### 개요

Frontend, backend, tests를 각각 다른 teammate가 담당하여 병렬 개발

**해결하는 문제:**
- Full-stack 기능은 여러 레이어를 순차적으로 개발해야 함
- 단일 에이전트가 모든 레이어를 완성하려면 시간 소요
- 각 레이어의 전문성 부족

**핵심 아이디어:**
각 teammate가 서로 다른 파일을 소유하여 병렬 작업. **파일 충돌 방지가 핵심!**

### 팀 구성

| 역할 | 소유 파일 | 책임 |
|------|-----------|------|
| **API 레이어 담당** | `app/api/`, `controllers/` | API endpoint, request/response handling |
| **비즈니스 로직 담당** | `services/`, `models/` | Domain logic, validation, persistence |
| **UI 레이어 담당** | `components/`, `views/` | UI components, state management |
| **테스트 작성 담당** | `tests/` | Unit/integration tests for all layers |

### 프롬프트 예제

**기본 패턴:**
```
Implement a new user profile feature.
Spawn three teammates, each owning different files:

1. Backend teammate:
   - Files: app/api/profile.py, models/user_profile.py
   - Tasks: Create API endpoints, database schema, validation

2. Frontend teammate:
   - Files: components/ProfileCard.tsx, pages/ProfilePage.tsx
   - Tasks: Build UI, connect to API, handle loading/error states

3. Test teammate:
   - Files: tests/api/profile_test.py, tests/components/ProfileCard.test.tsx
   - Tasks: Write tests for all layers, verify integration

DO NOT let them edit the same file. Each teammate owns their layer.
```

**Plan Approval 적용 예제:**
```
Spawn an architect teammate to refactor the authentication module.
Require plan approval before they make any changes.

Plan should include:
- Which files will be modified
- What tests will be added
- How backward compatibility will be maintained

Only approve if the plan includes test coverage and migration strategy.
```

### 사용 시기

- 여러 레이어를 아우르는 신규 기능 개발
- 대규모 리팩토링 (각 레이어별 담당 분리)
- API + UI + Tests를 동시에 개발해야 할 때

### 주의사항

- **파일 소유권 명확히 정의**: 같은 파일을 두 teammate가 편집하면 충돌
- Lead가 레이어 간 인터페이스를 먼저 정의하는 것이 좋음
- 한 teammate의 작업이 다른 teammate의 전제 조건이라면 순서 지정 필요

---

## 패턴 4: 리서치/기술 조사

### 개요

Multiple perspectives investigating a technology decision

**해결하는 문제:**
- 기술 선택 시 단일 관점은 편향될 수 있음
- 장점만 보고 단점/리스크를 간과
- 대안 비교 없이 첫 번째 후보에 고착

**핵심 아이디어:**
현황 분석, 대안 리서치, 반론/리스크 분석을 각각 다른 teammate가 담당

### 팀 구성

| 역할 | 책임 |
|------|------|
| **현황 분석 teammate** | 현재 상태 파악, 문제점 정의, 요구사항 추출 |
| **대안 리서치 teammate** | 가능한 솔루션 조사, 각 옵션의 장단점 정리 |
| **반론/리스크 분석 teammate (devil's advocate)** | 각 대안의 약점, 도입 시 리스크, 숨겨진 비용 분석 |

### 프롬프트 예제

**기술 스택 선택:**
```
We need to choose a state management library for our React app.
Spawn three research teammates:

1. Current state analyst:
   - Analyze our existing Redux setup
   - Document pain points and requirements
   - Identify what features we actually need

2. Alternative researcher:
   - Research: Zustand, Jotai, Recoil, MobX
   - Compare bundle size, API complexity, TypeScript support
   - Find real-world usage examples and benchmarks

3. Devil's advocate:
   - Challenge each option's weaknesses
   - Identify migration costs and risks
   - Question if we even need a new library

Have them present findings and debate. I'll make the final decision.
```

**아키텍처 결정:**
```
Should we migrate from REST to GraphQL?
Create a research team:

- One teammate investigates our current REST API pain points
- One researches GraphQL benefits and implementation strategies
- One plays devil's advocate (migration cost, learning curve, complexity)

Each teammate should provide concrete evidence, not just opinions.
```

### 사용 시기

- 중요한 기술 스택 변경 결정
- 새로운 도구/라이브러리 도입 검토
- 아키텍처 개선 방향 탐색

### 주의사항

- 리서치 범위를 명확히 제한 (무한 리서치 방지)
- 구체적인 기준 제시 (성능, 유지보수성, 팀 학습 곡선 등)
- Lead가 최종 결정 권한을 가짐 (팀은 정보 제공 역할)

---

## Delegate 모드

### 개요

Lead focuses on coordination only. Cannot touch code directly.

**언제 사용:**
- Lead가 직접 구현하기 시작할 때 (조정 역할 벗어남)
- 순수 조정/합성 역할만 필요할 때
- Teammate들의 결과를 종합하는 작업만 남았을 때

**활성화 방법:**
팀 시작 후 `Shift+Tab` 키를 누름

### Delegate 모드의 효과

| 모드 | Lead 가능 동작 | Lead 불가능 동작 |
|------|---------------|-----------------|
| **일반 모드** | 코드 읽기/쓰기, teammate 조정 | - |
| **Delegate 모드** | Teammate 조정, 결과 종합, 방향 제시 | 코드 직접 편집, 파일 읽기/쓰기 |

### 사용 예제

```
# 팀 시작
Create a team to implement the shopping cart feature.
Spawn teammates for backend, frontend, and testing.

# [Shift+Tab] 눌러서 Delegate 모드 활성화

# Lead는 이제 조정만 수행
- "Backend teammate, API 스펙부터 정의해주세요"
- "Frontend teammate, Backend의 API 스펙 완료될 때까지 대기"
- "Test teammate, 두 teammate의 작업 완료 후 통합 테스트 작성"
```

### 주의사항

- Delegate 모드에서는 Lead가 파일을 읽을 수 없으므로 teammate에게 요약을 요청해야 함
- 필요 시 `Shift+Tab`으로 다시 일반 모드로 전환 가능
- 조정 역할에 집중할 때만 사용 (초기 탐색 단계에서는 비효율적)

---

## Plan Approval 워크플로우

### 개요

Teammate가 계획을 먼저 작성하고 Lead의 승인을 받은 후 실행

**해결하는 문제:**
- Teammate가 잘못된 방향으로 구현 시작
- 대규모 변경이 요구사항과 맞지 않는 경우
- 중요한 파일을 수정하기 전 검토 필요

### 워크플로우

```
1. Teammate works in read-only plan mode
   ↓
2. Finishes planning → sends plan to Lead
   ↓
3. Lead reviews plan → approves or rejects with feedback
   ↓ (if rejected)
4. Teammate revises plan → resubmits to Lead
   ↓ (if approved)
5. Teammate exits plan mode → begins implementation
```

### Lead의 판단 기준 제어

**테스트 커버리지 필수:**
```
Spawn a teammate to refactor the payment module.
Require plan approval.

Only approve plans that include:
- Unit tests for all public methods
- Integration tests for payment gateway interaction
- Test coverage > 80%

Reject plans that skip testing.
```

**데이터베이스 스키마 변경 제한:**
```
Spawn a teammate to optimize the user query.
Require plan approval.

Only approve if the plan:
- Does NOT modify database schema
- Only adds indexes or optimizes queries
- Includes before/after performance benchmarks

Reject plans that alter table structure.
```

**마이그레이션 전략 필수:**
```
Spawn a teammate to rename the User model to Account.
Require plan approval.

Approve only if the plan includes:
- Backward compatibility strategy
- Step-by-step migration plan
- Rollback procedure if issues occur
```

### 프롬프트 예제

```
Create a teammate to implement OAuth2 login.
Enable plan approval mode.

The teammate should submit a plan covering:
- Which files will be created/modified
- Security considerations (token storage, CSRF protection)
- Test strategy (mock OAuth provider, edge cases)
- Error handling approach

I will review and approve/reject the plan before implementation starts.
```

### 사용 시기

- 중요한 코드 변경 (인증, 결제, 데이터 마이그레이션)
- 여러 파일에 영향을 미치는 리팩토링
- 팀 전체의 코딩 규칙 준수 확인 필요할 때

### 주의사항

- 모든 작업에 plan approval을 요구하면 오버헤드 증가
- 간단한 버그 수정이나 단일 파일 변경은 plan approval 불필요
- Teammate가 계획 작성에 너무 오래 걸린다면 범위를 좁혀주기

---

## 패턴 조합 전략

### 예제 1: 리뷰 + 경쟁 가설

```
PR #456 has a performance regression but cause is unclear.
Create a debugging team:
- Spawn 3 teammates with different hypotheses (database, frontend rendering, API)
- Each investigates their theory
- After findings, spawn 3 review teammates (security, performance, tests)
- Review the proposed fix from all angles
```

### 예제 2: Cross-layer + Plan Approval

```
Implement real-time chat feature.
Spawn teammates for backend, frontend, tests.
Require plan approval from all three before implementation.

Each plan must include:
- API contract (shared interface)
- WebSocket connection handling strategy
- Test approach for real-time scenarios
```

### 예제 3: 리서치 + Delegate 모드

```
Research caching strategies for our app.
Spawn research teammates (Redis, Memcached, in-memory).
Enable delegate mode - I only coordinate and synthesize findings.

Each teammate presents:
- Performance benchmarks
- Cost analysis
- Integration complexity

I will compare results and decide.
```

---

## 안티패턴 (사용하지 말 것)

### ❌ 파일 충돌 허용

**잘못된 예:**
```
Spawn two teammates to refactor authentication.
Both can edit any file they want.
```

**결과**: `auth.py`를 두 teammate가 동시에 편집 → 변경사항 덮어씀

**올바른 방법**: 파일 소유권 명확히 분리

---

### ❌ 모든 작업에 Plan Approval 강제

**잘못된 예:**
```
Spawn a teammate to fix typo in README.
Require plan approval.
```

**결과**: 간단한 작업에 불필요한 승인 단계 추가

**올바른 방법**: 중요한 변경에만 plan approval 사용

---

### ❌ Teammate에게 Lead 역할 위임

**잘못된 예:**
```
Spawn a teammate to coordinate three other teammates.
```

**결과**: Nested coordination → 복잡도 증가, 책임 불명확

**올바른 방법**: Lead가 직접 모든 teammate 조정 (Delegate 모드 활용)

---

### ❌ 너무 많은 Teammate (5명 초과)

**잘못된 예:**
```
Spawn 10 teammates to review this PR from every possible angle.
```

**결과**: 조정 비용 폭증, 정보 과부하

**올바른 방법**: 3-5명으로 제한, 역할을 명확히 정의

---

## 다음 단계

- **[04-best-practices.md](./04-best-practices.md)**: 실전 팁과 트러블슈팅
- **[05-examples.md](./05-examples.md)**: 실제 사용 사례 모음
