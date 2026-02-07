# 설정 가이드

Agent Teams를 사용하기 위한 환경 설정과 최적화 방법을 안내합니다.

---

## 환경변수 활성화

Agent Teams는 실험적 기능으로, 사용하려면 환경변수를 활성화해야 합니다.

### 방법 1: settings.json 설정 (권장)

Claude Code의 설정 파일에 환경변수를 추가합니다.

**위치**: `~/.config/claude/settings.json` (또는 프로젝트별 `.claude/settings.json`)

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

이 방법은 Claude Code 실행 시 자동으로 적용됩니다.

### 방법 2: Shell 환경변수 설정

터미널 세션에서 직접 환경변수를 설정할 수도 있습니다.

```bash
# Bash/Zsh
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1

# 영구 적용 (~/.zshrc 또는 ~/.bashrc)
echo 'export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1' >> ~/.zshrc
source ~/.zshrc
```

---

## 디스플레이 모드

Agent Teams는 두 가지 디스플레이 모드를 지원합니다.

### In-process 모드 (기본)

모든 teammate가 메인 터미널 내부에서 실행됩니다.

**특징**:
- 단일 터미널 창에서 모든 teammate 관리
- `Shift+Up`/`Shift+Down`으로 teammate 선택
- `Enter`로 선택한 teammate 출력 보기
- `Escape`로 실행 중단
- 추가 도구 불필요 (모든 터미널에서 동작)

**장점**: 설정 없이 바로 사용 가능
**단점**: 한 번에 한 teammate 출력만 볼 수 있음

### Split-pane 모드

각 teammate가 독립적인 터미널 패널을 갖습니다.

**특징**:
- 각 teammate가 별도 패널에서 실행
- 모든 teammate의 출력을 동시에 확인
- **요구사항**: tmux 또는 iTerm2 필요

**장점**: 병렬 작업 시각화 용이
**단점**: tmux/iTerm2 설치 및 설정 필요

---

## 모드 설정 방법

### settings.json 설정

```json
{
  "teammateMode": "auto"
}
```

**옵션**:
- `"auto"` (기본): tmux 환경이면 split-pane, 아니면 in-process
- `"in-process"`: 강제로 in-process 모드 사용
- `"tmux"`: 강제로 split-pane 모드 사용 (tmux 필수)

### CLI 플래그 설정

실행 시 일시적으로 모드를 변경할 수 있습니다.

```bash
# In-process 모드 강제
claude --teammate-mode in-process

# Split-pane 모드 강제
claude --teammate-mode tmux
```

---

## tmux 설정

Split-pane 모드를 사용하려면 tmux를 설치해야 합니다.

### macOS 설치

```bash
# Homebrew 사용
brew install tmux
```

### tmux 세션 시작

```bash
# 새 tmux 세션 시작
tmux

# tmux 세션 내에서 Claude Code 실행
claude
```

**단축키**:
- `Ctrl+B` `%`: 세로로 패널 분할
- `Ctrl+B` `"`: 가로로 패널 분할
- `Ctrl+B` `방향키`: 패널 간 이동
- `Ctrl+B` `D`: 세션에서 나가기 (detach)
- `tmux attach`: 다시 세션으로 돌아가기

### tmux 설정 최적화 (선택사항)

`~/.tmux.conf` 파일을 생성하여 사용성을 개선할 수 있습니다.

```bash
# 마우스 모드 활성화
set -g mouse on

# 패널 번호 표시 시간 증가
set -g display-panes-time 2000

# 기본 셸 설정
set-option -g default-shell /bin/zsh

# 색상 지원
set -g default-terminal "screen-256color"
```

설정 적용:
```bash
tmux source-file ~/.tmux.conf
```

---

## iTerm2 설정

iTerm2 사용자는 tmux 대신 iTerm2의 split-pane 기능을 사용할 수 있습니다.

### 설치

```bash
# Homebrew Cask 사용
brew install --cask iterm2
```

### Python API 활성화

1. iTerm2 실행
2. **Settings (⌘,)** 열기
3. **General → Magic** 탭으로 이동
4. **Enable Python API** 체크박스 활성화

### it2 CLI 설치

```bash
# iTerm2 메뉴에서 설치
# iTerm2 → Install Shell Integration
```

이제 iTerm2에서 Claude Code 실행 시 자동으로 split-pane 모드가 작동합니다.

---

## Permission 최적화

Agent Teams는 리드 에이전트의 권한 설정을 상속받습니다.

### 기본 동작

- Teammate는 리드와 동일한 permission 모드 사용
- 리드가 읽기 전용 작업을 사전 승인하면, teammate도 동일하게 적용
- 불필요한 승인 프롬프트 감소

### 위험 모드 사용

```bash
# 리드가 권한 검사 스킵 시 모든 teammate도 동일하게 동작
claude --dangerously-skip-permissions
```

⚠️ **주의**: 이 옵션은 모든 파일 작업을 자동 승인하므로 신뢰할 수 없는 코드에는 사용하지 마세요.

### Teammate별 권한 변경

Teammate 생성 후 개별적으로 권한 모드를 변경할 수 있습니다.

```bash
# Teammate 선택 후
# 권한 설정 변경 UI에서 조정
```

### 권장 설정

**개발 환경**:
```json
{
  "autoApproveRead": true,
  "autoApproveWrite": false
}
```

**CI/CD 환경**:
```bash
claude --dangerously-skip-permissions
```

---

## 비활성화 방법

Agent Teams 기능을 비활성화하려면 환경변수를 제거하거나 `"0"`으로 설정합니다.

### settings.json 수정

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "0"
  }
}
```

또는 `env` 섹션 전체 제거:

```json
{
  // "env": {
  //   "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  // }
}
```

### Shell 환경변수 제거

```bash
# 임시 해제
unset CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS

# 영구 해제 (~/.zshrc에서 해당 라인 삭제)
vi ~/.zshrc
# export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 줄 삭제
source ~/.zshrc
```

---

## 문제 해결

### "Agent Teams not available" 오류

**원인**: 환경변수가 제대로 설정되지 않음

**해결**:
```bash
# 환경변수 확인
echo $CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS

# 출력이 "1"이 아니면 설정 재확인
```

### tmux에서 패널이 생성되지 않음

**원인**: tmux 버전이 오래됨

**해결**:
```bash
# tmux 버전 확인 (3.0 이상 권장)
tmux -V

# 업데이트
brew upgrade tmux
```

### iTerm2 Python API 오류

**원인**: Python API가 활성화되지 않음

**해결**:
1. iTerm2 → Settings → General → Magic
2. "Enable Python API" 체크
3. iTerm2 재시작

### Teammate 출력이 보이지 않음

**원인**: In-process 모드에서 다른 teammate 선택 중

**해결**:
- `Shift+Up`/`Shift+Down`으로 원하는 teammate 선택
- `Enter`로 출력 확인

---

## 다음 단계

설정을 완료했다면:

1. [사용법 가이드](03-usage.md)에서 기본 명령어 학습
2. [kidsnote_ios 사례](04-case-study-kidsnote.md)에서 실전 예제 확인
3. [Best Practices](05-best-practices.md)에서 효율적인 사용 팁 습득
