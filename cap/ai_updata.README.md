# 🤖 ARIA — Adaptive Reasoning Intelligence Assistant

> **완전 로컬 기반 · 프라이버시 중심 · MCP/A2A 표준 프로토콜 적용**
> 인터넷 없이도 동작하고, 나를 진짜로 기억하는 AI 개인 비서

---

## 📌 프로젝트 개요

| 항목 | 내용 |
|------|------|
| 프로젝트명 | ARIA (Adaptive Reasoning Intelligence Assistant) |
| 개발 언어 | Python 3.10+ |
| GUI 프레임워크 | PyQt6 |
| 개발 기간 | 2025.03 ~ 2025.06 (약 4개월) |
| 개발 인원 | 1인 (캡스톤 디자인) |
| 핵심 프로토콜 | MCP (Model Context Protocol) + A2A (Agent-to-Agent Protocol) |

---

## 🎯 제작 목적 및 차별화 전략

### 기존 AI 비서의 한계

| 문제점 | 기존 서비스 | ARIA |
|---|---|---|
| **개인정보 유출** | 모든 대화가 외부 서버 전송 | 100% 로컬 저장, 외부 전송 없음 |
| **인터넷 의존** | 오프라인 사용 불가 | Ollama 로컬 모델로 완전 오프라인 동작 |
| **기억 부재** | 대화 종료 시 기억 초기화 | ChromaDB 벡터 DB로 영구 장기 기억 |
| **수동적 응답** | 물어봐야만 답변 | 패턴 학습 후 능동적으로 제안 |
| **확장 불가** | 기능 추가 불가 폐쇄 구조 | MCP 플러그인 방식으로 자유롭게 확장 |
| **단일 AI** | 하나의 모델이 모든 것 처리 | A2A 멀티 에이전트 협업 구조 |

### 핵심 차별화 포인트

```
① 하이브리드 LLM  →  Ollama(무료/로컬) + Claude API(고성능) 자동 전환
② MCP 표준 프로토콜  →  도구 추가 시 기존 코드 수정 없이 플러그인 방식 확장
③ A2A 멀티 에이전트  →  전문 에이전트들이 병렬 협업으로 복잡한 작업 처리
④ 장기 기억 (RAG)  →  과거 대화, 메모, 문서를 벡터 DB에 저장해 영구 기억
⑤ 능동적 비서  →  사용자 패턴 학습 후 먼저 제안하고 의견을 제시
```

---

## 🏗️ 전체 시스템 블록도

```mermaid
graph TB
    User["👤 사용자"]

    subgraph UI["🖥️ 사용자 인터페이스 (PyQt6)"]
        Dashboard["메인 대시보드"]
        ChatUI["💬 AI 채팅창"]
        VoiceBtn["🎙️ 음성 입출력"]
        ModelToggle["⚙️ LLM 전환 스위치\nOllama ↔ Claude"]
    end

    subgraph LLMLayer["🧠 하이브리드 LLM Provider Layer"]
        LLMManager["LLM Manager\n(자동 라우팅)"]
        Claude["☁️ Claude API\nSonnet 4\n고성능 · 복잡한 작업"]
        Ollama["💻 Ollama 로컬\nGemma3 12B\n무료 · 오프라인 · 빠름"]
    end

    subgraph Orchestrator["🎯 메인 오케스트레이터"]
        Router["Task Router\n의도 파악 및 에이전트 위임"]
        ContextMgr["컨텍스트 매니저\n대화 흐름 유지"]
        ProfileLearner["🔍 패턴 학습기\n사용자 습관 자동 분석"]
    end

    subgraph A2A["🤝 A2A 멀티 에이전트 (Agent-to-Agent Protocol)"]
        ScheduleAgent["📅 스케줄러 Agent\n일정 등록 · 조회 · 알림"]
        DocumentAgent["📄 문서 Agent\n파일 요약 · 분석"]
        ResearchAgent["🔍 리서치 Agent\n웹 검색 · 뉴스 요약"]
        MemoryAgent["🧠 메모리 Agent\n기억 저장 · 검색"]
    end

    subgraph MCP["🔌 MCP 서버 레이어 (Model Context Protocol)"]
        CalMCP["📆 Calendar MCP\nadd / get / delete"]
        FileMCP["📁 File MCP\nread / write / search"]
        SearchMCP["🌐 Search MCP\nweb_search / summarize"]
        MemoMCP["📝 Memo MCP\nsave / query / tag"]
        VectorMCP["🗃️ Vector MCP\nembed / similarity_search"]
        WeatherMCP["🌤️ Weather MCP\ncurrent / forecast"]
    end

    subgraph Data["💾 로컬 데이터 저장소 (완전 로컬)"]
        ChromaDB["ChromaDB\n벡터 DB\n장기기억 · 문서 임베딩"]
        SQLite["SQLite\n일정 · 메모\n구조화 데이터"]
        Profile["사용자 프로필\n패턴 · 선호 · 관심사"]
    end

    User <--> Dashboard
    User <--> VoiceBtn
    ModelToggle --> LLMManager
    ChatUI --> LLMManager

    LLMManager -->|"🌐 온라인 + 복잡"| Claude
    LLMManager -->|"📴 오프라인 or 단순"| Ollama
    Claude --> Router
    Ollama --> Router

    Router --> ScheduleAgent
    Router --> DocumentAgent
    Router --> ResearchAgent
    Router --> MemoryAgent
    Router <--> ContextMgr
    ContextMgr --> ProfileLearner

    ScheduleAgent -->|MCP| CalMCP
    DocumentAgent -->|MCP| FileMCP
    ResearchAgent -->|MCP| SearchMCP
    ResearchAgent -->|MCP| WeatherMCP
    MemoryAgent -->|MCP| MemoMCP
    MemoryAgent -->|MCP| VectorMCP

    CalMCP --> SQLite
    FileMCP --> SQLite
    MemoMCP --> SQLite
    VectorMCP --> ChromaDB
    ProfileLearner --> Profile
```

---

## 🧠 하이브리드 LLM 자동 라우팅 흐름

```mermaid
flowchart TD
    Input(["💬 사용자 입력"]) --> NetCheck{인터넷\n연결 확인}

    NetCheck -->|"❌ 오프라인"| Ollama["💻 Ollama\nGemma3 12B\n로컬 실행"]
    NetCheck -->|"✅ 온라인"| ModeCheck{모드 선택}

    ModeCheck -->|"⚙️ 수동 - Ollama"| Ollama
    ModeCheck -->|"⚙️ 수동 - Claude"| Claude["☁️ Claude API\nSonnet 4"]
    ModeCheck -->|"🤖 자동 라우팅"| Complexity{작업 복잡도\n분류}

    Complexity -->|"단순 질문\n메모 저장\n날씨 조회"| Ollama
    Complexity -->|"문서 분석\n복잡한 추론\n다중 에이전트"| Claude

    Ollama --> Response["📤 응답 생성"]
    Claude --> Response
    Response --> Memory["💾 ChromaDB\n대화 기억 저장"]
    Memory --> Profile["📊 사용자 패턴 업데이트"]
    Profile --> End(["✅ 사용자 출력"])
```

---

## 🤝 A2A 에이전트 협업 흐름

```mermaid
sequenceDiagram
    actor User as 👤 사용자
    participant Orch as 🎯 오케스트레이터
    participant Mem as 🧠 메모리 Agent
    participant Sched as 📅 스케줄러 Agent
    participant Research as 🔍 리서치 Agent
    participant Doc as 📄 문서 Agent

    User->>Orch: "내일 캡스톤 발표 준비해줘.\n관련 자료도 찾아줘"

    Orch->>Mem: 과거 캡스톤 관련 기억 조회 요청
    Mem-->>Orch: 이전 대화 컨텍스트 반환

    Note over Orch: 병렬 작업 위임 (A2A)

    par 동시 실행
        Orch->>Sched: 내일 발표 일정 등록 요청
        Orch->>Research: 캡스톤 발표 관련 자료 검색 요청
        Orch->>Doc: 기존 저장 문서 요약 요청
    end

    Sched-->>Orch: "내일 14:00 발표 일정 등록 완료"
    Research-->>Orch: "관련 자료 5건 수집 완료"
    Doc-->>Orch: "기존 문서 요약 완료"

    Orch->>Mem: 이번 대화 내용 저장 요청
    Mem-->>Orch: 저장 완료

    Orch->>User: 통합 결과 응답\n+ 능동적 제안 추가
```

---

## 🔌 MCP 서버 구조

```mermaid
graph LR
    subgraph Agents["에이전트 레이어"]
        A1["📅 스케줄러"]
        A2["📄 문서"]
        A3["🔍 리서치"]
        A4["🧠 메모리"]
    end

    subgraph MCP["MCP 서버 (표준 인터페이스)"]
        S1["📆 Calendar MCP\nadd_schedule()\nget_schedule()\ndelete_schedule()"]
        S2["📁 File MCP\nread_file()\nwrite_file()\nlist_files()"]
        S3["🌐 Search MCP\nweb_search()\nget_news()\nget_weather()"]
        S4["📝 Memo MCP\nsave_memo()\nsearch_memo()\ntag_memo()"]
        S5["🗃️ Vector MCP\nembed_text()\nsimilarity_search()\nstore_memory()"]
    end

    subgraph External["외부 서비스 · 로컬 저장소"]
        E1["Google Calendar API\n또는 로컬 SQLite"]
        E2["로컬 파일시스템"]
        E3["DuckDuckGo\nOpenWeather API\nRSS 피드"]
        E4["SQLite DB"]
        E5["ChromaDB"]
    end

    A1 -->|MCP 프로토콜| S1
    A2 -->|MCP 프로토콜| S2
    A3 -->|MCP 프로토콜| S3
    A4 -->|MCP 프로토콜| S4
    A4 -->|MCP 프로토콜| S5

    S1 --> E1
    S2 --> E2
    S3 --> E3
    S4 --> E4
    S5 --> E5
```

> **MCP의 핵심 장점**: 새 기능 추가 시 기존 코드 수정 없이 MCP 서버만 추가하면 됨
> → 향후 Slack MCP, 이메일 MCP, GitHub MCP 등 무제한 확장 가능

---

## 🧠 장기 기억 (RAG) 구조

```mermaid
flowchart LR
    subgraph Input["입력 데이터"]
        Conv["💬 대화 내용"]
        Memo["📝 메모"]
        File["📄 문서 파일\nPDF · DOCX · TXT"]
    end

    subgraph Process["처리"]
        Embed["🔢 임베딩 변환\n(nomic-embed-text)"]
        Chunk["✂️ 청크 분할"]
    end

    subgraph Store["저장"]
        VectorDB["🗃️ ChromaDB\n벡터 DB"]
    end

    subgraph Retrieve["검색 및 활용"]
        Query["🔍 유사도 검색"]
        Context["📋 컨텍스트 주입"]
        LLM["🧠 LLM 응답 생성"]
    end

    Conv --> Embed
    Memo --> Chunk
    File --> Chunk
    Chunk --> Embed
    Embed --> VectorDB
    VectorDB --> Query
    Query --> Context
    Context --> LLM
```

> **"저번에 말한 캡스톤 주제가 뭐였지?"** → 몇 주 전 대화도 정확히 기억해서 답변

---

## 📅 개발 로드맵

```mermaid
gantt
    title ARIA 개발 로드맵 (16주)
    dateFormat  YYYY-MM-DD

    section Phase 1 - 기반 구축
    Ollama + Python 환경 세팅       :p1a, 2025-03-03, 5d
    Claude API + LLM Manager 구현   :p1b, after p1a, 7d
    기본 채팅 UI (PyQt6)            :p1c, after p1b, 7d

    section Phase 2 - MCP 서버 구축
    Calendar MCP 서버 개발          :p2a, 2025-03-24, 7d
    File & Memo MCP 서버 개발       :p2b, after p2a, 7d
    Weather & Search MCP 개발       :p2c, after p2b, 7d
    Vector MCP + ChromaDB 연동      :p2d, after p2c, 7d

    section Phase 3 - A2A 에이전트
    메인 오케스트레이터 구현         :p3a, 2025-04-21, 7d
    스케줄러 & 문서 Agent 구현      :p3b, after p3a, 7d
    리서치 & 메모리 Agent 구현      :p3c, after p3b, 7d

    section Phase 4 - 고도화
    음성 입출력 (Whisper + TTS)     :p4a, 2025-05-12, 7d
    사용자 패턴 학습 & 능동 제안    :p4b, after p4a, 7d
    UI 완성 & 통합 테스트           :p4c, after p4b, 7d

    section Phase 5 - 마무리
    버그 수정 & 최적화              :p5a, 2025-06-02, 5d
    발표 자료 & 시연 준비           :p5b, after p5a, 9d
```

---

## 🛠️ 기술 스택

| 분류 | 기술 | 선택 이유 |
|---|---|---|
| **LLM (로컬)** | Ollama + Gemma3 12B | 오프라인 동작, 빠른 응답, 자연스러운 말투 |
| **LLM (클라우드)** | Claude API (Sonnet 4) | MCP 네이티브 지원, 최고 수준 한국어 |
| **MCP** | Python MCP SDK | Anthropic 공식 표준 프로토콜 |
| **A2A** | Google A2A SDK | 멀티 에이전트 표준 프로토콜 |
| **GUI** | PyQt6 | 데스크탑 앱 배포, 풍부한 위젯 |
| **음성 인식** | OpenAI Whisper (로컬) | 한국어 인식률 최상, 오프라인 동작 |
| **음성 합성** | pyttsx3 / Coqui TTS | 완전 로컬, 빠른 응답 |
| **벡터 DB** | ChromaDB | 완전 로컬, 설치 간단, 장기 기억 |
| **관계형 DB** | SQLite | 경량 로컬 DB, 일정·메모 저장 |
| **스케줄러** | APScheduler | 알림, 백그라운드 작업 |
| **임베딩** | nomic-embed-text | Ollama 지원, 로컬 실행 |
| **웹 검색** | DuckDuckGo Search | 무료, API 키 불필요 |
| **날씨** | OpenWeatherMap API | 무료 플랜 제공 |
| **뉴스** | RSS Feed + AI 요약 | 실시간 뉴스, 비용 없음 |

---

## 📁 프로젝트 폴더 구조

```
ARIA/
├── main.py                      # 앱 진입점
├── config.py                    # API 키 & 설정 (env 기반)
├── requirements.txt
│
├── core/                        # 핵심 엔진
│   ├── llm_manager.py           # 하이브리드 LLM 관리 & 자동 라우팅
│   ├── orchestrator.py          # 메인 오케스트레이터 (A2A 위임)
│   ├── context_manager.py       # 대화 컨텍스트 유지
│   └── profile_learner.py       # 사용자 패턴 학습
│
├── agents/                      # A2A 에이전트들
│   ├── base_agent.py            # 공통 베이스 클래스
│   ├── scheduler_agent.py       # 일정 관리 에이전트
│   ├── document_agent.py        # 문서 처리 에이전트
│   ├── research_agent.py        # 리서치 에이전트
│   └── memory_agent.py          # 기억 관리 에이전트
│
├── mcp_servers/                 # MCP 서버들
│   ├── calendar_mcp.py          # 캘린더 MCP
│   ├── file_mcp.py              # 파일시스템 MCP
│   ├── search_mcp.py            # 웹 검색 MCP
│   ├── memo_mcp.py              # 메모 MCP
│   ├── vector_mcp.py            # 벡터 DB MCP
│   └── weather_mcp.py           # 날씨 MCP
│
├── memory/
│   ├── vector_store.py          # ChromaDB 관리
│   ├── embedder.py              # 텍스트 임베딩
│   └── retriever.py             # 유사도 검색
│
├── voice/
│   ├── stt.py                   # 음성 인식 (Whisper 로컬)
│   └── tts.py                   # 음성 합성
│
├── ui/                          # PyQt6 UI
│   ├── main_window.py           # 메인 대시보드
│   ├── chat_widget.py           # AI 채팅창
│   ├── calendar_widget.py       # 캘린더 뷰
│   ├── weather_widget.py        # 날씨 위젯
│   ├── news_widget.py           # 뉴스 위젯
│   └── model_toggle.py          # LLM 전환 스위치
│
├── services/                    # 외부 서비스 연동
│   ├── weather.py               # OpenWeather API
│   ├── news.py                  # RSS 뉴스 수집
│   └── web_search.py            # DuckDuckGo 검색
│
├── data/                        # 로컬 데이터 (gitignore)
│   ├── chroma_db/               # 벡터 DB
│   ├── aria.db                  # SQLite DB
│   └── user_profile.json        # 사용자 패턴 프로필
│
└── tests/
    ├── test_llm_manager.py
    ├── test_agents.py
    └── test_mcp_servers.py
```

---

## ✨ 핵심 기능 명세

### 💬 자연어 대화
- 한국어/영어 혼용 자연스러운 대화
- AI스럽지 않은 친근한 말투 (페르소나 설계)
- 이전 대화 맥락 유지 및 연속 대화

### 📅 일정 관리
- "내일 오후 3시 팀미팅 등록해줘" → 자동 캘린더 등록
- 오늘/이번 주 일정 조회
- 일정 30분 전 자동 알림 팝업

### 🧠 장기 기억
- 과거 대화, 메모, 문서를 ChromaDB에 영구 저장
- "저번에 말한 캡스톤 주제가 뭐였지?" → 정확히 기억

### 📄 문서 요약
- PDF, DOCX, TXT 업로드 → 핵심 요약
- 논문 분석: 목적/방법/결론 구조화 추출

### 🌤️ 날씨 & 📰 뉴스
- 실시간 날씨 조회 및 예보
- RSS 뉴스 수집 후 AI가 핵심만 요약

### 💡 능동적 제안
- 사용자 패턴 학습 후 먼저 제안
- "내일 발표 있는데, 오늘 준비 시작할까요?"

### 🎙️ 음성 인터페이스
- 한국어 음성 명령 인식 (Whisper 로컬)
- 자연스러운 한국어 음성 응답 (TTS)

---

## 🔮 향후 확장 계획

- [ ] Slack MCP 서버 추가 (메시지 전송/조회)
- [ ] 이메일 MCP 서버 추가
- [ ] GitHub MCP 서버 추가 (이슈/PR 관리)
- [ ] 모바일 앱 연동 (원격 알림)
- [ ] 사용자별 파인튜닝 모델 적용
- [ ] 다중 사용자 지원

---

## 🖥️ 개발 환경

```
OS        : Windows 11
CPU       : AMD Ryzen 9 7945HX (16 Core / 32 Thread)
RAM       : 32GB DDR5
GPU       : NVIDIA RTX 4070 Laptop (8GB VRAM)
SSD       : Samsung 980 Pro 2TB (NVMe)
Python    : 3.11+
CUDA      : 12.x
```

---

## 🎓 캡스톤 디자인 어필 포인트

> *"단순한 Claude API 래퍼가 아닙니다.*
> *MCP · A2A 최신 표준 프로토콜을 적용한 확장 가능한 구조,*
> *오프라인에서도 동작하는 하이브리드 LLM,*
> *ChromaDB 기반 장기 기억으로 진짜 나를 아는 비서를 구현했습니다."*

---

*본 프로젝트는 캡스톤 디자인 수업 1인 프로젝트로 개발되었습니다.*
