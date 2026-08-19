# Game Localization Common QA & Runtime Safety Standard

Version: 1.0
Updated: 2026-08-19
Scope: `D:\Game - Localizing` 아래의 모든 게임 한글화 프로젝트

이 문서는 플랫폼·게임별 세부 규칙보다 상위에 두는 **공통 QA / 바이너리 안전 / 런타임 회귀 방지 규약**이다.
각 프로젝트의 `HANDOFF.md`, `WORKLOG.md`, `.ai-bridge/current-plan.md`, 프로젝트별 `AGENTS.md`는 이 문서를 기본 규약으로 참조한다.

프로젝트별로 명시된 규칙이 이 문서보다 더 엄격하면 프로젝트 규칙을 따른다. 다만 원본 보존, 포인터/오프셋/크기 안전성, 글리프 커버리지, 물리 레이아웃 검증과 같은 **런타임 안전 규칙은 명시적인 검증 근거 없이 완화하지 않는다.**

---

## 1. 최우선 원칙

1. **원본은 불변(immutable)으로 취급한다.**
   - 원본 ROM/ISO/BIN/CUE/archive를 직접 수정하지 않는다.
   - 빌드는 검증된 원본 또는 검증된 기준 빌드에서 재현 가능해야 한다.
   - 입력 파일의 SHA-256 또는 동등한 강한 해시를 기록한다.

2. **번역 품질보다 런타임 안정성이 먼저다.**
   - 텍스트가 더 자연스러워도 고정 블록/버퍼/포인터 한계를 넘으면 적용하지 않는다.
   - 구조를 이해하지 못한 상태에서 “길이가 조금 늘어도 되겠지”라고 가정하지 않는다.

3. **`PASS`의 범위를 정확히 쓴다.**
   - `정적 inventory PASS`는 런타임 PASS가 아니다.
   - `텍스트 QA PASS`는 폰트/그래픽/아카이브/ISO 통합 PASS가 아니다.
   - 최종 완료는 최소한 `source QA → binary QA → integrated build QA → runtime smoke`가 모두 통과해야 한다.

4. **한 번에 한 계층만 바꾼다.**
   - 텍스트, 폰트, 그래픽, 압축 아카이브, 실행 코드/후킹, ISO/ROM 물리 배치를 가능한 한 분리한다.
   - 프리징 회귀가 생기면 어떤 계층이 원인인지 즉시 역추적할 수 있어야 한다.

5. **검증된 상태를 보존한다.**
   - 이미 런타임 PASS한 데이터의 포맷·크기·오프셋·해시를 불필요하게 재생성하지 않는다.
   - unrelated 파일을 broad rebuild / broad stage / broad replace하지 않는다.

---

## 2. 번역 원문과 참고 자료의 우선순위

### 2.1 번역 원문

1. 실제 게임의 일본어/원어 데이터를 최우선 원문으로 삼는다.
2. 기존 한국어 초안은 개선 대상이지 원문이 아니다.
3. 영문 스크립트, 팬 위키, GameFAQs, 공략집 등은 다음 용도로만 사용한다.
   - 화자/청자 식별
   - 장면 순서
   - 고유명사 확인
   - 말투·관계성 메타데이터
   - 누락 문맥 보조
4. 별도 승인이 없는 한 참고용 영문 스크립트를 번역 원문으로 대체하지 않는다.

### 2.2 원문 불확실 시

- OCR이나 추측으로 번역하지 않는다.
- 바이너리 원문, 실제 화면, 다른 중복 소비자, 대사집을 교차 확인한다.
- 확정되지 않으면 `미확인` 플래그를 남긴다.

---

## 3. 화자·청자·어투 QA

### 3.1 화자 메타데이터

모든 주요 대화는 가능한 경우 다음 메타데이터를 가진다.

- speaker
- listener 또는 addressee
- scene / consumer / entry id
- register (반말/존댓말/군대식/공식적 표현 등)
- 직전·직후 문맥

### 3.2 말투 일관성

- 동일 인물이 동일 관계에서 존댓말/반말을 임의로 오가지 않게 한다.
- 등장인물 사전 또는 프로젝트별 voice policy가 있으면 반드시 따른다.
- 이름·계급·호칭은 단순 문자열 치환보다 **관계성**을 기준으로 판정한다.
- 중복 대사가 여러 CODEC/컷신/VOX/미션로그 소비처에 존재하면 모두 추적한다.

### 3.3 고유명사와 용어

- canonical terminology table을 둔다.
- 인명, 조직명, 기술명, 지명, 장비명, UI명은 소비처 전체에서 전수 검사한다.
- 한 번 수정한 용어가 legacy TM이나 duplicate table에서 다시 살아나지 않게 한다.

---

## 4. 한국어 문장 QA

### 4.1 기본 문장부호 정책

프로젝트별 예외가 없으면 다음을 기본으로 한다.

1. **일반 대화 레코드의 마지막 단일 마침표 `.`는 생략한다.**
   - 예: `알겠습니다.` → `알겠습니다`
   - 단, UI/설명문/시스템 메시지는 해당 게임의 원래 스타일을 따른다.

2. **한 레코드 안에 독립된 문장이 둘 이상이면 내부 마침표는 유지한다.**
   - 예: `알겠다. 서둘러라`

3. **`{br}` / 개행은 문장부호가 아니다.**
   - `알겠다{br}어서 가라`가 독립 문장 2개라면 `알겠다.{br}어서 가라`가 기본이다.

4. **호격에는 한국어 쉼표를 사용한다.**
   - `스네이크, 조심해`
   - `괜찮아요, 스네이크?`
   - 단, `안 돼! 메릴!!`, `나야! 스네이크`처럼 앞 감탄 뒤 이름을 **독립적으로 외치는 연출**은 예외로 둘 수 있다.

5. **주어를 호격으로 오판하지 않는다.**
   - `나오미 박사가…`, `나오미 박사는…`의 `나오미` 뒤에는 호격 쉼표를 넣지 않는다.

6. `?`, `!`, `…`, `...`는 발화 의도와 연출을 보존한다.
7. 일본어의 `、`나 띄어쓰기를 한국어 쉼표로 기계적으로 복제하지 않는다.
8. 감탄부호/물음표 다음에 새 한국어 구가 붙을 때는 적절한 공백, 개행 또는 문장 재구성을 적용한다.

### 4.2 띄어쓰기·형태소·조사

- 어절 단위 띄어쓰기를 우선한다.
- 조사(은/는, 이/가, 을/를, 과/와, 으로/로 등)는 고유명사 치환 이후 재검사한다.
- 형태소 분리는 화면 폭 때문에 꼭 필요한 경우에만 허용한다.
- 의미 없는 일본어식 직역 어순, 일본어식 존대, 중복 표현을 제거한다.
- 반복 단어, 탈락된 주어/목적어, 비문 후보를 기계 탐지 후 사람이 판정한다.

---

## 5. 줄바꿈·폭·화면 레이아웃 QA

1. **문자 수가 아니라 실제 렌더러 폭을 기준으로 검사한다.**
   - 가변폭/고정폭
   - 한글/ASCII 폭 차이
   - 아이콘·버튼·색상 코드·제어코드 소비 폭
   - 좌우 여백

2. 줄바꿈 우선순위:
   1. 의미 덩어리
   2. 어절 경계
   3. 자연스러운 형태소 경계
   4. 마지막 수단으로 문장 축약

3. 금지 또는 강한 경고 대상:
   - 조사만 다음 줄에 남음
   - 어미만 다음 줄에 남음
   - 인명/지명/장비명 분리
   - 숫자+단위, 버튼명, 약어 분리
   - 한글 조합을 시각적으로 부자연스럽게 절단
   - 숨겨진 4번째 줄 생성
   - 우측 정렬/중앙 배치 침범

4. 자동 줄바꿈은 후보 생성용이다. 최종 개행은 문맥과 실제 화면을 기준으로 승인한다.

---

## 6. 폰트·글리프 QA

1. TTF/OTF에 문자가 있다는 이유만으로 PASS하지 않는다.
2. 실제 게임에서 사용하는 다음 항목을 검증한다.
   - code → glyph slot mapping
   - static/dynamic glyph plan
   - slot count
   - glyph index 범위
   - 실제 ROM/ISO에 삽입된 2bpp/4bpp/texture glyph 데이터
3. 필요한 글리프가 비어 있거나 잘못된 슬롯을 참조하면 FAIL이다.
4. 이미 검증된 빌드보다 활성 글리프 수가 줄거나 슬롯이 0으로 소거되면 회귀로 간주한다.
5. 가능한 경우 보고서에 다음을 기록한다.
   - required glyph count
   - active physical slot count
   - missing index / missing character
   - font/plan/map SHA-256
6. **missing glyph = 0**을 배포 조건으로 한다.

---

## 7. 그래픽·팔레트 QA

텍스트 그래픽, 로고, UI, TIM/FCP/T texture 등은 다음을 보존한다.

- 원본 해상도/캔버스 크기
- bpp
- palette / CLUT 구조
- 투명 인덱스
- tile/texture entry count
- swizzle/tile order
- 원본이 요구하는 index vocabulary
- 압축 방식과 메타데이터

### 원칙

1. 원본 팔레트가 있는 자산은 가능하면 **source-native palette projection**을 사용한다.
2. 새 색상/인덱스를 임의로 늘리지 않는다.
3. 그래픽 하나를 바꾸면서 같은 컨테이너의 비대상 엔트리를 재인코딩하지 않는다.
4. 변경하지 않은 엔트리는 가능하면 byte-identical을 유지한다.
5. 작은 로고/로딩 로고/타이틀 변형처럼 비슷한 중복 자산도 전수 조사한다.

---

# 8. 프리징·크래시 방지 바이너리 안전 규약

이 절은 **배포 차단 규칙**이다.

## 8.1 고정 크기 우선

구조가 완전히 해명되지 않은 게임에서는 기본적으로:

- 원본 파일 크기 유지
- 레코드 크기 유지
- 블록 크기 유지
- 블록 시작 오프셋 유지
- 파일 시작 LBA/sector 유지
- entry count/order 유지

를 우선한다.

**포인터 재배치와 relocation은 구조가 증명된 경우에만 허용한다.**

## 8.2 텍스트 필드 / 레코드 / 블록 예산

각 텍스트 소비처에서 다음을 따로 검사한다.

- encoded byte length
- terminator 포함 여부
- record header/footer
- alignment (2/4/8/16/sector 등)
- text capacity
- glyph capacity
- block capacity

문자 수가 같아도 인코딩 바이트 수와 정렬 때문에 크기가 늘 수 있다.
예: 쉼표 1바이트 추가가 4바이트 정렬 경계를 넘겨 레코드를 4바이트 키울 수 있다.

**capacity 초과는 1바이트라도 FAIL이다.**

## 8.3 포인터·오프셋·크기 테이블

수정 후 반드시 확인한다.

- pointer table
- offset table
- size table
- record count
- archive index
- next-entry boundary
- end marker / sentinel
- checksum이 존재하면 checksum

한 항목의 길이가 변했는데 뒤 항목 포인터가 그대로면 프리징/크래시 가능성이 매우 높다.

## 8.4 압축 데이터

zlib, Yay0, LZ 계열 등 압축 컨테이너를 수정할 때:

1. 압축 해제 결과가 정상인지 검사한다.
2. descriptor의 compressed size / uncompressed size를 검사한다.
3. 할당 span/sector가 존재하는 포맷이면 실제 압축 크기와 별도로 검사한다.
4. **retail/reference보다 descriptor 또는 sector allocation이 줄어드는 것이 런타임에서 안전하다고 증명되지 않았다면 축소하지 않는다.**
5. 과거 정상 빌드의 allocation floor를 유지할 수 있으면 유지한다.
6. 모든 변경 블록에 대해 다시 압축 해제 테스트를 수행한다.

## 8.5 ISO / ROM 물리 레이아웃

GameCube FST, CD sector, PS1/PS2 LBA, Saturn file table 등 플랫폼별 물리 테이블을 사용하는 경우:

- 원본과 candidate의 file offset/size/LBA 비교
- overlap 0
- out-of-image 0
- raw table/FST 의도치 않은 변경 0
- 비대상 파일 offset/size 변경 0

을 검증한다.

가능하면 **고정 FST/LBA 위치에서 내용만 교체**한다.

## 8.6 엔디언·정수 폭

- 16/24/32-bit 값을 추측으로 쓰지 않는다.
- signed/unsigned를 구분한다.
- little/big endian을 원본 구조대로 유지한다.
- 포인터/크기 연산 시 overflow/truncation을 검사한다.

## 8.7 아카이브 재구성

아카이브 전체를 다시 패킹할 때는 다음을 보존하거나 명시적으로 검증한다.

- entry order
- entry name/hash
- alignment
- padding
- compressed/uncompressed flag
- per-entry descriptor
- table count
- archive footer/header

한 자산만 수정할 수 있는데 전체 아카이브를 불필요하게 재패킹하지 않는다.

## 8.8 물리 중복 소비자

같은 화면/대사가 논리적으로 하나여도 실제 바이너리에는 여러 복제 레코드가 존재할 수 있다.

따라서:

1. logical table만 수정하고 끝내지 않는다.
2. actual builder가 소비하는 physical inventory를 확인한다.
3. 같은 일본어 원문/한국어 구문/해시/근접 구조로 duplicate consumer를 전수 조사한다.
4. base table과 expanded/merged/runtime table이 따로 있으면 모두 동기화한다.
5. 수정 스크립트는 **재실행 시 change 0**이 되는 멱등 상태를 목표로 한다.

## 8.9 원자적 빌드와 readback

최종 이미지/아카이브 생성은 가능하면:

1. `.partial` 또는 임시 파일에 생성
2. 입력 SHA 검증
3. replacement size/hash 검증
4. 쓰기 수행
5. 디스크/ROM에서 해당 영역을 다시 읽어 replacement와 SHA 비교
6. 보존 대상 영역의 SHA 비교
7. 전체 output SHA 계산
8. 모든 검증이 끝난 후 final 이름으로 atomic move

순으로 한다.

검증 중 실패한 `.partial`을 정상 RC로 취급하지 않는다.

---

## 9. 프리징이 발생했을 때의 원인 분리 순서

프리징을 번역 문자열 하나의 문제라고 단정하지 않는다.
다음 순서로 계층을 분리한다.

### 9.1 먼저 재현 조건 기록

- 디스크/버전
- 정확한 장면
- 진입 순서
- 프리징 직전 화면/대사/UI
- 입력 조작
- 항상 재현 / 간헐 재현
- 정상 마지막 빌드 SHA
- 문제 첫 빌드 SHA

### 9.2 계층별 회귀 분리

가능하면 아래 조합을 만들어 원인을 좁힌다.

1. 원본 텍스트 + 원본 폰트 + 원본 그래픽 + 원본 archive
2. 새 텍스트만
3. 새 텍스트 + 새 폰트
4. 새 그래픽만
5. 새 archive 물리 통합만
6. 최종 통합본

### 9.3 우선 조사 항목

1. 문자열 encoded length / terminator
2. record/block byte budget
3. pointer/offset/size
4. alignment/padding
5. static/dynamic glyph index
6. renderer별 별도 버퍼
7. archive entry descriptor
8. compression descriptor / sector span
9. texture/TIM/CLUT/tile reference
10. FST/LBA/physical overlap

### 9.4 금지

- 프리징이 난 상태에서 원인 불명인 채 다른 대량 수정까지 섞지 않는다.
- “에뮬레이터에서 한 번 됐다”만으로 구조 안전성을 확정하지 않는다.
- 이전 PASS 빌드를 덮어써 비교 기준을 잃지 않는다.

---

## 10. 정적 QA 게이트

프로젝트가 해당 기능을 사용한다면 최소한 아래를 자동 검사한다.

### 텍스트

- untranslated target-language source 잔존
- 고유명사/용어 통일
- 화자/어투 conflict
- 종결/내부 문장부호 정책
- 호격 쉼표
- 조사
- 반복어/비문 후보
- 금지 문자열/깨진 제어코드

### 레이아웃

- line width
- visible row count
- explicit break issue
- right/center alignment overflow
- byte overflow

### 바이너리

- encoded field overflow
- block overflow
- pointer/size mismatch
- glyph missing
- descriptor shrink
- decompression error
- archive entry mismatch
- LBA/FST layout difference
- overlap/out-of-range

정적 QA에서 하나라도 release-blocking failure가 있으면 RC를 만들지 않는다.

---

## 11. 런타임 스모크 테스트

정적 PASS 후 실제 게임에서 최소한 다음을 확인한다.

1. 부팅/타이틀
2. 새 게임/로드
3. 메뉴/상태창/장비창
4. 일반 대화
5. 무전/대사창
6. 컷신/동영상 자막
7. 전투 전후 장면
8. 장면 전환/맵 이동
9. 저장/로드
10. 디스크 교체가 있다면 교체 지점
11. 엔딩 또는 후반 전용 소비처
12. 최근 수정한 모든 화면

특히 **상태창 진입, 컷신 시작/종료, 맵 전환, 보스전 후 전환, 저장/로드**는 버퍼·아카이브·스트리밍 경계 문제를 드러내기 쉬우므로 우선 점검한다.

---

## 12. 빌드 단계 규칙

권장 단계:

1. `SOURCE_QA`
2. `STATIC_BINARY_QA`
3. `RC_BUILD`
4. `RC_READBACK_QA`
5. `RUNTIME_SMOKE`
6. `CANONICAL_PROMOTION`
7. `PATCH_PACKAGE`
8. `RELEASE`

### RC와 canonical 구분

- RC는 테스트용이다.
- canonical은 사용자가 런타임 PASS를 선언한 빌드만 승격한다.
- 패치/xdelta/배포 ZIP은 canonical 승격 이후 만든다.
- 여러 디스크 게임은 모든 디스크 PASS 이후 최종 배포 패치를 만든다.

---

## 13. 보고서에 남길 최소 정보

가능하면 JSON/MD에 다음을 기록한다.

- source path + SHA-256
- output path + SHA-256
- file size
- changed logical entry count
- changed physical consumer count
- overflow count
- missing glyph count
- layout failure count
- pointer/LBA/FST difference count
- overlap/out-of-range count
- preserved region hashes
- static QA status
- runtime smoke status
- canonical promoted 여부
- release patch created 여부

`PASS`라고 쓸 때는 반드시 어떤 범위의 PASS인지 적는다.

---

## 14. Git / 작업 디렉터리 안전 규칙

1. `git add .`를 기본 사용하지 않는다.
2. unrelated dirty files를 reset/clean/delete하지 않는다.
3. ISO/BIN/ROM, `.work`, 거대 임시 산출물은 명시적 정책 없이 커밋하지 않는다.
4. 중요한 런타임 PASS 시점은 checkpoint commit으로 남기는 것을 권장한다.
5. 커밋/푸시는 사용자가 요청하거나 프로젝트 규칙이 명시한 경우에만 수행한다.
6. 최종 빌드에 필요 없는 오래된 RC는 삭제할 수 있으나, 현재 canonical과 마지막 비교 기준은 보존한다.

---

## 15. 프로젝트 인계 규칙

새 세션/새 에이전트는 작업 시작 전에 최소한 다음을 읽는다.

1. 이 문서: `D:\Game - Localizing\LOCALIZATION_QA_STANDARD.md`
2. 프로젝트의 `HANDOFF.md`
3. 프로젝트의 `WORKLOG.md` (있다면)
4. 프로젝트의 `.ai-bridge/current-plan.md`
5. 프로젝트별 `AGENTS.md` / voice policy / terminology policy

권장 인계 문구:

> 먼저 `D:\Game - Localizing\LOCALIZATION_QA_STANDARD.md`를 읽고 공통 QA·런타임 안전 규약으로 적용한다. 그 다음 이 프로젝트의 `HANDOFF.md`, `WORKLOG.md`, `.ai-bridge/current-plan.md`를 읽어 프로젝트별 예외와 현재 상태를 이어받는다. 공통 규약보다 프로젝트 규칙이 더 엄격하면 더 엄격한 쪽을 따른다.

---

## 16. 플랫폼별 추가 주의 예시

이 절은 공통 규약을 대체하지 않고 보강한다.

### GameCube / Wii 계열

- FST offset/size/LBA 보존
- DOL/static font reserve 검증
- compressed `stage.dat`류 descriptor/sector span 축소 금지 또는 명시 검증
- raw FST byte-identical 여부 확인

### PS1 / PS2 계열

- XA/STR/CD sector layout
- TIM/CLUT/bpp
- archive pointer/offset table
- fixed block / Shift-JIS 또는 커스텀 코드 페이지
- alignment 및 sector boundary

### Sega Saturn

- FCP/CHZ/EXP 등 게임별 컨테이너 엔트리 순서와 팔레트
- CD file table/LBA
- VDP 계열 palette/index 제한

### N64

- ROM endian/byte order
- segmented pointer / DMA table
- 압축 블록 크기/오프셋
- texture format / palette / alignment

플랫폼별 구조가 다르므로 “다른 프로젝트에서 됐던 방법”을 그대로 복사하지 않는다.

---

# 17. Definition of Done

한글화의 특정 단계가 `완료`이려면 최소한 다음이 만족돼야 한다.

- 번역 누락 0 또는 명시된 보류 목록 존재
- 용어/화자/어투 QA PASS
- 한국어 문장부호/조사/줄바꿈 QA PASS
- 화면 폭 overflow 0
- byte/block overflow 0
- missing glyph 0
- 포인터/오프셋/크기 구조 검증 PASS
- 압축/아카이브 무결성 PASS
- 물리 layout overlap/out-of-range 0
- 보존 대상 회귀 0
- 빌드 readback PASS
- 사용자가 요구한 런타임 smoke PASS

정적 검사만 끝난 경우에는 `정적 완료`, `runtime smoke pending`처럼 명확히 표시하고 최종 완료라고 부르지 않는다.

---

## 18. 핵심 체크리스트 요약

- [ ] 원본 SHA 확인
- [ ] 번역 원문 확인
- [ ] 화자/청자/어투 확인
- [ ] 용어 중복 소비처 전수 확인
- [ ] 문장부호 정책 확인
- [ ] 조사/띄어쓰기/형태소 확인
- [ ] 실제 렌더러 폭 확인
- [ ] encoded byte budget 확인
- [ ] record/block alignment 확인
- [ ] pointer/offset/size 확인
- [ ] 글리프 plan 및 실제 슬롯 확인
- [ ] 그래픽 palette/CLUT/index 확인
- [ ] 압축 descriptor/span 확인
- [ ] archive entry/order 확인
- [ ] LBA/FST/physical layout 확인
- [ ] readback hash 확인
- [ ] 보존 영역 hash 확인
- [ ] runtime smoke 테스트
- [ ] canonical 승격 후에만 패치 제작

이 체크리스트 중 런타임 안전 관련 항목을 “번역만 바꿨으니 괜찮다”는 이유로 생략하지 않는다.
