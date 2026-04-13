# EXEM 방탈출 게임

EXEM 핵심가치 기반 온라인 방탈출 게임입니다. 신규입사자 온보딩 또는 팀빌딩에서 활용할 수 있습니다.

## 배포 URL

https://exem-escape.vercel.app

## 게임 구성

5개 스테이지로 구성되며, 각 스테이지에서 EXEM 핵심가치 관련 퀴즈를 풀어 탈출합니다.

| 스테이지 | 장소 | 핵심가치 |
|----------|------|----------|
| 1 | 책상 서랍 | 열정 (Passion) |
| 2 | 파일 캐비닛 | 신속 (Speed) |
| 3 | 컴퓨터 | 몰입 (Flow) |
| 4 | 금고 | 성취/책임, 협력/수용 |
| 5 | 탈출 출구 | 투명/진정성 |

## 퀴즈 유형 (8종)

`choice` (객관식), `fill` (주관식), `dial` (다이얼 맞추기), `match` (연결하기), `order` (순서 맞추기), `ox` (O/X), `scenario` (시나리오), `toggle` (토글 선택)

## 구조

```
exem-escape/
├── index.html           # 전체 게임 (HTML + CSS + JS, ~3100줄)
├── logo.png             # EXEM 로고
├── end*.jpg             # 엔딩 배경 이미지
├── qrcode_deploy.png    # 배포 URL QR코드
├── Unica77LLTT-*.ttf    # 게임 폰트
└── README.md
```

## 수정 방법

### 퀴즈 문제 수정

`index.html` 내 `quizData` 객체에서 수정합니다.

```javascript
const quizData = {
  stage1: [
    {
      type: "choice",              // 퀴즈 유형
      question: "질문 내용",        // 문제
      options: ["보기1", "보기2"],  // 선택지 (choice 유형)
      answer: 0,                   // 정답 인덱스
      hint: "힌트 내용"            // 힌트
    },
    // ...
  ],
  // stage2, stage3, ...
};
```

### 퀴즈 유형별 데이터 구조

```javascript
// choice (객관식)
{ type: "choice", question: "...", options: ["A", "B", "C"], answer: 1 }

// fill (주관식)
{ type: "fill", question: "...", answer: "정답텍스트" }

// dial (다이얼)
{ type: "dial", question: "...", answer: "1234" }

// match (연결하기)
{ type: "match", question: "...", pairs: [["왼쪽", "오른쪽"], ...] }

// order (순서)
{ type: "order", question: "...", items: ["첫째", "둘째", "셋째"] }

// ox (O/X)
{ type: "ox", question: "...", answer: true }

// scenario (시나리오)
{ type: "scenario", question: "...", options: ["행동1", "행동2"], answer: 0 }

// toggle (복수 선택)
{ type: "toggle", question: "...", options: ["항목1", "항목2", "항목3"], answer: [0, 2] }
```

### 스테이지 추가/삭제

`quizData`에 `stage6` 등을 추가하고, 스테이지 진행 로직에서 총 스테이지 수를 수정합니다.

### 시네마틱(인트로/컷씬) 수정

`playCinematic()` 함수에 전달하는 씬 배열을 수정합니다.

```javascript
playCinematic([
  { text: "표시할 텍스트", transition: "fadeIn", textEffect: "typewriter", duration: 3000 },
  // ...
], callback);
```

트랜지션: `fadeIn`, `slideLeft`, `zoomIn`, `blur`, `glitch`
텍스트 효과: `typewriter`, `fadeLines`, `reveal`

## 배포

```bash
# Vercel로 배포
npx vercel --prod --yes
```

## 기술 스택

- 순수 HTML/CSS/JS (단일 파일, 외부 라이브러리 없음)
- Vercel 배포
