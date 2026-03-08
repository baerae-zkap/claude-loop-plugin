# Claude Loop

Rate limit에 걸려도 자동으로 재시도하는 Claude Code 래퍼 스크립트.

## 이게 뭔가요?

`claude-loop`은 **독립적인 쉘 스크립트**입니다. Claude Code 내부에서 실행되는 게 아니라, **터미널에서 Claude Code를 감싸서 실행**합니다.

```
일반적인 사용:        claude "프롬프트"           → rate limit 걸리면 끝
claude-loop 사용:     claude-loop "프롬프트"     → rate limit 걸리면 대기 후 자동 재개
```

## 설치

### 방법 1: Claude Code 스킬로 설치 (권장)

```bash
# Claude Code 안에서
/plugin marketplace add baerae-zkap/claude-loop-plugin
/install-claude-loop
```

스킬이 하는 일:
1. `~/.local/bin/claude-loop` 스크립트 생성
2. PATH에 `~/.local/bin` 추가

### 방법 2: 수동 설치

```bash
# 스크립트 다운로드
mkdir -p ~/.local/bin
curl -o ~/.local/bin/claude-loop https://raw.githubusercontent.com/baerae-zkap/claude-loop-plugin/main/scripts/claude-loop.sh
chmod +x ~/.local/bin/claude-loop

# PATH 추가 (~/.zshrc 또는 ~/.bashrc)
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

## 사용법

**중요: Claude Code 밖에서, 일반 터미널에서 실행합니다.**

```bash
# 터미널에서 실행
claude-loop "implement user authentication"
```

### 동작 방식

```
1. claude-loop이 claude를 실행
2. 작업 진행...
3. Rate limit 감지됨
4. 5분 대기 (카운트다운 표시)
5. 이전 세션 자동 재개
6. 작업 계속...
7. 완료될 때까지 반복
```

### 환경 변수

```bash
# 재시도 간격 변경 (기본 300초 = 5분)
CLAUDE_LOOP_DELAY=180 claude-loop "fix errors"

# 최대 재시도 횟수 (기본 100회)
CLAUDE_LOOP_MAX_RETRIES=50 claude-loop "big task"

# 디버그 모드 (JSON 응답 확인)
CLAUDE_LOOP_DEBUG=1 claude-loop "test"
```

## 플러그인 vs 스크립트

| 구분 | 설명 |
|------|------|
| **플러그인/스킬** | 설치 도우미. Claude Code 안에서 `/install-claude-loop` 실행하면 스크립트를 설치해줌 |
| **claude-loop 스크립트** | 실제 도구. 터미널에서 `claude-loop "프롬프트"`로 사용 |

```
┌─────────────────────────────────────────────────┐
│  Claude Code                                    │
│  └─ /install-claude-loop  → 스크립트 설치      │
└─────────────────────────────────────────────────┘
                    ↓ 설치
┌─────────────────────────────────────────────────┐
│  터미널 (Claude Code 밖)                        │
│  └─ claude-loop "프롬프트" → Claude 실행/재시도 │
└─────────────────────────────────────────────────┘
```

## License

MIT
