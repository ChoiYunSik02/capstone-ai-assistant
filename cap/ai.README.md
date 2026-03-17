# 🤖 AI 개인 비서 데스크탑 앱

> 음성으로 말하면 AI가 스케줄 관리, 날씨 안내, 뉴스 요약, 알림까지 대신 처리해주는 올인원 데스크탑 비서

---

## 📌 프로젝트 개요

| 항목 | 내용 |
|------|------|
| 프로젝트명 | AI 개인 비서 데스크탑 앱 |
| 개발 언어 | Python 3.10+ |
| GUI 프레임워크 | PyQt6 |
| 개발 기간 | 2025.03 ~ 2025.06 (약 4개월) |
| 개발 인원 | 1인 |

---

## 🎯 제작 목적

스케줄 확인, 날씨 검색, 뉴스 읽기 등 매일 반복되는 작업을 **AI가 대신 처리**해주는 개인 비서 앱. 사용자는 말 한마디만 하면 되고, AI가 알아서 필요한 정보를 찾아 답변하거나 일정을 등록해줍니다.

---

## 🏗️ 전체 시스템 블록도

```mermaid
flowchart TD
    User(["👤 사용자"])

    subgraph INPUT["📥 입력 모듈"]
        MIC["🎙️ 음성 입력\n(Whisper API)"]
        TEXT["⌨️ 텍스트 입력"]
    end

    subgraph CORE["🧠 AI 코어"]
        STT["음성 → 텍스트 변환\n(STT)"]
        AI["Claude AI\n명령 분석 & 응답"]
        INTENT["의도 파악\n(스케줄 / 날씨 / 뉴스 / 대화)"]
    end

    subgraph SERVICES["⚙️ 서비스 모듈"]
        SCHEDULE["📅 스케줄 관리\n등록 / 수정 / 알림"]
        WEATHER["🌤️ 날씨 안내\nOpenWeather API"]
        NEWS["📰 뉴스 요약\nRSS + AI 요약"]
        CLOCK["🕐 시계 & D-day\n카운터"]
    end

    subgraph OUTPUT["📤 출력 모듈"]
        GUI["🖥️ PyQt6 GUI\n대시보드"]
        TTS["🔊 음성 출력\n(TTS)"]
        NOTIFY["🔔 알림 팝업"]
    end

    User -->|말하기| MIC
    User -->|타이핑| TEXT
    MIC --> STT
    STT --> AI
    TEXT --> AI
    AI --> INTENT
    INTENT -->|일정 요청| SCHEDULE
    INTENT -->|날씨 요청| WEATHER
    INTENT -->|뉴스 요청| NEWS
    INTENT -->|시간 요청| CLOCK
    SCHEDULE --> GUI
    WEATHER --> GUI
    NEWS --> GUI
    CLOCK --> GUI
    GUI --> TTS
    SCHEDULE -->|시간 도달| NOTIFY
```

---

## 🔄 음성 명령 처리 흐름

```mermaid
sequenceDiagram
    actor 사용자
    participant 음성인식 as 🎙️ 음성인식<br>(Whisper)
    participant AI as 🧠 Claude AI
    participant 서비스 as ⚙️ 서비스 모듈
    participant GUI as 🖥️ GUI

    사용자 ->> 음성인식 : "내일 오후 3시에 팀미팅 등록해줘"
    음성인식 ->> AI : 텍스트 변환 후 전달
    AI ->> AI : 의도 파악 (스케줄 등록)
    AI ->> 서비스 : 날짜/시간/내용 파싱
    서비스 ->> 서비스 : DB에 일정 저장
    서비스 ->> GUI : 캘린더 업데이트
    GUI ->> 사용자 : "내일 오후 3시 팀미팅 등록 완료!" 출력 + 음성 응답
```

---

## 🧩 모듈 구성도

```mermaid
flowchart LR
    subgraph UI["🖥️ UI Layer (PyQt6)"]
        DASHBOARD["메인 대시보드"]
        CHAT["AI 채팅창"]
        CAL["캘린더 뷰"]
        WIDGET["날씨 / 뉴스 위젯"]
    end

    subgraph LOGIC["⚙️ Logic Layer"]
        VOICE["음성 처리\n(Whisper)"]
        AICORE["AI 엔진\n(Claude API)"]
        SCHED["스케줄러\n(APScheduler)"]
    end

    subgraph DATA["💾 Data Layer"]
        DB["SQLite DB\n(일정 저장)"]
        CACHE["캐시\n(날씨/뉴스)"]
        CONFIG["설정 파일\n(config.json)"]
    end

    subgraph EXTERNAL["🌐 외부 API"]
        CLAUDEAPI["Claude API"]
        WEATHERAPI["OpenWeather API"]
        RSSAPI["뉴스 RSS"]
        WHISPERAPI["Whisper API"]
    end

    DASHBOARD <--> AICORE
    CHAT <--> AICORE
    CAL <--> SCHED
    WIDGET <--> CACHE

    VOICE --> AICORE
    AICORE <--> CLAUDEAPI
    AICORE --> SCHED
    SCHED <--> DB
    AICORE --> CACHE
    CACHE <--> WEATHERAPI
    CACHE <--> RSSAPI
    VOICE <--> WHISPERAPI
    AICORE <--> CONFIG
```

---

## 📅 개발 로드맵

```mermaid
gantt
    title 캡스톤 개발 일정
    dateFormat  YYYY-MM-DD
    section 1개월차
    PyQt6 기본 UI 구성         :a1, 2025-03-01, 2w
    시계 & D-day 위젯          :a2, after a1, 2w
    section 2개월차
    Claude AI 연동 & 채팅 기능  :b1, 2025-04-01, 2w
    날씨 API 연동               :b2, after b1, 1w
    뉴스 RSS 요약 기능          :b3, after b2, 1w
    section 3개월차
    음성 인식 (Whisper) 연동    :c1, 2025-05-01, 2w
    스케줄 관리 & 알림 기능     :c2, after c1, 2w
    section 4개월차
    UI 완성도 개선              :d1, 2025-06-01, 1w
    버그 수정 & 테스트          :d2, after d1, 1w
    발표 준비 & 배포            :d3, after d2, 2w
```

---

## 🛠️ 기술 스택

| 분류 | 기술 |
|------|------|
| 언어 | Python 3.10+ |
| GUI | PyQt6 |
| AI 대화 | Claude API |
| 음성 인식 | OpenAI Whisper API |
| 음성 출력 | pyttsx3 (TTS) |
| 날씨 | OpenWeatherMap API |
| 뉴스 | RSS Feed + AI 요약 |
| 스케줄러 | APScheduler |
| DB | SQLite |

---

## 📁 프로젝트 폴더 구조

```
ai-assistant/
├── main.py                  # 앱 진입점
├── config.json              # API 키 & 설정
├── requirements.txt         # 패키지 목록
│
├── ui/                      # PyQt6 UI 파일
│   ├── main_window.py       # 메인 대시보드
│   ├── chat_widget.py       # AI 채팅창
│   ├── calendar_widget.py   # 캘린더
│   └── weather_widget.py    # 날씨 위젯
│
├── core/                    # 핵심 로직
│   ├── ai_engine.py         # Claude API 연동
│   ├── voice.py             # 음성 인식/출력
│   └── scheduler.py        # 스케줄 & 알림
│
├── services/                # 외부 서비스
│   ├── weather.py           # 날씨 API
│   └── news.py              # 뉴스 RSS
│
└── database/
    └── db.py                # SQLite 관리
```

---

## 🔮 향후 확장 아이디어

- [ ] 모바일 앱 연동 (원격 알림)
- [ ] 얼굴 인식으로 자동 로그인
- [ ] 개인 일정 패턴 학습 및 자동 추천
- [ ] 다국어 음성 인식 지원
