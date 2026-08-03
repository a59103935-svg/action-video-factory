---
name: action-video-factory-storyboard-visual
description: >
  텍스트 스토리보드(20_Storyboard.md)를 받아 스케치 스타일의 실제 그림콘티 패널 이미지를 생성.
  각 샷을 "storyboard sketch, pencil drawing, black and white" 스타일로 nano banana pro 또는
  Flux Dev로 생성하고, 패널을 격자(grid) 형태로 조립. 샷 번호, 샷 타입, 카메라 무빙 화살표,
  액션 설명, 대사가 포함된 완성형 그림콘티 시트 출력.
  픽사/라타투이 스타일의 전문 그림콘티와 동일한 포맷.
  Action Video Factory 파이프라인의 2.5단계.
  트리거: 그림콘티, 스토리보드 이미지, 콘티 시각화, 패널 생성, storyboard visual
type: skill
project: skills-library
plugin: action-video-factory
aliases: [storyboard-visual, 그림콘티, 콘티이미지, panel-generation]
tags: [type/skill, plugin/action-video-factory, scope/craft, topic/storyboard, topic/visual]
status: active
---

# Action Video Factory — Storyboard Visual (그림콘티 패널 생성)

텍스트 스토리보드를 받아 실제 눈으로 볼 수 있는 그림콘티 시트로 만드는 단계.
각 샷이 스케치 이미지 + 어노테이션으로 구성된 패널이 되고, 패널들이 격자로 조립된다.

목표 출력 포맷: 씬당 1장 (A4/16:9), 6~9개 패널, 전문 그림콘티 레이아웃.

`../../references/action-video-foundations.md`를 로드한다.

---

## 언제 사용하나

- `action-video-factory-storyboard`가 20_Storyboard.md를 완성한 직후
- 감독/프로듀서에게 시각적으로 공유해야 할 때
- 촬영 전 블로킹 검토용 실물 콘티가 필요할 때

---

## 그림콘티 스타일 정의

### 스케치 스타일 (기본값)
전문 그림콘티의 표준 — 픽사, 드림웍스, 한국 영화사의 실제 콘티와 동일한 느낌.

```
스타일 프롬프트 접미사 (모든 패널에 고정 사용):
"storyboard panel, pencil sketch illustration, black and white,
 professional film storyboard style, clean linework,
 cinematic composition, slightly rough hand-drawn quality,
 no color, white background, comic book panel framing"
```

### 스타일 변형 옵션

| 스타일 | 추가 키워드 | 분위기 |
|---|---|---|
| **클래식 스케치** (기본) | pencil sketch, hand-drawn | 픽사/드림웍스 콘티 |
| **인크 드로잉** | ink drawing, bold lines, noir | 느와르/스릴러 콘티 |
| **러프 섬네일** | rough thumbnail, quick sketch, gestural | 초기 아이디어 단계 |
| **세미 컬러** | light watercolor wash, minimal color | 감정 강조 콘티 |

---

## 패널 생성 워크플로우

### 1단계 — 샷 리스트 정리

`20_Storyboard.md`에서 각 샷의 핵심 정보 추출:
- 샷 번호
- 샷 사이즈 + 앵글
- 키프레임 SUBJECT + COMPOSITION (이 두 개만 이미지 생성에 사용)
- 카메라 무빙 (화살표로 패널에 표시)
- 액션 설명 (1줄)
- 대사 (있으면, 최대 1줄)

### 2단계 — 패널 이미지 생성

모델: **nano banana pro** (기본) 또는 **Flux Dev** (더 정교한 드로잉 필요 시)

각 패널 프롬프트 구조:
```
[씬 내용 설명, SUBJECT + COMPOSITION에서 추출]
[스타일 프롬프트 접미사 고정]

예시:
"two characters facing each other in a narrow corridor,
 medium shot, eye level, tension between them,
 foreground character turns away,
 storyboard panel, pencil sketch illustration, black and white,
 professional film storyboard style, clean linework,
 cinematic composition, no color, white background"
```

**중요**: 포토리얼 키프레임 프롬프트(4단계용)와 다르다.
스케치 스타일이므로 인물 외모 디테일보다 **구도와 동작**에 집중.

### 3단계 — 어노테이션 레이어 추가

각 패널 이미지 위에 또는 아래에 텍스트 어노테이션을 배치:

```
┌─────────────────────────────────────────┐
│ [씬번호]-[샷번호]    [샷타입] / [앵글] │  ← 헤더
│                                         │
│   [생성된 스케치 이미지]              │ ← 패널 이미지
│        ↙ (무빙 화살표)                  │
│                                         │
├─────────────────────────────────────────┤
│ 렌즈: [mm] / STYLE: [코드]             │  ← 기술 정보
│ 액션: [1줄 설명]                        │  ← 동작
│ 대사: "[대사]"                          │  ← 대사 (있으면)
│ → [다음 샷 번호] / [트랜지션 타입]     │  ← 전환
└─────────────────────────────────────────┘
```

### 4단계 — 격자 조립

씬당 패널들을 격자 레이아웃으로 조립:

| 씬 내 샷 수 | 레이아웃 |
|---|---|
| 3~4개 | 2열 × 2행 (4패널) |
| 5~6개 | 3열 × 2행 (6패널) |
| 7~9개 | 3열 × 3행 (9패널) |
| 10개 이상 | 여러 장으로 분할 |

조립 방법 (Python):

```python
from PIL import Image, ImageDraw, ImageFont
import math

def assemble_storyboard(panels, scene_title, output_path, cols=3):
    """
    panels: 패널 이미지 경로 리스트
    scene_title: 씬 제목
    cols: 열 수
    """
    PANEL_W, PANEL_H = 560, 460   # 패널 크기 (이미지 360px + 어노 100px)
    MARGIN = 20
    HEADER_H = 60

    rows = math.ceil(len(panels) / cols)
    sheet_w = cols * (PANEL_W + MARGIN) + MARGIN
    sheet_h = HEADER_H + rows * (PANEL_H + MARGIN) + MARGIN

    sheet = Image.new('RGB', (sheet_w, sheet_h), 'white')
    draw = ImageDraw.Draw(sheet)

    # 헤더
    draw.text((MARGIN, 15), scene_title, fill='black')
    draw.line([(MARGIN, HEADER_H), (sheet_w - MARGIN, HEADER_H)], fill='black', width=2)

    # 패널 배치
    for i, panel_path in enumerate(panels):
        row, col = divmod(i, cols)
        x = MARGIN + col * (PANEL_W + MARGIN)
        y = HEADER_H + MARGIN + row * (PANEL_H + MARGIN)
        panel_img = Image.open(panel_path).resize((PANEL_W, 360))
        sheet.paste(panel_img, (x, y))

    sheet.save(output_path)
    return output_path
```

설치:
```bash
pip install Pillow --break-system-packages
```

---

## 출력물

```
20_Storyboard_Visual/
├── scene-01-sheet.png        ← 씬 1 전체 그림콘티 시트
├── scene-01-panels/
│   ├── shot-01-01.png        ← 개별 패널 (원본)
│   ├── shot-01-02.png
│   └── ...
├── scene-02-sheet.png
└── ...
```

최종 시트 하나가 공유 가능한 **그림콘티 페이지** 1장.

---

## 카메라 무빙 화살표 규칙

패널 이미지 위에 화살표를 overlay로 추가하거나, 프롬프트에 텍스트로 지시:

| 무빙 | 화살표 표현 |
|---|---|
| Dolly In | → (중앙으로 향하는 화살표) |
| Dolly Out | ← (중앙에서 멀어지는 화살표) |
| Pan Left | ←── |
| Pan Right | ──→ |
| Tilt Up | ↑ |
| Tilt Down | ↓ |
| Crane Up | ↑↑ (굵게) |
| Static | • (점) |
| Handheld | ≈ (물결) |

이미지 생성 프롬프트에 "with camera movement arrow showing [방향]" 추가하거나,
PIL로 overlay 그리기.

---

## 예시 패널 프롬프트들

**씬 7-1 (WS, 옥상, 두 인물):**
```
"rooftop at night, rain, two figures standing apart,
 wide shot, eye level, one figure with back turned,
 the other approaching from behind, rainy atmosphere,
 storyboard panel, pencil sketch illustration, black and white,
 professional film storyboard style, clean linework, no color"
```

**씬 7-3 (CU, 눈 마주침):**
```
"extreme close-up of two faces turning to face each other,
 eye level, intense emotional moment,
 rain on faces, dramatic lighting implied through shadow,
 storyboard panel, pencil sketch illustration, black and white,
 professional film storyboard style, clean linework, no color"
```

---

## 제약

- 스케치 스타일 프롬프트 접미사는 모든 패널에 고정 — 일관성이 생명.
- 실제 인물 likeness 불가 — "a man", "a woman", "a figure" 사용.
- @단계 포토리얼 키프레임 프롬프트와 혼용 금지 — 전혀 다른 목적의 프롬프트.
- 패널당 이미지 생성 1회, 스타일 일관성이 깨지면 전체 씬 재생성 (개별 교체 금지).
- 패널 해상도: 최소 560×360px (시트 조립 후 A4 인쇄 가능 수준).
