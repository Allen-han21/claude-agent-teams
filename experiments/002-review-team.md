# 실험 002: 코드 리뷰 팀

**실험일**: 2026-02-07
**팀 구성**: Lead 1 + Reviewer 3 (로직/안전성/아키텍처)
**대상**: kidsnote_ios PR #7402 "[PK-33175] 임시저장 복원 시 원아/반 선택 정보 유지"
**소요 시간**: 약 5분 (팀 생성 → 결과 수합)

---

## 실험 설계

### 목표
실제 프로덕션 PR을 Agent Teams로 다관점 코드 리뷰하고,
단일 리뷰 대비 발견 범위와 품질을 비교한다.

### PR 개요
| 항목 | 내용 |
|------|------|
| PR | #7402 |
| 티켓 | PK-33175 |
| 변경 | +109 / -11 (5 files) |
| 핵심 | 임시저장 복원 시 원아/반 선택 정보가 초기화되는 버그 수정 |
| 아키텍처 | ReactorKit (UIKit) |

### 팀 구성
| 역할 | Agent 타입 | 관점 |
|------|-----------|------|
| **Team Lead** | 메인 세션 | PR diff 분석, 팀 조율, 종합 판정 |
| **logic-reviewer** | Explore (read-only) | 로직 정확성, 엣지 케이스, nil 처리 |
| **safety-reviewer** | Explore (read-only) | 크래시 위험, Obj-C 브릿징, 스레드 안전성 |
| **arch-reviewer** | Explore (read-only) | ReactorKit 패턴, 코드 중복, AGENTS.md 컨벤션 |

### 실행 방식
- Lead가 `gh pr diff 7402`로 전체 diff를 확인
- 각 리뷰어에게 diff 핵심 + 검토 포인트 + 파일 목록 전달
- 3명 완전 병렬 실행 (의존성 없음, fan-out/fan-in 패턴)
- 결과 수합 후 Lead가 종합 판정

---

## 종합 리뷰 결과

### 전체 요약

| 관점 | Critical | Warning | Info | 핵심 발견 |
|------|----------|---------|------|----------|
| **로직** | 0 | 4 | 3 | fallback 전략 비대칭, 빈 enrollment 시 UI 갭 |
| **안전성** | 0 | 2 | 5 | Obj-C 브릿징 안전, 크래시 경로 없음 |
| **아키텍처** | 0 | 3 | 5 | reduce() 내 side effect (기존 패턴), 컨벤션 준수 양호 |
| **합계** | **0** | **9** | **13** | |

### 종합 판정: **APPROVE (조건부)**

머지 가능. 아래 2가지 사항은 확인/코멘트 권장.

---

### 교차 발견 사항 (3명 이상에서 공통 지적)

#### 1. Enrollment vs Class fallback 전략 비대칭 (logic + arch 공통)
- `validEnrollmentIDs`: `totalEnrollments` nil → **빈 배열** 반환
- `validClassIDs`: `classInfo` nil → **savedClasses 그대로** 반환
- 도메인적으로 의도된 차이일 수 있으나 주석으로 명시 권장

#### 2. `UserInfo.shared()` 중복 호출 (logic + safety 공통)
- ReportWriteReactor: guard에서 이미 바인딩한 `userInfo`가 있는데 `UserInfo.shared()?`를 다시 호출
- OpenboardWriteViewReactor: 한 줄에서 `UserInfo.shared()`를 두 번 호출
- 기능적 문제는 없으나 지역 변수 사용이 더 안전하고 가독적

---

### 관점별 고유 발견

#### 로직 리뷰어 고유
- **W2**: Album에서 모든 enrollment가 무효할 때 `enrollmentOrigin`이 설정 안 됨 → UI 갭 가능
- **I3**: Notice/Openboard에서 validClasses를 UI만 업데이트하고 `state.writingModel.classesTo`는 원본 유지 → 바로 전송 시 무효 ID 포함 가능

#### 안전성 리뷰어 고유
- **전체 안전 확인**: 모든 `as?` 캐스팅에 fallback 존재, NSNull은 `compactMap`으로 방어, 스레드는 기존 패턴과 동일 수준
- **I2**: `UserInfo.shared()`는 `dispatch_once` 싱글톤으로 실질적 nil 불가 (Obj-C nullability 미지정으로 Optional import)

#### 아키텍처 리뷰어 고유
- **W1**: reduce() 내 side effect (sub-reactor action 호출) — 기존 프로젝트 관행이므로 이 PR의 문제 아님
- **I4**: 4개 Reactor 간 공통 프로토콜 불필요 — 데이터 모델/sub-reactor가 모두 달라 UserInfo Extension 공통화가 적절
- **I2**: 주석 컨벤션(/// [목적], /// [기능]) 잘 준수됨

---

## 실험 관찰 기록

### Subagent 리뷰 vs Agent Teams 리뷰 비교

| 측면 | Subagent 3개 병렬 | Agent Teams 3명 |
|------|------------------|----------------|
| **결과 품질** | 유사할 것으로 추정 | 유사함 |
| **교차 참조** | Lead가 수동으로 교차 대조 | 동일 (이번 실험에서는 리뷰어 간 DM 미사용) |
| **컨텍스트 관리** | 결과가 Main에 합류 → 오염 | 완전 격리 ✅ |
| **토큰 비용** | ~1.5x × 3 ≈ 4.5x | ~4x (3명 팀) |

### 핵심 관찰

1. **이번 리뷰는 Subagent로도 동일한 품질 달성 가능**: 3명의 리뷰어가 서로 DM을 교환하지 않았으므로 fan-out/fan-in 패턴과 동일. Subagent 3개 병렬 호출로 같은 결과를 얻을 수 있음.

2. **Teams가 더 나았을 시나리오**: 만약 logic-reviewer가 발견한 "fallback 비대칭"에 대해 safety-reviewer에게 "Obj-C 브릿징 관점에서 이 비대칭이 안전한가?" 라고 DM을 보내고, 그 답변을 바탕으로 판단을 수정하는 **교차 검증 워크플로우**가 있었다면 Teams만의 고유 가치가 발현되었을 것.

3. **Lead의 역할이 핵심**: PR diff를 분석하고 각 리뷰어에게 적절한 검토 포인트를 배분하는 것이 리뷰 품질을 결정. 모호한 지시("리뷰해줘")보다 구체적 검토 포인트가 훨씬 효과적.

4. **Explore 타입 적합**: 코드 리뷰는 read-only이므로 Explore agent가 최적. 실수로 코드를 수정할 위험 없음.

---

## 발표 반영 포인트

- **실제 프로덕션 코드 리뷰 사례**로 데모 가능
- **3관점 동시 리뷰**의 시각적 효과 (tmux split-pane에서 3개 패널 동시 작업)
- **솔직한 비교**: "이번 리뷰는 Subagent로도 충분했다. Teams의 진짜 가치는 리뷰어 간 교차 검증이 필요할 때."
