# 🗺️ 캠퍼스 맛집 지도 앱 — 캡스톤 설계 문서

---

## 1. 전체 시스템 구조

```mermaid
block-beta
  columns 3

  User(["👤 사용자 (학생)"])
  space
  Firebase[("🔥 Firebase")]

  space space space

  block:App:1
    columns 1
    label["📱 Flutter 앱"]
    Map["① 지도 홈"]
    List["② 목록"]
    Detail["③ 상세 페이지"]
    Review["④ 리뷰 작성"]
    Fav["⑤ 즐겨찾기"]
  end

  space

  block:Backend:1
    columns 1
    label2["☁️ 백엔드 서비스"]
    Auth["Firebase Auth\n구글 로그인"]
    DB["Firestore DB\n식당·리뷰 데이터"]
    Storage["Firebase Storage\n사진 저장"]
    Maps["Google Maps API\n지도·핀 표시"]
  end

  User --> App
  App --> Firebase
  Firebase --> Backend
```

---

## 2. 화면 흐름 (User Flow)

```mermaid
flowchart TD
    A(["앱 실행"]) --> B{"로그인 여부"}
    B -- 미로그인 --> C["구글 로그인"]
    C --> D
    B -- 로그인됨 --> D["🗺️ 홈 지도 화면"]

    D --> E["📍 맛집 핀 클릭"]
    D --> F["📋 목록 보기"]
    D --> G["⭐ 즐겨찾기"]

    E --> H["🍽️ 식당 상세 페이지"]
    F --> H

    H --> I["📝 리뷰 작성"]
    H --> J["❤️ 즐겨찾기 추가"]

    I --> K["별점 + 사진 + 한줄평 입력"]
    K --> L["Firebase에 저장"]
    L --> H
```

---

## 3. 데이터 구조

```mermaid
erDiagram
    USER {
        string uid
        string name
        string email
        string profileImage
    }

    RESTAURANT {
        string id
        string name
        string category
        float latitude
        float longitude
        string address
        string phone
        float avgRating
        int reviewCount
    }

    REVIEW {
        string id
        string restaurantId
        string userId
        float rating
        string comment
        string imageUrl
        timestamp createdAt
    }

    FAVORITE {
        string userId
        string restaurantId
        timestamp savedAt
    }

    USER ||--o{ REVIEW : "작성"
    USER ||--o{ FAVORITE : "저장"
    RESTAURANT ||--o{ REVIEW : "보유"
    RESTAURANT ||--o{ FAVORITE : "저장됨"
```

---

## 4. 기술 스택

```mermaid
block-beta
  columns 4

  A["📱 Flutter\n앱 개발"]:1
  B["🔥 Firebase\n백엔드/DB"]:1
  C["🗺️ Google Maps\n지도 API"]:1
  D["🔐 Firebase Auth\n로그인"]:1
```

---

## 5. 개발 일정 (16주)

```mermaid
gantt
    title 캡스톤 개발 일정
    dateFormat  YYYY-MM-DD
    section 기초
    Flutter 기초 학습        :a1, 2025-03-01, 3w
    환경 세팅 및 프로젝트 생성 :a2, after a1, 1w
    section 핵심 기능
    Google Maps 연동         :b1, after a2, 2w
    Firebase 연동 및 데이터   :b2, after b1, 3w
    리뷰·별점·즐겨찾기        :b3, after b2, 3w
    section 마무리
    UI 다듬기 및 데이터 입력  :c1, after b3, 2w
    테스트 및 버그 수정       :c2, after c1, 1w
    발표 준비                :c3, after c2, 1w
```

---

## 6. 차별화 포인트

| 기능 | 설명 |
|------|------|
| 🕐 혼잡도 표시 | 점심·저녁 시간대별 붐비는 정도 |
| 🏷️ 태그 필터 | `#혼밥가능` `#단체모임` `#가성비` |
| 📸 사진 피드 | 최근 리뷰 사진 인스타 스타일로 모아보기 |
