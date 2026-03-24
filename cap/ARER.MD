# 🤖 나만의 개인비서 AI - 시스템 아키텍처 & 구현 계획

> 작성자: 2101091 최윤식  
> 작성일: 2026년 03월 24일  
> 목표: Ollama + MCP + A2A 기반 완전 무료 로컬 AI 개인비서

---

## 1. 전체 시스템 개요

```mermaid
graph TB
    User(["👤 사용자"])

    subgraph UI ["🖥️ PyQt6 앱 (프론트엔드)"]
        Chat["💬 채팅 창"]
        Tray["📋 사이드 트레이\n(대화 목록)"]
        Voice["🎙️ 음성 입력\n(Whisper 로컬)"]
    end

    subgraph Brain ["🧠 AI 두뇌 레이어"]
        Router["⚡ Intelligent Router\n(난이도/프라이버시 분류)"]
        Ollama["🦙 Ollama 로컬\n(Qwen2.5 7B / EXAONE)"]
        GroqAPI["☁️ Groq API\n(Llama 3.3 70B, 무료)"]
        GeminiAPI["☁️ Gemini Flash\n(1M 토큰, 무료)"]
    end

    subgraph Memory ["🗃️ 메모리 레이어"]
        MD["📄 MD 파일\n(채팅방별 저장)"]
        ActionLog["📋 Action Log\n(수행 명령 기록)"]
        VectorDB["🔍 ChromaDB\n(벡터 검색)"]
        Profile["👤 사용자 프로필\nuser_profile.md"]
    end

    subgraph MCP ["🔧 MCP 서버 레이어 (도구들)"]
        CalMCP["📅 Calendar MCP\n(Google Calendar)"]
        FileMCP["📁 FileSystem MCP\n(파일 읽기/쓰기)"]
        SearchMCP["🌐 Web Search MCP\n(Brave API, 무료)"]
        GitMCP["🐙 GitHub MCP\n(커밋/PR)"]
        BrowserMCP["🌍 Browser MCP\n(Playwright, 자동화)"]
        NoteMCP["📝 Memo MCP\n(메모장/노트)"]
    end

    subgraph A2A ["🤝 A2A 에이전트 레이어"]
        MainAgent["🎯 메인 에이전트\n(오케스트레이터)"]
        NewsAgent["📰 뉴스 에이전트\n(검색+요약)"]
        ScheduleAgent["📅 일정 에이전트\n(캘린더 전담)"]
        ShopAgent["🛒 쇼핑 에이전트\n(구매 자동화)"]
    end

    User --> UI
    UI --> Brain
    Brain --> Router
    Router -->|"단순/로컬"| Ollama
    Router -->|"복잡/빠른"| GroqAPI
    Router -->|"긴 문서"| GeminiAPI

    Ollama --> MainAgent
    GroqAPI --> MainAgent
    GeminiAPI --> MainAgent

    MainAgent --> MCP
    MainAgent --> A2A
    MainAgent --> Memory

    A2A --> NewsAgent
    A2A --> ScheduleAgent
    A2A --> ShopAgent

    NewsAgent --> SearchMCP
    ScheduleAgent --> CalMCP
    ShopAgent --> BrowserMCP

    Memory --> MD
    Memory --> ActionLog
    Memory --> VectorDB
    Memory --> Profile
```

---

## 2. 대화 흐름 (Sequence Diagram)

```mermaid
sequenceDiagram
    actor 사용자
    participant UI as PyQt6 UI
    participant Router as Intelligent Router
    participant Ollama as Ollama (로컬)
    participant MCP as MCP 서버
    participant Memory as 메모리 (MD+ChromaDB)

    사용자->>UI: "내일 오후 2시에 캡스톤 발표 일정 등록해줘"
    UI->>Memory: 과거 관련 대화 검색
    Memory-->>UI: "저번 캡스톤 관련 대화 3개 발견"

    UI->>Router: 메시지 + 과거 컨텍스트 전달
    Router->>Router: 난이도 분류 (단순 → 로컬)
    Router->>Ollama: 컨텍스트 포함 요청

    Ollama->>Ollama: "calendar 도구 써야겠다" 판단
    Ollama->>MCP: calendar.create_event() 호출
    MCP-->>Ollama: 일정 등록 완료 응답

    Ollama-->>UI: "내일 오후 2시 캡스톤 발표 등록했어!"
    UI->>Memory: 대화 내용 MD 파일 저장
    UI->>Memory: Action Log에 기록\n("캘린더 등록: 캡스톤 발표 2026-03-25 14:00")
    UI-->>사용자: 응답 표시
```

---

## 3. MCP 동작 원리 상세

```mermaid
flowchart LR
    Input["사용자 입력\n'메모장에 할일 적어줘'"]

    subgraph Ollama_Block ["🦙 Ollama (Tool Calling)"]
        Understand["명령 이해"]
        SelectTool["도구 선택\n→ FileSystem MCP"]
        GenCode["실행 파라미터 생성\n{path: '할일.txt', content: '...'}"]
    end

    subgraph MCP_Block ["🔧 MCP 서버"]
        Receive["파라미터 수신"]
        Execute["실제 실행\nopen('할일.txt', 'w').write(...)"]
        Return["결과 반환\n'저장 완료'"]
    end

    Result["✅ '메모장에 저장했어!'"]

    Input --> Understand
    Understand --> SelectTool
    SelectTool --> GenCode
    GenCode --> Receive
    Receive --> Execute
    Execute --> Return
    Return --> Result
```

---

## 4. A2A 에이전트 통신 구조

```mermaid
flowchart TD
    User2(["사용자: '오늘 AI 뉴스 찾아서\n요약하고 캘린더에 정리 시간 잡아줘'"])

    MainAgent2["🎯 메인 에이전트\nOllama 두뇌"]

    subgraph A2A_Flow ["A2A 프로토콜 (HTTP + JSON-RPC)"]
        NewsAgent2["📰 뉴스 에이전트\n→ Brave 검색\n→ 내용 수집\n→ 요약 생성"]
        ScheduleAgent2["📅 일정 에이전트\n→ 빈 시간 파악\n→ Google Calendar 등록"]
    end

    FinalResponse["✅ '오늘 AI 뉴스 5개 요약했고\n내일 오전 10시에 정리 시간 잡았어!'"]

    User2 --> MainAgent2
    MainAgent2 -->|"Task 1: 뉴스 수집"| NewsAgent2
    MainAgent2 -->|"Task 2: 일정 등록"| ScheduleAgent2
    NewsAgent2 -->|"요약 결과 반환"| MainAgent2
    ScheduleAgent2 -->|"등록 완료 반환"| MainAgent2
    MainAgent2 --> FinalResponse
```

---

## 5. 메모리 시스템 구조

```mermaid
flowchart TB
    subgraph Chat_Files ["📂 채팅 파일 (MD)"]
        C1["chat_20260320_1030\n_CppRectangle과제.md"]
        C2["chat_20260321_1800\n_교회수련회일정.md"]
        C3["chat_20260324_1432\n_개인비서설계.md"]
    end

    subgraph Index ["📋 인덱스"]
        TrayIdx["tray_index.json\n(사이드트레이 목록)"]
        ActionIdx["action_log.json\n(수행 명령 전체 기록)"]
    end

    subgraph Vector ["🔍 벡터 DB (ChromaDB)"]
        V1["임베딩 벡터\n(의미 기반 검색)"]
    end

    subgraph Profile2 ["👤 사용자 프로필"]
        P1["user_profile.md\n- 이름: 최윤식\n- 전공: 임베디드 SW\n- 깃허브: ChoiYunSik02\n- 자주 쓰는 기능: C++, 캘린더\n- 패턴: 화요일 과제 제출"]
    end

    NewChat["새 대화 발생"] --> C3
    C3 --> TrayIdx
    C3 --> ActionIdx
    C3 --> V1
    V1 -->|"의미 검색"| SearchResult["관련 과거 대화 추출"]
    P1 -->|"컨텍스트 주입"| SearchResult
```

---

## 6. 사이드 트레이 & 채팅방 구조

```mermaid
flowchart LR
    subgraph Tray ["📋 사이드 트레이"]
        Today["● 오늘"]
        T1["📌 개인비서 설계"]
        T2["📌 Ollama MCP 연동"]

        ThisWeek["● 이번 주"]
        T3["📌 교회 수련회 일정"]
        T4["📌 C++ ch02 과제"]

        LastWeek["● 지난주"]
        T5["📌 졸업학점 계획"]

        Today --> T1
        Today --> T2
        ThisWeek --> T3
        ThisWeek --> T4
        LastWeek --> T5
    end

    subgraph AutoTitle ["🏷️ 자동 주제 생성"]
        FirstMsg["첫 메시지 입력"]
        TitleGen["Ollama가 10자 이내\n주제 자동 생성"]
        FileName["파일명 생성\nchat_YYYYMMDD_HHMM\n_주제.md"]

        FirstMsg --> TitleGen --> FileName
    end

    subgraph ChatWindow ["💬 채팅 창"]
        ChatArea["대화 내역 표시"]
        InputBox["메시지 입력창"]
        ActionBadge["⚡ 수행한 액션 뱃지\n(캘린더 등록, 파일저장 등)"]
    end

    T1 -->|"클릭"| ChatArea
```

---

## 7. Intelligent Router (모델 자동 선택)

```mermaid
flowchart TD
    Input2["사용자 입력"]
    Classify["난이도 + 프라이버시 분류"]

    Check1{"개인정보\n포함?"}
    Check2{"문서 길이\n32k 토큰 초과?"}
    Check3{"복잡한\n추론 필요?"}

    Local["🦙 Ollama 로컬\nQwen2.5 7B\n(완전 무료 + 오프라인)"]
    Groq["⚡ Groq API\nLlama 3.3 70B\n(무료 tier, 초고속)"]
    Gemini["🌟 Gemini Flash\n(무료 tier, 1M 토큰)"]

    Input2 --> Classify --> Check1
    Check1 -->|"Yes (민감)"| Local
    Check1 -->|"No"| Check2
    Check2 -->|"Yes (긴 문서)"| Gemini
    Check2 -->|"No"| Check3
    Check3 -->|"Yes (복잡)"| Groq
    Check3 -->|"No (단순)"| Local
```

---

## 8. 스킬 시스템 (OpenClaw 방식 참고)

```mermaid
flowchart LR
    subgraph Skills ["📁 skills/"]
        S1["📁 calendar/\nSKILL.md"]
        S2["📁 github/\nSKILL.md"]
        S3["📁 shopping/\nSKILL.md"]
        S4["📁 news/\nSKILL.md"]
        S5["📁 memo/\nSKILL.md"]
    end

    subgraph SKILL_Content ["SKILL.md 내용 예시"]
        SC["## 캘린더 등록 스킬\n\n### 설명\n사용자가 일정 등록을\n요청할 때 사용\n\n### 사용 도구\n- calendar MCP 서버\n- Google Calendar API\n\n### 예시 명령\n'내일 2시에 미팅 잡아줘'\n'다음주 월요일 수업 등록해줘'"]
    end

    Ollama2["🦙 Ollama"] -->|"스킬 파일 읽어서\n동작 방식 이해"| Skills
    S1 --> SC

    NewSkill["💡 새 스킬 추가\n= SKILL.md 파일 하나 추가하면 끝"]
```

---

## 9. 구현 단계별 계획

```mermaid
gantt
    title 개인비서 AI 개발 일정
    dateFormat  YYYY-MM-DD
    section 1단계 기반
    PyQt6 UI 레이아웃        :a1, 2026-03-25, 7d
    Ollama API 연동          :a2, after a1, 5d
    MD 파일 저장 시스템      :a3, after a1, 5d

    section 2단계 메모리
    ChromaDB 벡터 DB 연동   :b1, after a3, 7d
    사이드 트레이 구현       :b2, after a3, 5d
    Action Log 시스템        :b3, after b1, 5d

    section 3단계 MCP
    FileSystem MCP           :c1, after b3, 3d
    Calendar MCP             :c2, after c1, 4d
    Web Search MCP           :c3, after c2, 4d
    GitHub MCP               :c4, after c3, 4d
    Browser MCP (Playwright) :c5, after c4, 7d

    section 4단계 고급
    Intelligent Router       :d1, after c5, 5d
    A2A 에이전트 구조        :d2, after d1, 7d
    스킬 시스템              :d3, after d2, 5d
    음성입력 (Whisper)       :d4, after d3, 5d
```

---

## 10. 기술 스택 최종 확정

```mermaid
mindmap
  root((개인비서 AI))
    UI
      PyQt6
      사이드트레이
      채팅창
      음성입력 Whisper
    두뇌
      Ollama 로컬
        Qwen2.5 7B
        EXAONE 3.5
      Groq API 무료
      Gemini Flash 무료
    메모리
      MD 파일 저장
      ChromaDB 벡터DB
      Action Log JSON
      사용자 프로필 MD
    MCP 도구
      Google Calendar
      FileSystem
      Brave 웹검색
      GitHub
      Playwright 브라우저
    A2A
      메인 오케스트레이터
      뉴스 에이전트
      일정 에이전트
      쇼핑 에이전트
```

---

## 11. 비용 계획

| 컴포넌트 | 도구 | 비용 |
|---|---|---|
| UI 프레임워크 | PyQt6 | **₩0** |
| 로컬 LLM | Ollama + Qwen2.5 7B | **₩0** |
| 클라우드 LLM (보조) | Groq 무료 tier | **₩0** |
| 클라우드 LLM (긴 문서) | Gemini Flash 무료 tier | **₩0** |
| 벡터 DB | ChromaDB | **₩0** |
| 웹 검색 | Brave Search API 무료 2000회/월 | **₩0** |
| 캘린더 | Google Calendar API | **₩0** |
| 브라우저 자동화 | Playwright | **₩0** |
| 음성인식 | Whisper (로컬) | **₩0** |
| **총합** | | **완전 무료 ₩0** |

---

## 12. 폴더 구조

```
📁 personal-ai-assistant/
├── 📁 app/
│   ├── main.py              # PyQt6 앱 진입점
│   ├── ui/
│   │   ├── chat_window.py   # 채팅창
│   │   └── tray_panel.py    # 사이드 트레이
│   ├── brain/
│   │   ├── router.py        # Intelligent Router
│   │   ├── ollama_client.py # Ollama 연동
│   │   └── groq_client.py   # Groq API 연동
│   ├── memory/
│   │   ├── md_manager.py    # MD 파일 관리
│   │   ├── action_log.py    # Action Log
│   │   └── vector_db.py     # ChromaDB
│   ├── mcp/
│   │   ├── calendar_mcp.py  # 캘린더 MCP
│   │   ├── file_mcp.py      # 파일시스템 MCP
│   │   ├── search_mcp.py    # 웹검색 MCP
│   │   ├── github_mcp.py    # GitHub MCP
│   │   └── browser_mcp.py   # 브라우저 MCP
│   └── agents/
│       ├── main_agent.py    # 메인 오케스트레이터
│       ├── news_agent.py    # 뉴스 에이전트
│       ├── schedule_agent.py# 일정 에이전트
│       └── shop_agent.py    # 쇼핑 에이전트
│
├── 📁 memory/
│   ├── 📁 chats/            # 채팅방별 MD 파일
│   ├── 📁 vector_index/     # ChromaDB 인덱스
│   ├── tray_index.json      # 트레이 목록
│   ├── action_log.json      # 전체 액션 기록
│   └── user_profile.md      # 사용자 프로필
│
└── 📁 skills/
    ├── 📁 calendar/SKILL.md
    ├── 📁 github/SKILL.md
    ├── 📁 shopping/SKILL.md
    ├── 📁 news/SKILL.md
    └── 📁 memo/SKILL.md
```

---

> 📌 **핵심 요약**: Ollama는 "두뇌" 역할만 하고, MCP 서버들이 실제 도구로 동작하며, A2A는 에이전트끼리 HTTP로 통신하는 구조. 세 가지 모두 함께 사용 가능하며 비용은 완전 무료.
