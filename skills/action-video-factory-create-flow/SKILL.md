---
name: action-video-factory-create-flow
description: >
  Action Video Factory의 마스터 오케스트레이터. 대본(복붙 텍스트) 또는 줄콘티에서 시작해
  ScriptBreakdown → Concept → Storyboard → StoryboardVisual → Direction → Generate → Ship
  파이프라인을 단계별 리뷰 포즈와 함께 진행.
  각 단계에서 해당 전용 스킬을 시퀀스. 단일 단계만 필요한 경우 해당 스킬로 직접 라우팅.
  트리거: 영상 만들기, 콘티 작업, 대본 → 영상, 줄콘티 → 영상, 전체 파이프라인, action Video factory
type: skill
project: skills-library
plugin: action-video-factory
aliases: [action-factory, action-pipeline, 영상파이프라인, 콘티영상제작]
tags: [type/skill, plugin/action-video-factory, scope/orchestrator, topic/pipeline]
status: active
---

# Action Video Factory — Create Flow (마스터 오케스트레이터)

대본 텍스트에서 생성 가능한 클립까지, 6개의 전용 스킬을 리뷰 포즈와 함께 시퀀스한다.

`../../references/action-video-foundations.md`를 먼저 로드한다.

```
대본(복붙) → [0] ScriptBreakdown → [1] Concept → [2] Storyboard
          → [2.5] StoryboardVisual(그림콘티) → [3] Direction → [4] Generate → [5] Ship
```

---

## 언제 사용하나

- 대본 텍스트가 있고 영상까지 가야 할 때 — 전체 파이프라인
- 단일 단계만 필요하면 해당 스킬로 직접:
  - 대본 → 줄콘티만 → `action-video-factory-script-breakdown`
  - 방향성만 → `action-video-factory-concept`
  - 텍스트 콘티만 → `action-video-factory-storyboard`
  - 그림콘티만 → `action-video-factory-storyboard-visual`
  - 룩 잠금만 → `action-video-factory-creative-direction`
  - 프롬프트/생성만 → `action-video-factory-higgsfield-prompt`

---

## 단계 0 — Script Breakdown (대본 → 줄콘티)

`action-video-factory-script-breakdown` 호출.
대본 텍스트를 씬별로 분해 — 장소/시간/인물/핵심액션/감정곡선/씬기능.
`00_LineScript.md` + `run_log.md` 생성.

추론 불가한 것만 확인: 로케이션 사진 여부, 플랫폼 (극장/OTT/SNS).

**포즈**: 줄콘티 검토. 씬 분리가 맞는지 확인.

---

## 단계 1 — Concept (방향성 기획)

`action-video-factory-concept` 호출. `10_Concepts.md` 생성.

**포즈**: 3~5개 방향성 제시 → 사용자가 선택. 선택 없이 다음 단계 금지.

---

## 단계 2 — Storyboard (텍스트 콘티)

`action-video-factory-storyboard` 호출.
로케이션 사진이 있으면 이 단계 시작 전에 분석 먼저.
`20_Storyboard.md` 생성.

**포즈**: 샷 플랜 + STYLE 프리셋 배정 검토.

---

## 단계 2.5 — Storyboard Visual (그림콘티)

`action-video-factory-storyboard-visual` 호출.
`20_Storyboard.md`의 각 샷을 스케치 스타일 이미지로 생성 → 격자 조립.
`20_Storyboard_Visual/` 폴더에 씬별 그림콘티 시트 저장.

**포즈**: 그림콘티 검토. 샷 구도가 의도와 맞는지 확인. 수정 요청 시 해당 패널만 재생성.

---

## 단계 3 — Direction (룩 잠금)

`action-video-factory-creative-direction` 호출. `30_Direction.md` 생성.

**포즈**: 룩 사인오프. **이 사인오프 없이는 어떤 영상도 생성하지 않는다 — 크레딧은 실제 돈.**

---

## 단계 4 — Generate (영상 생성)

`action-video-factory-higgsfield-prompt` 호출. `40_Prompts.md` 생성.
키프레임(포토리얼) 먼저 생성 → 검토 → 클립 생성.

**포즈**: 키프레임 검토 후 비디오 크레딧 투입. 클립 검토 후 Ship.

---

## 단계 5 — Ship

최종 편집, 컬러, 인코딩. `assets/final/`에 최종본 저장.

---

## 운영 원칙

- 단계당 리뷰 포즈 하나. 사용자가 압도당하지 않도록.
- 파이프라인은 à la carte: 줄콘티가 이미 있으면 1단계부터; 텍스트 콘티가 있으면 2.5단계부터.
- 그림콘티(2.5단계)와 포토리얼 키프레임(4단계)은 다른 목적 — 혼용 금지.
- 크레딧 탐색 우선: `veo3_1_lite` 탐색 패스 → 히어로 모델 최종 픽.
