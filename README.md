# Claude Code Agent Teams

> **여러 Claude Code 인스턴스가 팀으로 협업하는 실험적 기능을 탐구합니다.**

Claude Code의 Agent Teams는 하나의 Lead가 여러 Teammates를 조율하며, 공유 Task List와 Mailbox로 서로 소통하는 멀티 에이전트 시스템입니다.

## 왜 Agent Teams인가?

기존 Subagents는 **단방향**입니다. 메인 에이전트가 작업을 위임하고, 결과만 돌려받습니다.

Agent Teams는 **양방향**입니다. Teammates끼리 직접 메시지를 주고받고, 서로의 발견을 도전(challenge)하며, Task를 자율적으로 가져갑니다.

| | Subagents | Agent Teams |
|---|---|---|
| 통신 | 결과만 보고 | 직접 메시징 |
| 조정 | 메인이 전부 관리 | 공유 Task List + 자율 |
| 토큰 | 낮음 | 높음 (독립 context) |
| 적합 | 집중된 단일 작업 | 토론/협업 필요한 복잡 작업 |

## 빠른 시작

### 1. 환경변수 활성화

`settings.json`에 추가:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### 2. 팀 생성 요청

Claude Code에서 자연어로 요청:

```
코드 리뷰 팀을 만들어줘:
- 보안 검토 teammate
- 성능 분석 teammate
- 테스트 커버리지 teammate
각자 리뷰하고 결과를 종합해줘.
```

### 3. 팀 관리

- **Shift+Up/Down**: teammate 전환 (in-process 모드)
- **Shift+Tab**: delegate 모드 (Lead가 코드 안 건드리고 조정만)
- **Ctrl+T**: 공유 Task List 토글

## 프로젝트 구조

```
docs/              # 핵심 개념, 설정, 패턴, 모범사례, 제한사항
examples/          # 팀 구성 프롬프트 예제 (리뷰, 디버깅, 개발, 리서치)
configs/           # 설정 예시 (민감 정보 마스킹)
experiments/       # 실험 기록 & 결과
presentation/      # 발표 자료
```

## 문서

- [핵심 개념](docs/01-concept.md) - 아키텍처, Subagents 비교
- [설정 가이드](docs/02-setup.md) - 환경변수, 디스플레이 모드, tmux
- [사용 패턴](docs/03-patterns.md) - 리뷰팀, 디버깅팀, 개발팀, 리서치팀
- [모범 사례](docs/04-best-practices.md) - 팀 크기, 작업 분배, 비용 관리
- [제한사항](docs/05-limitations.md) - 현재 한계 & 워크어라운드

## 비용 참고

Agent Teams는 팀원 수만큼 독립 context window를 사용합니다.
- 3명 팀 = 약 4배 토큰 (Lead + 3 Teammates)
- **환경변수 1줄**로 활성화/비활성화 → 완전 가역적
- 팀을 만들지 않으면 기존과 동일한 비용

## 참고 자료

- [공식 문서: Agent Teams](https://docs.anthropic.com/en/docs/claude-code/agent-teams)
- [공식 문서: Subagents](https://docs.anthropic.com/en/docs/claude-code/sub-agents)

## License

MIT
