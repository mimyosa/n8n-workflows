# n8n Workflows

개인적으로 구축한 n8n 자동화 워크플로우 모음입니다.
n8n을 Docker로 self-host하여 운영 중입니다.

## Workflows

| 워크플로우 | 설명 |
|-----------|------|
| [AI News Daily Briefing](./workflows/ai-news-daily-briefing.json) | 매일 아침 AI 뉴스를 자동 수집·요약하여 Telegram/Slack으로 전송 |

---

## AI News Daily Briefing

### 개요

RSS 피드에서 AI 관련 뉴스를 수집하고 Google Gemini로 한국어 요약 후 Telegram과 Slack으로 전송하는 워크플로우입니다.
소프트웨어 개발자/개발사 관점에서 유용한 뉴스(API 업데이트, SDK, LLM 트렌드 등)를 우선 선별합니다.

### 워크플로우 구조

```
Schedule Trigger (매일 8시 KST)
      ↓
RSS Read1 (TechCrunch) ──┐
RSS Read2 (The Verge)  ──┤── Merge ── Code (중복제거/정렬) ── Basic LLM Chain ──→ Telegram
RSS Read3 (VentureBeat)──┤                                          ↑           ↘→ Slack
RSS Read4 (Google News)──┘                             Google Gemini Chat Model
```

### 주요 기능

- RSS 4개 소스에서 뉴스 수집 및 중복 제거
- 최신순 정렬 후 상위 5개 선별
- Google Gemini 2.5 Flash로 한국어 요약
- Telegram Bot / Slack 동시 전송
- 각 RSS 노드 오류 발생 시 나머지 소스로 계속 진행 (On Error: Continue)
- Basic LLM Chain 실패 시 자동 재시도 (Retry On Fail)

### 뉴스 소스

| 소스 | URL |
|------|-----|
| TechCrunch AI | https://techcrunch.com/tag/artificial-intelligence/feed/ |
| The Verge AI | https://www.theverge.com/rss/ai-artificial-intelligence/index.xml |
| VentureBeat AI | https://venturebeat.com/category/ai/feed/ |
| Google News AI | https://news.google.com/rss/search?q=AI+API+OR+LLM+OR+AI+SDK+OR+AI+developer+OR+Claude+API+OR+OpenAI+API+OR+Gemini+API&hl=ko&gl=KR&ceid=KR:ko |

---

## 설치 및 실행 방법

### 요구사항

- Windows 10/11
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)

### 1. n8n 실행

PowerShell에서 아래 명령어 실행:

```powershell
docker run -d --name n8n -p 5678:5678 --restart always `
  -v n8n_data:/home/node/.n8n `
  -e GENERIC_TIMEZONE="Asia/Seoul" `
  -e TZ="Asia/Seoul" `
  docker.n8n.io/n8nio/n8n
```

브라우저에서 http://localhost:5678 접속 후 계정 생성

### 2. Credential 등록

n8n 좌측 메뉴 **Credentials** 에서 아래 3개 등록:

| Credential | 발급처 |
|-----------|--------|
| Telegram API | [@BotFather](https://t.me/BotFather) 에서 Bot Token 발급 |
| Google Gemini API | [Google AI Studio](https://aistudio.google.com) 에서 API Key 발급 |
| Slack API | [Slack API](https://api.slack.com/apps) 에서 Bot Token 발급 (`chat:write`, `chat:write.public` 권한 필요) |

### 3. 워크플로우 Import

1. n8n 좌측 메뉴 **Workflows** 클릭
2. 우상단 **`...`** → **Import from file** 클릭
3. `workflows/ai-news-daily-briefing.json` 선택

### 4. Placeholder 교체

Import 후 아래 노드에서 값 교체:

| 노드 | 항목 | 교체할 값 |
|------|------|----------|
| `Send a text message` | Chat ID | Telegram Chat ID |
| `Send a message` | Channel ID | Slack 채널 ID |

> **Telegram Chat ID 확인 방법**
> 봇에게 `/start` 메시지 전송 후 아래 URL 접속:
> `https://api.telegram.org/bot{YOUR_BOT_TOKEN}/getUpdates`
> 응답 JSON에서 `result[0].message.chat.id` 값 사용

### 5. 워크플로우 활성화

우상단 토글을 **Active** (초록색) 로 변경

---

## n8n 관리 명령어

```powershell
docker start n8n    # 시작
docker stop n8n     # 중지
docker logs n8n     # 로그 확인
```

---

## Tech Stack

![n8n](https://img.shields.io/badge/n8n-self--hosted-orange)
![Docker](https://img.shields.io/badge/Docker-Desktop-blue)
![Gemini](https://img.shields.io/badge/Google-Gemini%202.5%20Flash-green)
![Telegram](https://img.shields.io/badge/Telegram-Bot-blue)
![Slack](https://img.shields.io/badge/Slack-Bot-purple)