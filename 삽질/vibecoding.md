
# 바이브 코딩 잘하는 법
> _AI와 대화하며 자연어로 기능을 설계, 구현, 테스트하는 새로운 개발 방식_

---

## 1. 바이브 코딩이란?

바이브 코딩(Vibe Coding)은  
**AI 코딩 에이전트(ChatGPT, GitHub Copilot 등)**와 대화하면서  
자연어를 중심으로 코드 작성/수정/확장까지 이루어지는  
**“소통 기반 개발”** 방식이다.

### 기존 개발과의 차이
| 전통 개발 | 바이브 코딩 |
|-----------|--------------|
| 명세서 → 설계 → 코드 | 설명 → 대화 → 코드 생성 |
| IDE 중심 | 대화 중심 + IDE 연동 |
| 함수, 모듈로 분할 설계 | 대화 흐름에 따라 점진적 구현 |
| 스스로 다 구현 | AI가 설계/코드/테스트 보조 |

---

## 2. 실전 워크플로우

### STEP 1: 문제 정의 (자연어로 설명)
```
“현재 로그인은 이메일/비밀번호만 지원하는데, 구글 OAuth도 붙이고 싶어.”
```

### STEP 2: 기능 요청 (구체적으로 말하기)
```
“Spring Security 기반으로, 구글 OAuth 2.0을 이용해 로그인하는 코드를 작성해줘.”
```

### STEP 3: 코드 확인 & 수정보완
```
“근데 JWT 발급은 빠졌네. 로그인 성공 시 JWT 발급하는 코드도 추가해줘.”
```

### STEP 4: 통합 및 테스트
```
“이제 기존 LoginController에 통합하고, 테스트 코드도 만들어줘.”
```

---

## 3. 프롬프트 설계 노하우

| 상황 | 좋은 프롬프트 예시 |
|------|--------------------|
| 버그 수정 | “이 코드가 NullPointerException을 발생시키는데, 이유와 수정 방법 알려줘.” |
| 기능 추가 | “기존 `UserService`에 OTP 인증 로직을 추가하고 싶어. Redis를 써서 5분 만료되게 해줘.” |
| 리팩토링 | “이 메서드가 너무 길고 복잡한데, 의미별로 분리해서 리팩토링해줘.” |
| 테스트 | “이 기능에 대한 JUnit 테스트 코드를 작성해줘. 성공/실패 케이스 모두 포함해줘.” |

---

## 4. AI에게 잘 말하는 법

### AI가 이해하기 좋은 정보
- 현재 사용 중인 **프레임워크** (예: Spring Boot, Express)
- 기존 코드 일부 (기능, 클래스 이름 등)
- **목표**: “어떻게 되고 싶다” 명확히
- **문제**: “지금 이게 안 되고 있다” 구체화

### 프롬프트 구조 추천
```
[배경 설명]  
→ [현재 코드/상황 요약]  
→ [요청할 작업 구체화]  
→ (선택) 제약 조건 or 추가 정보
```

---

## 5. 실무 활용 예시

### A. Kafka 기반 알림 시스템 만들기
- “Kafka로 알림 메시지 전송하는 Producer/Consumer 코드를 만들어줘.”
- “Slack 메시지 전송 실패 시 DLQ로 보내고, 5초 간격으로 재시도 해줘.”

### B. Elasticsearch 검색 기능 붙이기
- “숙소 이름, 카테고리, 태그 기반으로 검색되는 기능 구현해줘.”
- “자동완성과 오타 허용도 포함해줘.”

### C. 크롤링 시스템 설계
- “Playwright를 사용해서 야놀자 숙소 정보를 수집하는 Python 크롤러 코드를 작성해줘.”
- “세부 페이지에서 대표 이미지, 숙소 설명, 서비스 항목도 같이 수집해줘.”

---

## 6. 주의할 점과 팁

| 실수 유형 | 대처법 |
|-----------|--------|
| 맥락 부족 | “지금 이 코드는 어디서 쓰이고, 어떤 문제를 겪고 있는지”부터 설명 |
| 너무 많은 요청 | 한번에 하나씩 요청하고, 점진적으로 확장 |
| 이해 없이 복붙 | 코드의 의미와 부작용을 꼭 분석 |
| 프롬프트가 두루뭉술 | “어떤 상황에서”, “무엇을”, “어떻게”를 명확히 기술 |

---

## 7. 반복 가능한 바이브 루틴

1. 문제 정의 (자연어 설명)
2. 코드 요청 (기능 중심)
3. 코드 생성 → 수정 요청
4. 기존 프로젝트에 통합
5. 테스트 코드 요청
6. 리팩토링/주석 요청
7. 최종 패키징 (압축 or 배포 스크립트 요청)

---

## 8. 추천 툴 조합

| 목적 | 툴 |
|------|-----|
| 자연어 기반 AI | ChatGPT, GPT-4o, Claude, GitHub Copilot |
| 실시간 실행 | Jupyter Notebook, CodeSandbox, Replit |
| 프롬프트 편집/관리 | Notion, Obsidian, VSCode Prompt Manager |
| IDE 연동 | Continue (VSCode/JetBrains), Cody |

---

## 9. 바이브 코딩 잘하는 사람의 특징

- 자연어로 **기능/문제/요청을 잘 설명**한다
- AI가 작성한 코드를 **비판적으로 검토**하고 수정할 줄 안다
- 점진적으로 **기능을 확장**하며 협업한다
- 도구를 **문제 해결의 수단**으로 정확히 활용한다


---

> **바이브 코딩은 “빨리 구현”이 아니라 “빠르게 소통하고 점점 정교화”하는 과정이다.**  
> AI는 도우미이지, 책임자가 아니다. 결국 **의도와 구조를 설계하는 건 개발자** 자신이다.
