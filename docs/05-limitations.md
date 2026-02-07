# 현재 제한사항

> Agent Teams는 **실험적 기능**입니다. 아래 제한사항들은 향후 업데이트로 해결될 수 있습니다.

## 세션 관련

### 세션 재개 불가 (In-process)
`/resume`과 `/rewind`로 in-process teammates를 복원할 수 없습니다.

**워크어라운드**: 세션 재개 후 Lead에게 새 teammates를 spawn하도록 요청하세요.

### 한 세션에 한 팀만
Lead는 동시에 하나의 팀만 관리할 수 있습니다.

**워크어라운드**: 현재 팀을 정리(clean up)한 후 새 팀을 시작하세요.

## 팀 구조

### 중첩 팀 불가
Teammates는 자신의 팀이나 teammates를 생성할 수 없습니다. Lead만 팀을 관리합니다.

### Lead 고정
팀을 생성한 세션이 영구적으로 Lead입니다. Teammate를 Lead로 승격하거나 리더십을 이전할 수 없습니다.

## Task 관리

### Task 상태 지연
Teammates가 Task를 완료했지만 상태를 업데이트하지 않는 경우가 있습니다. 의존하는 Task가 차단될 수 있습니다.

**워크어라운드**: 작업이 실제로 완료되었는지 확인하고, Lead에게 상태를 수동 업데이트하도록 지시하세요.

## Permission

### Spawn 시 개별 권한 설정 불가
모든 teammates는 Lead의 permission 모드로 시작합니다. Spawn 시점에 개별 모드를 설정할 수 없습니다.

**워크어라운드**: spawn 후 개별 teammate의 모드를 변경할 수 있습니다.

### Permission 프롬프트 과다
Teammate의 권한 요청이 Lead에게 전달되어 마찰이 생길 수 있습니다.

**워크어라운드**: permission settings에서 자주 사용하는 read-only 도구를 미리 allow 설정하세요.

## 디스플레이

### Split pane 호환성
Split-pane 모드는 다음 환경에서 지원되지 않습니다:
- VS Code 통합 터미널
- Windows Terminal
- Ghostty

**워크어라운드**: In-process 모드를 사용하세요 (어떤 터미널에서든 동작).

### tmux 제한
tmux는 특정 OS에서 제한이 있으며, macOS에서 가장 안정적입니다.

**워크어라운드**: iTerm2에서 `tmux -CC`를 사용하는 것이 권장됩니다.

## 종료

### 느린 종료
Teammates는 현재 요청이나 도구 호출이 완료된 후 종료됩니다. 시간이 걸릴 수 있습니다.

### 정리는 Lead만
Teammates가 cleanup을 실행하면 팀 context가 올바르게 해결되지 않아 리소스가 불일치 상태가 될 수 있습니다.

**규칙**: 항상 Lead에게 "Clean up the team"을 요청하세요.

### 고아 tmux 세션
팀 종료 후 tmux 세션이 남아있을 수 있습니다.

```bash
# 확인
tmux ls

# 정리
tmux kill-session -t <session-name>
```

## 비용

### 토큰 사용량 스케일링
Teammate 수 × 독립 context window = 비용 급증 가능.

| 구성 | 대략적 비용 배수 |
|------|----------------|
| 단일 세션 | 1x |
| Subagent | ~1.5x |
| 3명 팀 | ~4x |
| 5명 팀 | ~6x |

**핵심**: 리서치/리뷰/새 기능에는 추가 토큰이 보통 가치 있음. 단순 작업에는 단일 세션이 효율적.
