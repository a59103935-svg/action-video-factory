# Action Video Factory

**줄콘티 → 스토리보드 콘티 → Higgsfield 영상 제작** 통합 파이프라인.

액션+감정씬을 위한 표준 영화 문법 완전 적용 버전.
`video-factory` (bpainter)의 파이프라인 뼈대 + `inference-sh/storyboard-creation`의 영화 문법 기준을 통합하고,
중간 스토리보드 스킬을 액션 콘티 전용으로 완전히 재작성.

---

## 파이프라인

```
줄콘티(대본)
    │
    ▼
[action-video-factory-concept]
  3~5개 방향성 기획. 카메라 전략, 거장 레퍼런스, 실현 가능성.
    │
    ▼ 선택된 콘셉트
[action-video-factory-storyboard]  ★ 핵심 오버라이드
  샷-바이-샷 콘티. 영화 문법 완전 적용.
  + 로케이션 사진 분석 (선택)
    │
    ▼ 20_Storyboard.md
[action-video-factory-creative-direction]
  전체 룩 잠금. 레지스터/빛/컬러/그레인/STYLE 일관성.
    │
    ▼ 30_Direction.md
[action-video-factory-higgsfield-prompt]
  키프레임 프롬프트 + 모션 프롬프트 + Higgsfield MCP 실행.
    │
    ▼ 생성된 클립
```

---

## 스킬 구성

| 스킬 | 역할 | 원본 대비 |
|---|---|---|
| `action-video-factory-create-flow` | 마스터 오케스트레이터 | video-factory-create-flow 기반, 액션 파이프라인으로 조정 |
| `action-video-factory-concept` | 방향성 기획 (3~5개) | video-factory-concept 기반, 거장 레퍼런스 통합 |
| `action-video-factory-storyboard` | **액션+감정 전용 콘티** | **완전 재작성** — inference-sh 문법 + 거장 라이브러리 + 로케이션 분석 |
| `action-video-factory-creative-direction` | 룩 잠금 | video-factory-creative-direction 기반, 액션 레지스터 추가 |
| `action-video-factory-higgsfield-prompt` | 생성 엔진 | 모델 라우팅 업데이트 (kling 3.0 기본, minimax 감정씬) |

---

## 레퍼런스 파일

| 파일 | 내용 |
|---|---|
| `references/action-video-foundations.md` | 파이프라인, 샷 타입(ECU~EWS), 앵글, 무빙, 연속성 룰, 렌즈 가이드, 모델 라우팅 |
| `references/action-filmmaker-library.md` | 거장 STYLE 프리셋 라이브러리 (박찬욱, 류승완, 봉준호 등 + PTA, Fincher, Nolan 등) |

---

## action-video-factory-storyboard 핵심 기능

### 영화 문법 엔진
- 샷 사이즈 8종: ECU / CU / MCU / MS / MLS / LS / WS / EWS
- 카메라 앵글 �ܢ�: Eye Level / High / Low / Bird's Eye / Worm's Eye / Dutch / OTS
- 카메라 무빙 9종: Pan / Tilt / Dolly / Truck / Crane / Zoom / Steadicam / Handheld / Static
- 연속성 규칙 자동 체크: 180도룰, 매치온액션, 아이라인매치, 스크린디렉션

### 거장 기법 라이브러리 (STYLE 프리셋)
씬당 1~2개로 절제. 과잉 사용 방지 내장.

**한국 감독**: 박찬욱(PCW_SYMMETRY, PCW_LONG_TAKE_FIGHT 등), 박훈정(PHJ_SLOW_IMPACT 등), 류승완(RSW_OVERHEAD_FIGHT 등), 허진호(HJH_SHALLOW_LINGER 등), 이창동(LCD_DOCUMENTARY_GAZE 등), 봉준호(BJH_GENRE_PIVOT 등)

**해외 감독**: PTA(PTA_ONER), Nolan(CN_IMAX_LOW), Fincher(DF_COLD_PUSH), Cuarón(AC_CHAOS_ONER), Raimi(SR_CRASH_ZOOM)

**특수 기법**: SPEC_RACK_FOCUS, SPEC_360_ORBIT, SPEC_SMASH_CUT, SPEC_MATCH_CUT_EMOTION, SPEC_SPLIT_DIOPTER 등

### 로케이션 사진 분석
사진 입력 → 공간 구조, 광원, 카메라 포지션 제안, 블로킹 활용 요소 → `20_Location_Analysis.md`

### 출력: 그림콘티 패키지
- `20_Storyboard.md`: 텍스트 스토리보드 (샷마다 사이즈/앵글/렌즈/무빙/블로킹/STYLE/키프레임 프롬프트)
- `20_Storyboard_Visual/`: nano banana pro로 생성한 패널 이미지
- 패널 어노테이션 포맷 (씬번호, 샷타입, 렌즈, STYLE, 액션, 대사, SFX, 트랜지션)
- ASCII 샷 스트립 (전체 흐름 시각화)

---

## 모델 라우팅 요약

| 용도 | 모델 |
|---|---|
| 멀티샷 액션/격투 | kling3_0 mode pro |
| 분위기/배경/루프 | seedance_2_0 |
| 감정씬 (자연스러운 얼굴) | minimax_hailuo |
| 히어로샷 에스컬레이션 | cinematic_studio_3_0 / veo3_1 |
| 탐색 패스 | veo3_1_lite 720p |
| 키프레임 (기본) | nano banana pro |
| 키프레임 (텍스트 포함) | chatgpt image 2 |

---

## 설치

### Claude Code (CLI)

```bash
# 이 디렉토리를 Claude Code 플러그인 경로에 설치
# 예: ~/.claude/plugins/ 또는 프로젝트 .claude/plugins/ 하위

# 옵션 A — 글로벌 설치
cp -r action-video-factory ~/.claude/plugins/

# 옵션 B — 프로젝트 로컬 설치
cp -r action-video-factory .claude/plugins/
```

### Claude Cowork (데스크탑 앱)

```bash
# Cowork 스킬 디렉토리에 복사
# 위치는 앱 설정 > 스킬 디렉토리에서 확인
cp -r action-video-factory /path/to/cowork/skills/
```

### 사용 방법

설치 후 대화에서 다음과 같이 시작:

```
# 전체 파이프라인
"action-video-factory 실행해줘. 줄콘티: [씬 설명]"

# 단일 단계
"action-video-factory-storyboard: [콘셉트 내용]"
"action-video-factory-higgsfield-prompt: [스토리보드 내용]"
```

---

## 의존성

- **Higgsfield MCP**: 클립 생성에 필요. Claude Code/Cowork에서 Higgsfield MCP 플러그인 설치 필요.
- **인터넷 접속**: Higgsfield API 호출용.
- **크레딧**: Higgsfield 계정 크레딧 필요.

---

## 원본 저장소

- `video-factory` 파이프라인: [bpainter/composable-dxp-claude-marketplace](https://github.com/bpainter/composable-dxp-claude-marketplace)
- 영화 문법 기준: [inference-sh/skills](https://github.com/inference-sh/skills) (guides/video/storyboard-creation)
