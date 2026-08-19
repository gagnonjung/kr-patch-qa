# kr-patch-qa

레트로 게임 **한국어 패치/한글화 프로젝트에서 공통으로 적용할 QA 및 런타임 안전 기준**을 정리한 저장소입니다.

번역 문장의 자연스러움만 검사하는 규칙집이 아니라, 실제 게임에서 한글화가 안전하게 동작하도록 **텍스트 → 글리프/폰트 → 그래픽 → 포인터/오프셋 → 압축/아카이브 → ROM·디스크 물리 배치 → 최종 빌드 → 런타임 스모크 테스트**까지 하나의 검증 흐름으로 다룹니다.

> **핵심 원칙:** 정적 검사 PASS는 런타임 PASS가 아닙니다.
> 최종 배포 후보는 실제 게임의 소비 경로와 런타임에서 별도로 검증해야 합니다.

## 기준 문서

공통 규약의 원문은 다음 문서입니다.

- [`LOCALIZATION_QA_STANDARD.md`](LOCALIZATION_QA_STANDARD.md)

프로젝트별 규칙이 이 문서보다 더 엄격하면 **더 엄격한 프로젝트 규칙을 우선**합니다. 다만 원본 보존, 포인터/오프셋/크기, 글리프 커버리지, 압축/할당 스팬, 물리 레이아웃과 같은 런타임 안전 조건은 검증 근거 없이 완화하지 않습니다.

## 무엇을 검사하는가

`kr-patch-qa`는 다음 범위를 공통 QA 대상으로 봅니다.

- 한국어 문장부호, 호격 쉼표, 조사, 띄어쓰기, 어절/형태소, 인물별 존대·반말, 용어 통일
- 실제 렌더러 폭을 기준으로 한 줄바꿈과 표시 영역 검증
- 번역문에 필요한 글리프의 매핑 및 활성 슬롯 커버리지
- 폰트/텍스처/팔레트/CLUT/타일 구조와 보호 영역 보존
- 고정 길이 레코드, 문자열 스팬, 포인터, 오프셋, 정렬, 패딩, 크기 필드 검증
- 압축 스트림 크기와 실제 할당 스팬/섹터 예약을 분리한 검증
- 아카이브, 파일 테이블, FST/LBA 등 물리 배치 회귀 검사
- 최종 이미지에 실제로 기록된 바이트를 다시 읽는 readback 검증
- 프리징/크래시 발생 시 last-known-good ↔ first-known-bad 기준의 원인 격리
- RC 빌드, 런타임 스모크, 캐노니컬 승격, 패치 패키징, 릴리스 단계 구분

## Agent Skill 설치

이 저장소는 문서만 읽어 사용하는 규약이면서, **Codex와 Claude Code에서 설치할 수 있는 Agent Skill**도 함께 제공합니다.

대표 사용 문구는 다음처럼 잡았습니다.

```text
한글화 QA 스킬 설치해줘
한글화 QA 스킬 적용해줘
```

설치 후에는 아래 표현도 같은 QA 작업으로 인식하도록 스킬 메타데이터에 포함되어 있습니다.

- `한글화 QA`
- `한글 패치 QA`
- `한글 패치 검수`
- `배포 전 QA`
- `RC 검증`
- `프리징 회귀 검수`

### Codex

안정판은 `main` 채널을 사용합니다.

```sh
codex plugin marketplace add gagnonjung/kr-patch-qa --ref main
codex plugin add kr-patch-qa@kr-patch-qa
```

설치가 끝나면 **새 스레드/세션**에서 다음처럼 요청합니다.

```text
한글화 QA 스킬 적용해줘.
이 프로젝트의 현재 RC를 공통 QA 규약으로 검증해줘.
```

같은 저장소의 새 버전으로 갱신할 때는 marketplace를 최신 `main`으로 갱신한 뒤 플러그인을 업데이트하거나 재설치하고 새 스레드에서 적용합니다.

### Claude Code

```text
/plugin marketplace add gagnonjung/kr-patch-qa@main
/plugin install kr-patch-qa@kr-patch-qa
```

설치 후 플러그인을 reload하거나 새 세션을 열고 다음처럼 요청합니다.

```text
한글화 QA 스킬 적용해줘.
이 한글 패치를 배포 전 QA 기준으로 검수해줘.
```

### create-kr-patch와 함께 설치할 때

`create-kr-patch`와 `kr-patch-qa`는 경쟁하는 스킬이 아니라 역할이 다릅니다.

```text
create-kr-patch  = 한글 패치를 조사·설계·구현하는 방법론
kr-patch-qa      = 결과를 한국어 QA·바이너리·런타임 기준으로 검증하는 공통 규약
```

둘 다 설치된 상태에서는 예를 들어 다음처럼 요청할 수 있습니다.

```text
create-kr-patch 기준으로 작업을 이어가고,
한글화 QA 스킬도 적용해서 RC_READBACK_QA까지 검증해줘.
```

런타임 확인이 필요한 단계에서 `emucap`도 연결되어 있다면 같은 작업 흐름에서 `RUNTIME_SMOKE` 증거까지 이어서 수집할 수 있습니다.

## 빠른 적용법

### 1. 프로젝트에서 공통 규약을 참조한다

각 한글화 프로젝트의 `AGENTS.md`, `HANDOFF.md`, `WORKLOG.md` 또는 작업 지침에 이 저장소를 공통 규약으로 연결합니다.

예시:

```md
## 공통 한글화 QA

공통 규약:
https://github.com/gagnonjung/kr-patch-qa/blob/main/LOCALIZATION_QA_STANDARD.md

작업 시작 전 공통 규약을 읽고 적용한다.
프로젝트별 규칙이 더 엄격하면 프로젝트 규칙을 우선한다.
정적 QA만으로 완료 처리하지 않고 런타임 스모크 결과를 별도로 기록한다.
```

규약을 프로젝트 내부에 복사해 사용할 수도 있지만, 여러 프로젝트에서 동일 기준을 유지하려면 **공통 저장소를 기준점으로 두고 프로젝트에는 프로젝트 고유 예외만 기록하는 방식**을 권장합니다.

### 2. 작업 시작 시 기준선을 고정한다

최소한 다음을 확인한 뒤 수정합니다.

- 지원하는 원본 ROM/ISO/BIN/CUE/archive 리비전과 강한 해시
- 현재 last-known-good 빌드 또는 배포 기준선
- 수정 대상 파일/논리 엔트리/실제 소비 경로
- 문자열 길이·레코드·포인터·압축·그래픽 등 변경되는 구조
- 프로젝트별 용어/화자/말투/표시 영역 규칙

지원 원본의 해시가 다르면 다른 리비전이라고 추정해 계속 진행하지 말고, 먼저 구조가 같은지 별도로 검증합니다. 원본은 불변으로 취급하고 수정된 산출물은 원본 또는 검증된 기준 빌드에서 재현 가능해야 합니다.

### 3. 사람이 편집하는 원본과 게임 소비 바이트를 분리한다

가능하면 다음 두 계층을 구분합니다.

```text
사람이 편집하는 번역 원본
  예: UTF-8 TSV / JSON / kor_edit
          ↓ 검증된 변환
게임이 실제 소비하는 확정 입력
  예: custom encoding / Shift-JIS code space / packed binary / output
```

원문, 레코드 식별자, 제어 토큰, 화자/음성/타이밍 같은 구조 필드는 수정 가능한 번역문과 분리해 보존하는 것이 좋습니다.

예를 들어 [`dc_sa2_ko`](https://github.com/gagnonjung/dc_sa2_ko)는 `messages.tsv`에서 `original`과 메시지 식별자·제어값을 보존하고 `translation`만 실제 한국어 입력으로 사용합니다. [`ps2_giogio_ko`](https://github.com/gagnonjung/ps2_giogio_ko)는 사람이 편집하는 `kor_edit`과 게임이 소비하는 확정 바이트 `output`을 분리합니다.

이 구분은 번역 원문을 사람이 검토하기 쉽게 유지하면서도, 최종 빌드 입력을 바이트 단위로 재현하고 검증하는 데 도움이 됩니다.

### 4. QA 단계를 분리해 기록한다

기본 단계는 다음과 같습니다.

```text
SOURCE_QA
  ↓
STATIC_BINARY_QA
  ↓
RC_BUILD
  ↓
RC_READBACK_QA
  ↓
RUNTIME_SMOKE
  ↓
CANONICAL_PROMOTION
  ↓
PATCH_PACKAGE
  ↓
RELEASE
```

각 단계의 PASS는 **그 단계의 범위만 의미**합니다.

예를 들어 텍스트 전수 검사와 바이너리 정적 검증이 모두 통과해도, 게임에서 해당 메뉴나 이벤트에 진입했을 때 프리징이 발생한다면 `RUNTIME_SMOKE`는 FAIL이며 캐노니컬 빌드로 승격할 수 없습니다.

### 5. 하나의 주 빌드 경로를 만든다

가능하면 최종 배포 산출물을 만드는 **단일 canonical pipeline/entry point**를 둡니다.

이 경로가 다음을 한 번에 책임지게 하는 것이 좋습니다.

1. 지원 원본 해시 검증
2. 원본 구조와 엔트리 모집단 검증
3. 승인된 번역/그래픽/바이너리 델타 적용
4. 슬롯/포인터/압축/정렬/크기 검증
5. 최종 아카이브 또는 이미지 직렬화
6. 결과 readback 및 기준선 비교

[`ps2_giogio_ko`](https://github.com/gagnonjung/ps2_giogio_ko)는 `tools/giogio_pipeline.py`를 유일한 주 진입점으로 두고 원본 ISO 확인부터 AFS 재조립, 기준선 비교, ISO 재기록, 최종 재추출 검증까지 연결합니다.

개별 도구가 각각 성공했다는 사실을 합산해서 “최종 빌드가 검증되었다”고 판정하지 않는 것이 중요합니다.

### 6. 수정 후 최종 산출물을 다시 읽는다

빌드 스크립트가 성공했다는 사실만으로 통과시키지 않습니다.

- 최종 ROM/디스크 이미지의 변경 영역을 다시 읽어 의도한 값과 일치하는지 확인
- 보호 영역이 원본 또는 승인된 기준과 같은지 확인
- 포인터/오프셋/크기/정렬/물리 배치에 설명되지 않은 변화가 없는지 확인
- overlap / out-of-range가 0인지 확인
- 출력 이미지의 크기와 해시 기록
- 고정 extent를 유지하는 구조라면 남는 공간의 패딩도 검증

`ps2_giogio_ko`처럼 **논리 아카이브를 먼저 검증하고 최종 ISO에 기록한 뒤 다시 추출해 동일 결과를 확인하는 방식**이 좋은 예입니다.

전체 ISO/ROM 해시가 과거 빌드와 다르더라도 곧바로 실패로 볼 수는 없습니다. 빌드 방식이 달라 전체 이미지 해시가 달라지는 경우에는, 실제 게임이 소비하는 논리 파일·아카이브·실행 파일과 보호해야 할 물리 배치가 의도대로 동일한지 별도로 증명해야 합니다.

### 7. 원본의 물리 배치를 가능한 한 보존한다

구조를 완전히 증명하지 못한 상태에서는 “새 데이터가 더 작아졌으니 공간도 줄여도 된다”고 가정하지 않습니다.

- 기존 extent/span/sector reservation을 유지할 수 있으면 우선 유지
- 압축 스트림 크기와 실제 할당 크기를 별도 값으로 취급
- 남는 고정 영역은 원본 규칙에 맞게 패딩
- 다른 파일의 LBA/FST/offset을 불필요하게 이동하지 않음

`ps2_giogio_ko`는 원본 ISO의 기존 extent 안에 결과를 기록하고 남는 AFS 슬롯을 0으로 채워 기존 파일 배치와 부트 구조를 유지합니다. 이 패턴은 다른 고정 배치 디스크 프로젝트에서도 재사용할 수 있습니다.

### 8. 공개 저장소와 저작물 경계를 분리한다

공개 저장소에는 가능한 한 다음만 둡니다.

- 번역 원본 및 프로젝트 메타데이터
- 소스 코드 형태의 툴체인
- 재현 가능한 빌드/검증 스크립트
- 합법적으로 재배포 가능한 차분 데이터
- 원본 리비전 식별용 해시

원본 ROM/ISO, 패치된 전체 이미지, 원본에서 통째로 추출한 게임 자산, 생성된 실행 파일은 저장소에서 제외하는 것을 기본으로 합니다.

`dc_sa2_ko`는 source-only 툴체인을 추적하고 추출 PRS/언팩 바이너리/빌드 출력/패치 디스크를 제외하며, `ps2_giogio_ko`도 원본 ISO·전체 패치 ISO·원본 추출 AFS/PZZ/실행 파일을 공개 저장소에 넣지 않습니다.

## 함께 사용하는 프로젝트

`kr-patch-qa`는 단독 규약으로도 사용할 수 있지만, 아래 두 프로젝트와 함께 사용하면 전체 한글화 파이프라인의 역할이 명확해집니다.

### create-kr-patch — 한글 패치 제작 방법론

**저장소:** [mcpads/create-retro-game-kr-patch](https://github.com/mcpads/create-retro-game-kr-patch)

`create-kr-patch`는 레트로 게임 한글 패치의 **전체 제작 방법론을 담당하는 Agent Skill**입니다.

ROM/디스크 조사, 텍스트 엔진 역공학, 한글 폰트와 인코딩, 텍스트 추출/재삽입, 포인터, ASM 훅, 그래픽 텍스트, 압축, 빌드와 검증 등 **“어떻게 한글 패치를 만들 것인가”**를 다룹니다.

반면 `kr-patch-qa`는 실제 프로젝트에서 반복적으로 발견된 한국어 QA와 런타임 회귀 사례를 바탕으로 **“이 결과를 어떤 공통 기준으로 PASS/FAIL 판정할 것인가”**를 보강합니다.

#### Codex에서 설치

안정판은 `main` 채널을 사용합니다.

```sh
codex plugin marketplace add mcpads/create-retro-game-kr-patch --ref main
codex plugin add create-kr-patch@kr-patch
```

설치 후 레트로 게임 한글화 작업을 요청하면 `create-kr-patch`가 작업 영역에 맞는 전략/플랫폼 문서를 선택합니다. 프로젝트 작업 지침에서는 동시에 `kr-patch-qa`의 공통 규약을 참조하도록 두면 됩니다.

#### Claude Code에서 설치

```text
/plugin marketplace add mcpads/create-retro-game-kr-patch@main
/plugin install create-kr-patch@kr-patch
```

#### 권장 역할 분담

```text
create-kr-patch
  └─ 조사 / 역공학 / 폰트 / 인코딩 / 추출 / 재삽입 / 빌드 방법론

kr-patch-qa
  └─ 한국어 QA / 런타임 안전 / 회귀 기준 / PASS 단계 / 캐노니컬 승격 규칙
```

두 문서에서 같은 일반 규칙을 중복 유지하기보다는, 기존 `create-kr-patch`가 이미 소유한 방법론은 그쪽을 따르고 **한국어 패치 QA에 특화된 추가 기준만 `kr-patch-qa`에서 공통화**하는 것을 원칙으로 합니다.

### emucap — 실제 에뮬레이터 런타임 검증

**저장소:** [gagnonjung/emucap](https://github.com/gagnonjung/emucap)

`emucap`은 실행 중인 에뮬레이터의 메모리·상태·화면을 AI 에이전트가 읽고 제어할 수 있게 하는 **레트로 게임 패치 디버깅용 MCP 인프라**입니다.

`kr-patch-qa`가 `RUNTIME_SMOKE`에서 요구하는 실제 실행 증거를 확보하거나, 한글화 이후 발생한 프리징·그래픽 깨짐·잘못된 상태 전이의 원인을 좁힐 때 사용할 수 있습니다.

emucap은 크게 두 계층을 제공합니다.

- **Control MCP**: 에뮬레이터 실행, 화면/메모리/상태 관찰, 입력, 세이브스테이트, 브레이크포인트, 회귀 분석
- **Tracking MCP**: 실험 run, gate, metric, intervention 등을 `.emucap/` 원장에 기록하고 비교

#### 설치/등록

전체 설치 절차는 emucap 저장소의 [한국어 README](https://github.com/gagnonjung/emucap/blob/main/README.ko.md)를 따릅니다.

Codex를 Windows에서 사용할 경우 emucap을 빌드한 뒤 PowerShell에서 다음 등록 스크립트를 사용합니다.

```powershell
tools/register-codex-mcp.ps1
```

등록 후 새 에이전트 세션에서 Control MCP와 Tracking MCP의 `bootstrap`이 모두 보이는지 확인합니다.

#### 한글화 QA에서의 기본 사용 흐름

```text
1. emucap bootstrap
2. launch_plan으로 대상 게임/시스템의 실행 인자 검증
3. launch
4. status로 실제 연결 및 runtime identity 확인
5. get_rom_info로 ROM identity 확인
6. 필요한 화면/메모리/상태/입력/브레이크포인트 검증
7. Tracking MCP에 run / gate / metric / intervention 기록
8. kr-patch-qa의 RUNTIME_SMOKE 결과로 반영
```

프리징이나 회귀를 조사할 때는 단순히 “에뮬레이터가 멈췄다”만 기록하지 않고, `kr-patch-qa`의 재현 규칙에 맞춰 **정확한 화면/진입 경로/마지막 정상 상태/입력/last-known-good/first-known-bad**를 남긴 뒤 emucap으로 원인 계층을 좁힙니다.

## 세 프로젝트를 함께 쓰는 권장 흐름

```text
┌──────────────────────────────────────────────┐
│ create-kr-patch                              │
│ 한글 패치의 조사·설계·구현 방법론            │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│ kr-patch-qa                                  │
│ 한국어 품질 + 바이너리/런타임 안전 QA 기준   │
└──────────────────────┬───────────────────────┘
                       │ RC_READBACK_QA 통과
                       ▼
┌──────────────────────────────────────────────┐
│ emucap                                       │
│ 실제 에뮬레이터에서 RUNTIME_SMOKE / 회귀 분석 │
└──────────────────────┬───────────────────────┘
                       │ PASS
                       ▼
             CANONICAL_PROMOTION
                       │
                       ▼
              PATCH_PACKAGE / RELEASE
```

실제 작업에서는 세 프로젝트가 직렬로만 움직이는 것은 아닙니다. 예를 들어 emucap에서 런타임 프리징을 발견하면 다시 `create-kr-patch`의 압축/재삽입/폰트/그래픽/런타임 자산 판단 영역으로 돌아가 원인을 수정하고, `kr-patch-qa` 기준으로 다시 정적 검증과 readback을 거친 뒤 같은 런타임 경로를 재검증합니다.

## 검증에 참고한 플랫폼과 게임

현재 공통 QA 규약을 정리하고 보강할 때 다음 실제 한글화 프로젝트의 구조·빌드·회귀 사례를 참고했습니다.

| 플랫폼 | 게임 |
|---|---|
| Dreamcast | Sonic Adventure 2 |
| GameCube | Metal Gear Solid: The Twin Snakes |
| PlayStation 2 | JoJo no Kimyou na Bouken: Ougon no Kaze |
| Sega Saturn | Slayers Royal |
| PlayStation | Suzuki Bakuhatsu |
| Nintendo 64 | The Legend of Zelda: Majora's Mask |

이 목록은 해당 게임의 구현 세부를 다른 플랫폼에 그대로 적용한다는 의미가 아닙니다. 공통 규칙은 실제 저장·주소 지정·렌더링·런타임 소비 구조가 같은 경우에만 재사용하고, 플랫폼별 세부 검증은 해당 구조에 맞게 치환합니다.

## 참고할 만한 실제 프로젝트

### Sonic Adventure 2 Korean Translation

- 저장소: [gagnonjung/dc_sa2_ko](https://github.com/gagnonjung/dc_sa2_ko)

공통화할 만한 패턴:

- 일본어 원문과 한국어 번역을 구조화된 TSV에서 분리
- 화자 및 게임 제어값을 번역 검토 정보로 함께 유지
- 메시지 식별자와 원문 필드는 수정 금지 대상으로 취급
- 추출 게임 리소스와 빌드 결과는 Git에서 제외
- 텍스트/폰트/이미지/자막 도구는 source-only 툴체인으로 추적
- 정적 인코딩/압축 크기 검증과 실제 인게임 검토를 별도 단계로 구분

### Metal Gear Solid: The Twin Snakes — GameCube

공통화할 만한 패턴:

- GameCube FST의 offset/size와 비대상 파일 물리 배치를 최종 이미지에서 다시 검증
- `stage.dat`처럼 압축 스트림 크기와 실제 allocation/sector span이 분리된 자산은 재압축 결과가 작아졌다는 이유만으로 known-good allocation floor를 축소하지 않음
- 텍스트·폰트·그래픽 자체가 정상이어도 압축 descriptor나 물리 통합 변경만으로 특정 장면 전환에서 프리징이 발생할 수 있으므로 계층별로 회귀를 분리
- 런타임 PASS가 확인된 마지막 정상 빌드와 문제 첫 빌드의 해시를 고정해 원인 범위를 좁힘
- 미션 로그·무전·컷신·장소명처럼 같은 표현이 여러 소비 경로에 존재할 수 있으므로 논리 문자열 하나가 아니라 실제 physical consumer를 전수 확인
- 여러 디스크로 구성된 게임은 디스크별 정적/readback/runtime PASS를 모두 확보한 뒤에만 최종 패치 패키지를 만듦

### 죠죠의 기묘한 모험 황금의 바람 — 한글화 재현 프레임워크

- 저장소: [gagnonjung/ps2_giogio_ko](https://github.com/gagnonjung/ps2_giogio_ko)

공통화할 만한 패턴:

- 원본 ISO MD5/SHA-256을 먼저 검증하고 다른 리비전은 자동 추정하지 않음
- 하나의 주 pipeline이 추출/재조립/검증을 책임짐
- 사람이 편집하는 `kor_edit`과 게임 소비용 확정 바이트 `output`을 분리
- 논리 아카이브와 실행 파일의 known-good 해시를 별도로 유지
- 고정 extent 안에서만 다시 기록하고 잔여 슬롯을 0으로 패딩
- 최종 ISO에서 변경 파일을 다시 읽어 post-write 결과를 검증
- 새 override는 기존 배포 해시 일치 대신 구조/슬롯/기록 안전성으로 검증
- 자동 QA가 보장하는 범위와 PCSX2/실기에서 확인해야 할 화면·진행·음성 동기 범위를 명시적으로 분리

### Slayers Royal — Sega Saturn

- 저장소: [gagnonjung/ss_slayersroyale_ko](https://github.com/gagnonjung/ss_slayersroyale_ko)

공통화할 만한 패턴:

- 정적 추정으로 잡았던 폰트 규격이 실기 증거와 충돌하면 기존 가정을 폐기하고 런타임 기준선을 다시 확정
- 글리프 저장 포맷과 실제 렌더러가 사용하는 표시 포맷을 분리해 추적
- 제어코드의 종류·순서·개수를 보호하고 줄바꿈만 허용된 범위에서 이동
- 동적 치환 토큰의 폭은 정적 추정만으로 PASS 처리하지 않고 런타임 검토 대상으로 유지
- 자동 QA의 `passed`와 사람의 번역/레이아웃 승인 범위를 분리
- MODE1/2352 변경 섹터는 EDC/ECC를 재생성하고 재추출 byte equality로 검증
- 실행 결과를 반드시 정확한 빌드 해시와 연결

### Suzuki Bakuhatsu — PlayStation

- 저장소: [gagnonjung/ps1_suzukibakuhatsu](https://github.com/gagnonjung/ps1_suzukibakuhatsu)

공통화할 만한 패턴:

- Mode2/2352와 XA/STR처럼 섹터 배치에 민감한 자산은 일반 ISO 재빌드 성공과 런타임 안전을 별도로 판단
- same-size 교체에서는 원본 LBA/sector/subheader를 보존하는 in-place 패치를 우선 검토
- 실제 VRAM/RAM 캡처의 바이트를 전체 자산에서 역검색해 진짜 런타임 소비 자산을 특정
- 시각적으로 같은 폰트/atlas 복제본을 진짜 소비자로 오인하지 않도록 runtime ground truth를 사용
- 화면/스테이지별 subset font에서 주소 가능한 슬롯 수와 물리 font blob의 tile 수를 별도 capacity로 관리
- 공유 glyph slot을 사용하는 다른 화면의 문자까지 합쳐 충돌을 검사
- 디코더 규칙이 틀렸다고 밝혀지면 불변 원본에서 전체 재추출하고 기존 번역 이관 audit과 idempotence를 검증
- 전체 모집단을 먼저 열거해 dialogue/empty/duplicate/unclassified 상태를 고정하고 누락을 실패 처리

### The Legend of Zelda: Majora's Mask — Nintendo 64

- 저장소: [gagnonjung/n64_zeldamm_ko](https://github.com/gagnonjung/n64_zeldamm_ko)

공통화할 만한 패턴:

- 카트리지 기반에서도 ROM byte order, DMA table, VROM/ROM 경계, raw/compressed 상태를 분리해 검증
- 메시지 추출/컴파일러의 blank-translation round trip을 byte-identical로 먼저 증명
- 비텍스트 control/button token의 종류·순서·개수를 컴파일 단계에서 강제
- 실제 렌더러의 폭 계산식을 모델링해 문자 수가 아닌 pixel width로 layout QA 수행
- source-used glyph를 보존하고 실제 미사용 슬롯만 한국어 destination code로 할당
- 폰트 capacity를 전체 slot 공급량과 현재 corpus demand로 함께 보고
- Yaz0 같은 압축 데이터는 decoded byte length와 stored compressed bytes를 분리하고 즉시 decompression round trip 검증
- 최종 ROM에서 변경 segment를 다시 추출·재파싱해 source→build→ROM→extract 경로를 닫음
- 에뮬레이터를 실행하지 않은 정적 빌드는 `runtime smoke pending`으로 명확히 유지

이 저장소들은 `kr-patch-qa`의 규칙을 그대로 구현한 템플릿이 아니라, **실제 한글화 프로젝트에서 재사용 가능한 안전 패턴을 확인할 수 있는 사례**로 봅니다.

Dreamcast/GameCube/PlayStation 2/Sega Saturn/PlayStation/Nintendo 64 사례에는 파일시스템·아카이브·압축·섹터 또는 DMA 재구성처럼 플랫폼별 구조가 섞여 있습니다. 여기서 얻은 `ISO extent`, `LBA`, 대형 archive descriptor, sector allocation, DMA table 같은 세부 규칙을 존재하지 않는 다른 플랫폼에 그대로 요구하지 않습니다.

`kr-patch-qa`는 비트 세대보다 **실제 저장·주소 지정·렌더링 구조**를 기준으로 적용합니다.

- 전 세대 공통: 원본/리비전 고정, 원문/번역 QA, byte/display/glyph capacity, final readback, 회귀 추적, runtime smoke, canonical 승격
- 디스크/파일시스템 기반: extent/LBA/FST/sector/archive descriptor 등 해당 구조 검증
- 8/16비트 카트리지 기반: ROM bank mapping, pointer width, bank crossing, mapper, free-space reachability, tile/VRAM/palette budget 등 해당 구조 검증
- N64처럼 카트리지 기반이지만 DMA/압축/segmented pointer를 많이 쓰는 시스템은 그 실제 소비 구조에 맞춰 별도 적용

즉 **없는 구조를 검사하는 것이 아니라, 각 플랫폼에서 같은 안전 목적을 담당하는 실제 경계를 검증**합니다.

## 예시: 프리징이 발생한 한글화 빌드

상태창 진입 시 한글화 빌드만 프리징된다고 가정합니다.

1. `kr-patch-qa` 기준으로 last-known-good와 first-known-bad 빌드를 고정합니다.
2. 텍스트, 폰트/글리프, 그래픽, 압축/아카이브, 물리 통합 중 독립적으로 분리 가능한 계층을 나눕니다.
3. `create-kr-patch`의 해당 판단 영역을 이용해 포인터/크기/소비 경로/압축 descriptor/할당 스팬 등의 구조를 확인합니다.
4. 새 RC를 만들고 `RC_READBACK_QA`에서 실제 최종 이미지의 변경 영역과 보호 영역을 검증합니다.
5. emucap으로 같은 상태창 진입 경로를 재현하고 화면·메모리·상태 전이를 확인합니다.
6. 런타임 PASS가 확인된 정확한 RC만 캐노니컬로 승격합니다.

이 과정에서 텍스트가 화면에 정상 출력된다는 이유만으로 관련 바이너리 구조까지 안전하다고 가정하지 않습니다.

## 새 규칙을 추가하는 기준

이 저장소는 실제 프로젝트에서 발견된 문제를 무조건 모두 모으는 로그가 아닙니다.

새 규칙은 다음 조건을 만족할 때 공통 규약으로 승격하는 것을 권장합니다.

- 하나의 특정 게임에만 해당하지 않고 여러 한글화 프로젝트에 재사용 가능할 것
- 원인과 실패 조건이 충분히 확인되어 있을 것
- 기존 `create-kr-patch` 방법론이 이미 동일한 규칙을 소유하고 있지 않을 것
- 기존 규칙보다 더 엄격한 안전 조건을 추가한다면 그 필요성이 설명될 것
- 정적 QA와 런타임 QA의 증거 범위를 혼동하지 않을 것

게임 하나의 특수한 포인터 값, 특정 주소, 특정 압축 수치 같은 값은 이 저장소에 일반 규칙으로 넣지 않고 해당 프로젝트 기록에 남깁니다.

## 권장 결과 보고

QA 결과에는 최소한 다음 상태를 서로 분리해서 남기는 것이 좋습니다.

```text
SOURCE_QA             PASS / FAIL / NOT RUN
STATIC_BINARY_QA      PASS / FAIL / NOT RUN
RC_BUILD              PASS / FAIL
RC_READBACK_QA        PASS / FAIL / NOT RUN
RUNTIME_SMOKE         PASS / FAIL / NOT RUN
CANONICAL_PROMOTION   YES / NO
PATCH_PACKAGE         CREATED / NOT CREATED
```

가능하면 다음 정보도 함께 기록합니다.

- 원본/출력 식별자와 해시, 크기
- 변경 논리 엔트리 수와 실제 물리 소비자 수
- byte/block overflow 수
- 누락 글리프와 active-slot regression 수
- 포인터/오프셋/크기/물리 배치 diff 수
- overlap / out-of-range 수
- 보호 영역 검증 결과
- last-known-good / first-known-bad 식별값
- 실제 런타임에서 확인한 경로와 미확인 범위

## 목적

`kr-patch-qa`의 목표는 “모든 게임에 같은 구현을 강제하는 것”이 아닙니다.

플랫폼과 게임 구조는 서로 다르지만, **원본을 보존하고, 변경 범위를 증명하고, 최종 산출물을 다시 읽고, 실제 런타임에서 확인한 범위만 PASS라고 부르는 습관**은 공통화할 수 있습니다.

실제 프로젝트에서 새로운 문제가 발견되면 먼저 해당 프로젝트에서 원인을 증명한 뒤, 여러 프로젝트에 재사용할 수 있는 규칙일 때만 이 공통 규약으로 승격합니다.
