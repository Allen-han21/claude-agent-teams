# 실험 001: 리서치 팀

**실험일**: 2026-02-07
**팀 구성**: Lead 1 + Researcher 2 + Devil's Advocate 1 (총 4 세션)
**주제**: Agent Teams vs Subagents 실제 차이점
**소요 시간**: 약 5분 (팀 생성 → 결과 수합)

---

## 실험 설계

### 목표
Agent Teams를 사용하여 "Agent Teams vs Subagents" 주제를 다각도로 분석하고,
Teams 자체의 동작 특성을 관찰한다.

### 팀 구성
| 역할 | Agent 타입 | 할당 작업 |
|------|-----------|-----------|
| **Team Lead** | 메인 세션 | 팀 조율, 결과 수합, 보고서 작성 |
| **researcher-1** | Explore (read-only) | Agent Teams 아키텍처 & 핵심 기능 분석 |
| **researcher-2** | Explore (read-only) | Subagents 아키텍처 & 비교 분석 |
| **devils-advocate** | Explore (read-only) | 리서처 결과 비판적 검토 |

### 의존성 구조
```
researcher-1 ──┐
               ├──→ devils-advocate
researcher-2 ──┘
```

---

## 관찰된 동작 특성

### 1. 팀 생성/소환 과정
- `TeamCreate` → 팀 파일(`~/.claude/teams/research-team/config.json`) + 태스크 디렉토리 생성
- `Task` tool로 teammate spawn 시 `team_name`, `name` 파라미터 필요
- Explore 타입 agent는 read-only → 리서치에 적합 (실수로 코드 수정 위험 없음)

### 2. 병렬 실행
- researcher-1과 researcher-2를 동시에 spawn → **실제 병렬 실행 확인**
- 두 리서처가 거의 동시에 결과를 반환 (researcher-2가 약간 먼저)
- 단일 세션에서 순차 실행했다면 2배 이상 소요되었을 작업

### 3. 메시징 시스템
- Teammate → Lead: `SendMessage(type: "message")`로 결과 전달
- 자동 배달: 메시지가 Lead의 turn에 자동으로 표시됨
- Idle 알림: teammate 작업 완료 후 자동으로 idle notification 발송

### 4. 의존성 관리
- `TaskUpdate(addBlockedBy)`: devils-advocate를 두 리서처에 의존하도록 설정
- 실제 구현: Lead가 두 결과를 확인한 후 수동으로 devils-advocate spawn
- **관찰**: `blockedBy`는 자동 실행이 아닌 "Lead가 참조하는 메타데이터" 역할

### 5. Shutdown 프로토콜
- `SendMessage(type: "shutdown_request")` → teammate에게 종료 요청
- teammate가 `shutdown_response(approve: true)` 반환 → 프로세스 종료
- 이후 `TeamDelete`로 팀 리소스 정리

---

## 리서치 결과 요약

### Researcher-1: Agent Teams 아키텍처

**4가지 핵심 구성요소:**
1. **Team Lead** - 팀 생성/관리/조율 (고정, 이전 불가)
2. **Teammates** - 독립 context window, 자율적 작업 수행
3. **Task List** - 공유 상태 관리, blocked/blockedBy 의존성
4. **Mailbox** - DM/broadcast/shutdown 등 메시징 프로토콜

**검증된 4가지 팀 패턴:**
- 코드 리뷰 팀 (보안/성능/테스트 3관점)
- 경쟁 가설 디버깅 (devil's advocate 포함)
- Cross-layer 기능 개발 (파일 소유권 분리 필수)
- 리서치/기술 조사 (현황 + 대안 + 반론)

**Permission 모드 6종:**
default, delegate, plan, acceptEdits, bypassPermissions, dontAsk

### Researcher-2: Subagents vs Teams 비교

| 측면 | Subagents | Agent Teams |
|------|-----------|-------------|
| **통신** | 단방향 (결과만 반환) | 양방향 (DM, broadcast) |
| **상태 관리** | 없음 (Main이 수동 추적) | 공유 Task List |
| **병렬성** | 제한적 (Main이 호출) | 독립 프로세스 |
| **컨텍스트 격리** | 결과가 Main에 합류 | 완전 격리 |
| **비용** | ~1.5x | ~4x (3명) ~ 6x (5명) |
| **자율성** | Main이 명시적 호출 | Task List에서 자율 선택 |

**선택 기준:**
- 결과만 필요 + 순차적 → Subagent
- 협업/논의 필요 + 병렬 가능 + ROI > 비용 → Agent Teams

### Devil's Advocate: 비판적 검토

**핵심 지적사항:**

1. **"진정한 병렬"은 과장** - API rate limit에 의해 실질적으로 "concurrent but throttled"
2. **비용 배수는 추정치** - 1.5x, 4x, 6x 모두 실측 데이터 없음. 메시징 오버헤드 미포함
3. **Subagent 병렬 호출로 80-90% 대체 가능** - Teams만의 고유 이점은 "teammate 간 실시간 대화/반론"
4. **일상 작업의 80-90%는 Teams 불필요** - 단일 세션 + Subagent로 충분
5. **제한사항 반영 부족** - 12개 중 3개만 완전 반영. Task 상태 지연, Lead 고정 등 누락
6. **숨겨진 비용** - spawn 비용, idle warm-up, 조율 실패 재작업, 고아 세션 정리

**예상 청중 반론 7개:**
- Subagent 병렬이면 충분하지 않나?
- ROI 증명 가능한가?
- 실험적 기능을 프로덕션에?
- 파일 충돌은? (last-write-wins, 자동 해결 안됨)
- 한 세션에 한 팀만?
- 다른 AI 도구와 차별점?
- Context window 한계로 teammate가 길을 잃으면?

---

## 실험에서 얻은 인사이트

### Agent Teams의 실제 가치 (이 실험에서 확인됨)

1. **컨텍스트 격리의 실질적 이점**: 3명이 각각 독립적으로 문서를 분석하면서 Main 세션의 context를 오염시키지 않음
2. **다관점 분석의 자연스러운 구현**: researcher + devil's advocate 패턴이 Subagent보다 자연스러움
3. **병렬 시간 단축**: 두 리서처가 동시에 작업 → 순차 대비 약 50% 시간 단축

### Agent Teams의 현실적 한계 (이 실험에서 확인됨)

1. **Lead의 수동 조율 필요**: blockedBy 설정에도 Lead가 직접 판단하여 다음 agent를 spawn
2. **Spawn 비용 체감**: 각 teammate가 프로젝트 설정을 독립 로딩 → 초기 overhead 존재
3. **Subagent 대비 명확한 우위 제한적**: 이 리서치 작업은 Subagent 3개 병렬 호출로도 유사한 결과 가능. 다만 devil's advocate가 다른 researcher의 결과를 "읽고 반박하는" 과정은 Teams가 더 자연스러웠음

### 발표에 반영할 핵심 메시지

> **"Agent Teams는 만능이 아니다. 일상 작업의 80-90%는 Subagent로 충분하다.
> 하지만 teammate 간 실시간 협업이 필요한 10-20%의 복잡한 시나리오에서
> Teams는 Subagent로는 불가능한 가치를 제공한다."**

**발표 전략:**
1. Subagent가 충분한 경우를 먼저 인정 → 신뢰도 확보
2. Teams만의 고유 가치를 정확히 제시 → 차별점 부각
3. 비용/제한사항을 솔직히 공유 → 현실적 도입 가이드
4. 라이브 데모는 리서치/리뷰 패턴 → 충돌 없이 안전하게 시연

---

## 부록: 팀 설정 참고

```json
{
  "team_name": "research-team",
  "members": [
    {"name": "researcher-1", "type": "Explore"},
    {"name": "researcher-2", "type": "Explore"},
    {"name": "devils-advocate", "type": "Explore"}
  ],
  "task_dependencies": {
    "devils-advocate": ["researcher-1", "researcher-2"]
  }
}
```
