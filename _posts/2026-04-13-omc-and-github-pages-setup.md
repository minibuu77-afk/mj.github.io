---
layout: post
title: "Claude Code에 OMC 플러그인 설치하고 GitHub Pages 블로그 만들기"
date: 2026-04-13 00:00:00 +0900
categories: [개발, Claude, GitHub]
tags: [claude-code, omc, github-pages, jekyll, ai]
---

오늘은 Claude Code에 oh-my-claudecode(OMC) 플러그인을 설치하고, GitHub Pages로 블로그를 처음 셋팅해본 경험을 정리합니다.

## 1. OMC(oh-my-claudecode)란?

OMC는 Claude Code를 강화해주는 플러그인입니다. 설치하면 Claude가 복잡한 작업을 자동으로 병렬화하고, 전문 에이전트에게 위임하는 등 더 스마트하게 동작합니다.

### 주요 특징

- **자동 병렬 실행**: 복잡한 태스크를 자동으로 나눠서 동시 처리
- **28+ 전문 에이전트**: architect, executor, debugger, designer 등 역할별 에이전트
- **모델 라우팅**: 작업 복잡도에 따라 Haiku/Sonnet/Opus를 자동 선택
- **HUD 상태바**: Claude Code 하단에 실시간 상태 표시
- **팀 모드**: 여러 에이전트가 공유 태스크 리스트로 협업

### 매직 키워드

명령어를 외울 필요 없이, 자연어에 키워드만 넣으면 됩니다:

| 키워드 | 효과 |
|--------|------|
| `ralph` | 완료될 때까지 끈질기게 시도 |
| `ralplan` | 반복적 플래닝 인터뷰 |
| `ulw` | 최대 병렬 실행 |
| `plan` | 플래닝 인터뷰 시작 |
| `team` | 여러 에이전트 동시 투입 |

## 2. OMC 설치 방법

Claude Code 안에서 아래 명령어로 플러그인을 설치합니다:

```
/plugin install oh-my-claudecode
```

그 다음 셋업 위자드를 실행합니다:

```
/oh-my-claudecode:omc-setup
```

위자드가 순서대로 안내해줍니다:

1. **CLAUDE.md 설치** - 글로벌 또는 프로젝트별 선택
2. **HUD 상태바 설정** - `~/.claude/hud/omc-hud.mjs` 설치 및 `settings.json` 연동
3. **MCP 서버 설정** - GitHub, Context7, Exa 등 외부 도구 연결
4. **에이전트 팀 활성화** - `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` 설정

### 셋업 후 생성/변경되는 파일

```
~/.claude/
├── CLAUDE.md              # OMC 에이전트 지시사항
├── settings.json          # statusLine, env, mcpServers 추가됨
├── hud/
│   ├── omc-hud.mjs        # HUD 상태바 스크립트
│   └── lib/
│       └── config-dir.mjs
└── .omc-config.json       # 설정값 저장 (실행 모드, 팀 설정 등)
```

### settings.json 주요 변경사항

```json
{
  "statusLine": {
    "type": "command",
    "command": "node ${CLAUDE_CONFIG_DIR:-$HOME/.claude}/hud/omc-hud.mjs"
  },
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  },
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "your_token_here"
      }
    }
  }
}
```

## 3. GitHub Pages + Jekyll 블로그 셋팅

GitHub Pages는 `username.github.io` 이름의 레포를 만들면 자동으로 정적 사이트로 배포해줍니다. Jekyll을 지원해서 마크다운으로 블로그 포스트를 쉽게 작성할 수 있습니다.

### 최소 설정 파일: `_config.yml`

```yaml
title: MJ's Dev Blog
description: 개발하면서 배운 것들을 기록합니다
theme: minima

plugins:
  - jekyll-feed

permalink: /:year/:month/:day/:title/
```

### 포스트 작성 규칙

`_posts/` 폴더에 `YYYY-MM-DD-제목.md` 형식으로 파일을 만들면 자동으로 블로그 포스트가 됩니다:

```markdown
---
layout: post
title: "포스트 제목"
date: 2026-04-13 00:00:00 +0900
categories: [카테고리]
---

본문 내용...
```

### GitHub API로 파일 올리기

`gh` CLI나 GitHub API로 직접 파일을 올릴 수 있습니다:

```bash
# base64로 인코딩 후 API 호출
CONTENT=$(cat post.md | base64)
curl -X PUT \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  "https://api.github.com/repos/username/username.github.io/contents/_posts/2026-04-13-post.md" \
  -d "{\"message\": \"Add post\", \"content\": \"$CONTENT\"}"
```

## 4. 오늘 배운 핵심

- OMC는 설치 후 **별도 명령어 없이** 자동으로 작동한다
- HUD 상태바는 Claude Code **재시작 후** 적용된다
- GitHub Pages는 `_config.yml`과 `_posts/` 폴더만 있으면 바로 Jekyll 블로그가 된다
- GitHub MCP 서버를 연결하면 Claude가 레포에 직접 파일을 올릴 수 있다

---

*이 포스트는 Claude Code + OMC가 자동으로 작성하고 업로드했습니다.*
