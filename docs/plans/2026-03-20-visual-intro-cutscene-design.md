# Visual Intro & Cutscene Cinematic System Design

**Date:** 2026-03-20
**Status:** Approved

## Problem

현재 인트로와 컷씬이 텍스트 타이핑 효과만으로 구성되어 시각적으로 밋밋함. 게임 세계에 빠져들기 어려움.

## Solution: Scene Sequencer Engine

씬 데이터 배열 기반 시퀀서 엔진을 구축하여, 인트로와 컷씬 모두 멀티씬 시네마틱 경험으로 전환.

## Architecture

### Scene Data Structure

```javascript
{
  bg: 'url or gradient',       // 배경 이미지/그라디언트
  text: '스토리 텍스트',         // 내러티브
  transition: 'fadeIn',         // 트랜지션 타입
  duration: 3000,               // 씬 지속 시간(ms)
  textEffect: 'typewriter',    // 텍스트 효과
  overlay: 'dark',             // 오버레이 (dark/light/none)
  effects: ['vignette'],       // 추가 시각 효과
}
```

### Transition Types (5)

| Type | Effect | Use Case |
|------|--------|----------|
| `fadeIn` | 서서히 등장 | 오프닝, 일반 전환 |
| `slideLeft` | 왼쪽으로 슬라이드 | 이동/진행 |
| `zoomIn` | 확대하며 등장 | 발견, 집중 |
| `blur` | 흐림→선명 | 깨어남, 회상 |
| `glitch` | 글리치 효과 | 긴장감, 디지털 |

### Text Effects (3)

- `typewriter` - 기존 타이핑 효과 (기본)
- `fadeLines` - 줄 단위 페이드인
- `reveal` - 마스크 해제 효과

### Playback Flow

```
씬 배열 → [씬1] → 트랜지션 → [씬2] → ... → 콜백(게임시작/스테이지로드)
                                                ↑
                                  클릭/탭으로 건너뛰기
```

## Intro Sequence (6 Scenes)

| Scene | Background | Text | Transition | Effects |
|-------|-----------|------|------------|---------|
| 1 | 검은 화면 | *(EXEM 로고만)* | fadeIn | 글로우 펄스 |
| 2 | 어두운 사무실 | "퇴근 후, 텅 빈 사무실..." | blur | 타이핑, 비네팅 |
| 3 | 책상 클로즈업 | "책상 위에 놓인 메모 한 장..." | zoomIn | 타이핑 |
| 4 | 잠긴 문 | "문을 열려 했지만... 잠겨 있다." | slideLeft | 타이핑, 화면 흔들림 |
| 5 | 어두운 복도 | "EXEM의 핵심가치를 증명하라..." | glitch | fadeLines |
| 6 | 사무실 전경 | 스테이지 아이콘 + "탈출 시작하기" 버튼 | fadeIn | 파티클 |

- Scenes 1-5: 자동 진행 (3-4초) + 클릭으로 다음 씬
- Scene 6: 시작 버튼으로 유저가 직접 게임 시작
- 건너뛰기 버튼 상시 표시

## Stage Cutscenes (2-3 Scenes Each)

### Stage 1→2 (책상→파일 캐비닛)

| Scene | Background | Text | Transition |
|-------|-----------|------|------------|
| 1 | 서랍 클로즈업 | "서랍에서 찾은 단서 카드를 손에 쥐고..." | zoomIn |
| 2 | 파일 캐비닛 | "파일 캐비닛의 금속 표면이 희미하게 빛납니다." | slideLeft |

### Stage 2→3 (파일 캐비닛→컴퓨터)

| Scene | Background | Text | Transition |
|-------|-----------|------|------------|
| 1 | 열린 캐비닛 | "캐비닛 안에서 발견한 USB..." | fadeIn |
| 2 | 모니터 화면 | "컴퓨터 화면이 깜빡이며 켜집니다." | glitch |

### Stage 3→4 (컴퓨터→금고)

| Scene | Background | Text | Transition |
|-------|-----------|------|------------|
| 1 | 화면 코드 | "모니터에 나타난 숫자의 의미는..." | blur |
| 2 | 금고 | "벽 뒤에 숨겨진 금고가 모습을 드러냅니다." | slideLeft |

### Stage 4→5 (금고→출구)

| Scene | Background | Text | Transition |
|-------|-----------|------|------------|
| 1 | 열린 금고 | "금고 안의 마지막 열쇠..." | zoomIn |
| 2 | 긴 복도 | "출구를 향해 마지막 발걸음을 옮깁니다." | slideLeft |
| 3 | 탈출문 | "하지만 마지막 관문이 남아있습니다..." | fadeIn |

## Visual Effects

### Vignette
- `box-shadow: inset 0 0 150px rgba(0,0,0,0.7)`
- 인트로/컷씬 전체에 기본 적용

### Particles
- Canvas 기반 20-30개 미세 입자
- 인트로 마지막 씬, 긴장감 높은 컷씬에 사용

### Glitch
- RGB 채널 분리 + 수평 오프셋
- CSS `clip-path` + `transform`

### Skip UI
- 우상단 반투명 "건너뛰기 >>" 버튼
- 클릭 시 전체 시퀀스 즉시 종료
- 개별 씬 클릭/탭 시 다음 씬

### Progress Indicator
- 하단 점(dot)으로 현재 씬 위치 표시
- ○○●○○○ 형태

## Constraints

- 단일 HTML 파일 유지
- Vanilla JS/CSS only (외부 라이브러리 없음)
- Vercel 정적 배포
- 배경 이미지: Unsplash 활용
- 사운드 없음 (현재 프로젝트 범위 외)

## Implementation Notes

- 기존 `showCutscene()` 함수를 씬 시퀀서로 교체
- 기존 `#intro` HTML을 씬 시퀀서 기반으로 재구성
- `stages[].cutscene` 문자열을 씬 배열로 변경
- 기존 `typeText()` 함수 재활용
