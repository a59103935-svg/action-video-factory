---
name: action-video-factory-higgsfield-prompt
description: >
  디렉팅된 스토리보드를 Higgsfield 실행 가능한 프롬프트 패키지로 변환하고 (요청 시) MCP로 직접 생성.
  모델 라우팅 (kling 3.0 기본 / seedance 2.0 분위기-배경, nano banana pro 키프레임,
  minimax_hailuo 자연스러운 인물 반응), 이미지-투-비디오 프롬프트 구조 (모션·카메라·분위기·사운드),
  파라미터 선택, 네거티브, 재시도, 웹 인코딩 핸드오프 담당. Action Video Factory 체인의 4단계.
  트리거: Higgsfield 프롬프트, 영상 생성, 모델 선택, 클립 만들기, 키프레임 생성
type: skill
project: skills-library
plugin: action-video-factory
aliases: [action-prompt, higgsfield-generate, 영상생성, 클립만들기]
tags: [type/skill, plugin/action-video-factory, scope/specialty, topic/higgsfield, topic/generation]
status: active
---

# Action Video Factory — Higgsfield Prompt (생성 엔진)

이 스킬이 엔진이다. 디렉팅된 스토리보드를 정확한 모델 선택, 파라미터, 프롬프트 텍스트로 변환하고,
Higgsfield MCP를 구동해 클립을 생성한다. 업스트림(Concept, Storyboard, Direction)은
이 단계가 기계적이고 정확할 수 있도록 존재한다.

`../../references/action-video-foundations.md`를 로드한다. 프로덕션 배치 전에 `models_explore`로 라이브 MCP 카탈로그를 검증한다 — 모델 목록은 자주 업데이트된다.

---

## 핵심: 두 단계 워크플로우

**모든 Higgsfield 비디오는 이미지-투-비디오다.** 프롬프트 하나로 비디오가 나오지 않는다:

1. **키프레임 스틸 생성** (`generate_image`, nano banana pro 기본 / 텍스트 포함 시 chatgpt image 2).
   이 프레임이 룩이다 — 디렉션 스펙의 빛, 팔레트, grain이 여기 있어야 한다.
2. **스틸을 애니메이트** (`generate_video`). 무엇이 변하는지만 서술 — 카메라, 피사체 모션, 분위기, 사운드.
   장면을 다시 묘사하지 않는다. 키프레임이 이미 고정한 컴포지션을 반복하지 않는다.

"비디오 프롬프트를 텍스트-투-비디오처럼 쓰기"가 가장 흔한 실패다.
**프레임 위의 모션**을 서술하라. 프레임 자체를 서술하지 마라.

---

## 모델 라우팅 (액션씬 특화)

### 기본 라우팅

| 씬 유형 | 모델 | 이유 |
|---|---|---|
| 멀티샷 액션/격투/추격 | `kling3_0` mode pro | 멀티샷 처리, 모션 트랜스퍼, 프레임 정확도 |
| 분위기/배경/루프 | `seedance_2_0` | 싱글 룩 유지, end_image 루프 지원 |
| 자연스러운 인물 반응/감정씬 | `minimax_hailuo` | 최고의 얼굴 물리학, 감정 뉘앙스 |
| 마케 히어로샷 | `cinematic_studio_3_0` 또는 `veo3_1` ultra | 최고 화질, 크레딧 정당화 필요 |
| 장르 컨트롤 필요 | `cinematic_studio_video_v2` genre 파라미터 | action/suspense/intimate 등 |
| 탐색 패스 | `veo3_1_lite` 720p | 빠르고 저렴, 모션 검토용 |
| 오디오 내장 | `wan2_7`, `kling2_6`, `veo3_1` | 동기화 오디오 렌더링 |

### nano banana pro 키프레임 라우팅

| 상황 | 모델 |
|---|---|
| 기본 인물/공간/분위기 | nano banana pro |
| 텍스트·라벨·복잡한 추상 | chatgpt image 2 |
| 실존 인물 likeness | 생성 불가 → 포스트 합성 |
| 브랜드 로고/마크 | 생성 불가 → 포스트 합성 |

---

## 모션 프롬프트 구조

각 샷의 비디오 프롬프트:

```
CAMERA: 단 하나의 무빙, 느리고 단일. 디렉션 스펙의 카메라 언어에 맞게.
SUBJECT MOTION: 프레임에서 물리적으로 무엇이 변하는가, 얼마나 빠르게.
ATMOSPHERE: 공기, 파티클, 빛의 거동, 물/패브릭 움직임 — 샷의 생명력.
PACING: 실시간 / 약한 슬로우모션. 레지스터의 템포 유지.
SOUND: 배경/사일런트는 off; 사운드 납품은 베드 서술 (오디오 지원 모델만).
NEGATIVES: 디렉션 스펙 NOT 목록 그대로 + 모션 클리셰 (크래시줌, 속도변속, AI파티클, 드론 reveal, 모핑 아티팩트, 손/얼굴 워핑).
```

**액션씬 특이사항**:
- 슬로우모션 포인트: 모션 프롬프트에서 정확히 어느 순간에 슬로우인지 명시
- 핸드헬드: "deliberate camera shake, urgent documentary feel"
- 스테디캠: "smooth tracking, immersive follow, no shake"
- 격투 임팩트: "sudden forceful impact, brief motion blur on contact"

---

## 파라미터 체크리스트 (generate_video 호출당)

- `model_id` — 라우팅 테이블에서 선택
- `medias` — 키프레임 `start_image` (루프는 `end_image` 동일 키프레임). media_id/job_id로.
- `aspect_ratio` — 16:9 기본, 21:9 시네마틱, 9:16 모바일
- `duration` — 모델 범위 내. kling3_0: 3~15s, seedance_2_0: 4~15s, minimax_hailuo: 6 또는 10s
- `resolution` / `mode` / `quality` — 탐색은 720p, 최종은 풀 res
- `sound` — 배경/액션 사운드 불필요 → off (크레딧 절감)
- `genre` — cinematic_studio_video_v2에서만, 관련 있을 때만

---

## 프롬프트 패키지 생성 및 실행

`40_Prompts.md` 작성 — 각 샷에 대해:
- 키프레임 프롬프트 + 이미지 모델
- 모션 프롬프트 (CAMERA/SUBJECT/ATMOSPHERE/PACING/SOUND/NEGATIVES)
- 정확한 비디오 모델 + 모든 파라미터
- 네거티브 (디렉션 NOT 목록 완본)

이 패키지 하나로 나중에 누구든 재생성하거나 확장할 수 있다.

**클립 생성 요청 시 순서**:
1. `show_plans_and_credits`로 배치 전 크레딧 확인*2. 모든 키프레임 먼저 생성 (`generate_image`)
3. 키프레임을 디렉션 스펙과 대조 검토 — 벗어난 것은 비디오 크레딧 투입 전에 재생성
4. 샷별 클립 생성 (`generate_video`). `job_display` / `show_generations`으로 폴링
5. 저장: keyframes → `assets/keyframes/`, clips → `assets/clips/`, 파일명 `shot-NN-label.{ext}`
6. `run_log.md`에 모든 생성 로그: 모델, 파라미터, 크레딧

---

## 액션씬 생성 특이사항

**격투 시퀀스**:
- 각 샷을 독립 클립으로. 편집으로 연결.
- 임팩트 샷: 짧게 (3~4s), kling3_0 mode pro
- 롱테이크 시뮬레이션: kling3_0 mode pro, 최대 duration (15s), 탐색 패스 필수

**감정씬**:
- minimax_hailuo 우선 (얼굴 물리학)
- 극도로 느린 Push In: `CAMERA: imperceptibly slow push in, barely perceptible over the full duration`
- 긴 홀드: duration을 길게, motion은 최소화

**추격씬**:
- 여러 짧은 클립 시퀀스. 각 3~5s.
- 스크린 디렉션 일관성 — 모든 클립에서 같은 방향
- 지리 설정 샷: seedance_2_0 Wide, 나머지 추격 샷: kling3_0

---

## 예시

**예시 1 — 격투 임팩트 샷 (PHJ_SLOW_IMPACT)**

키프레임 (nano banana pro):
> SUBJECT: 주먹이 카메라 쪽으로 뻗어나오는 극단적 클로즈업 / SETTING: 어두운 복도, 흐린 배경 / MOOD: 폭력적 순간의 정지 / LIGHT: 강한 측광 sidelight, 주먹에 하이라이트 / COMPOSITION: ECU, Low Angle, 손이 프레임 80% 채움 / LENS: 85mm 얕은 심도 / PALETTE: 어두운 navy, 피부톤, 차가운 그림자 / NOT: 선글라스 반사, 무지개빛 플레어, 슬로우 남발

모션 프롬프트:
```
CAMERA: locked, absolutely static.
SUBJECT MOTION: the fist decelerates from blur to near-stop over the first 2 seconds, extreme slow motion, then snap back to real-time impact and blur.
ATMOSPHERE: faint dust particles drift in the sidelight beam.
PACING: extreme slow then abrupt real-time snap.
SOUND: off.
NEGATIVES: no camera wobble, no speed ramp whoosh, no neon glow, no face/hand warping.
```
파라미터: `model_id: kling3_0`, mode: pro, `aspect_ratio: 16:9`, `duration: 4`, sound: off

---

**예시 2 — 감정씬 CU (LCD_DOCUMENTARY_GAZE)**

키프레임 (nano banana pro):
> SUBJECT: 여성 얼굴, 눈물 고임, 40대 한국인 / SETTING: 실내, 창가, 흐린 오후 빛 / MOOD: 비탄과 억제의 경계 / LIGHT: 창문 diffuse 자연광, 왼쪽, 부드러운 그림자 / COMPOSITION: CU, Eye Level, 얼굴이 프레임 중심, 약한 headroom / LENS: 85mm, 배경 완전 보케 / PALETTE: desaturated 따뜻한 회색, 피부톤, 창문의 white / NOT: 과도한 글리터 눈물, 극적인 rim light, 배우 워핑

모션 프롬프트:
```
CAMERA: imperceptibly slow push in, lens barely moves across the full 8 seconds.
SUBJECT MOTION: the character's facial muscles make the smallest possible movements — a barely visible swallow, eyelids heavy.
ATMOSPHERE: dust motes drift slowly through the window light.
PACING: real-time, patient. Let the actor breathe.
SOUND: off.
NEGATIVES: no speed ramps, no sudden zooms, no glowing particles, no warping skin.
```
파라미터: `model_id: minimax_hailuo`, `resolution: 1080`, `duration: 10`, sound: off

---

## 제약

- 키프레임 먼저. 항상. 비디오 프롬프트는 장면을 다시 묘사하지 않는다.
- 디렉션 사인오프 없이는 생성 금지. 탐색 패스 없이 히어로 모델 투입 금지.
- 액션씬은 사운드가 필요하지 않으면 sound: off.
- 루프는 end_image가 있는 모델 사용. 없으면 seam이 점프한다.
- 모든 NOT 목록을 네거티브에 — 디렉션 스펙에 쓰고 네거티브에 안 넣으면 모델이 무시한다.
- per-model duration/resolution/aspect 제약 준수 — 범위 외 파라미터는 서버가 거부한다.
