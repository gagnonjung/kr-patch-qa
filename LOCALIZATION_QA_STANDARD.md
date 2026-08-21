# Game Localization Common QA & Runtime Safety Standard

Version: 1.2
Updated: 2026-08-21
Scope: 레트로 게임 한국어 패치/한글화 프로젝트 전반

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

6. **지원 리비전을 해시로 고정한다.**
   - 지원하는 원본 ROM/ISO/BIN/CUE/archive의 MD5/SHA-256 또는 동등한 강한 식별값을 기록한다.
   - 입력 해시가 다르면 “같은 게임이니 구조도 같을 것”이라고 추정해 계속 진행하지 않는다.
   - 다른 리비전을 지원하려면 포인터·파일 배치·압축·실행 파일·아카이브 구조 등 현재 변경 경계가 동일하다는 별도 근거를 만든다.

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

### 2.3 사람이 편집하는 번역 원본과 게임 소비 바이트

가능하면 다음을 별도 계층으로 유지한다.

- 사람이 읽고 수정하는 UTF-8 번역 원본
- 원문·화자·장면·레코드 ID·제어 토큰 등 검증 메타데이터
- 게임의 실제 인코딩/패킹 형식으로 변환된 확정 빌드 입력

`original`, entry id, control field처럼 구조를 식별하는 값은 번역문과 같은 편집 대상으로 취급하지 않는다.
게임 소비 바이트가 수동 보정이나 특수 변환을 포함한다면 어떤 사람이 읽는 원본에서 어떤 검증된 변환을 거쳐 만들어졌는지 추적 가능해야 한다.

### 2.4 디코더/추출 규칙이 바뀌었을 때

기존 추출 규칙이 틀렸다고 밝혀지면 잘못 추출된 텍스트를 수동으로 덧대어 계속 쓰지 않는다.

1. 불변 원본에서 새 규칙으로 전체 대상 모집단을 다시 추출한다.
2. 기존 번역은 stable entry/message ID 또는 검증된 대응 관계로 새 원문에 이관한다.
3. 원문이 달라진 항목과 번역 재검토가 필요한 항목을 별도 audit으로 남긴다.
4. 재추출 후 전체 모집단 수, 누락/추가 엔트리, 제어 토큰을 다시 검증한다.
5. 같은 입력으로 재실행했을 때 추가 변경 0이 되는 idempotent 상태를 확인한다.

추출기가 “대부분 맞는다”는 이유로 이미 알려진 잘못된 디코딩을 canonical 입력으로 유지하지 않는다.

### 2.5 문맥 교정과 검색 정규화

자동 교정은 가능하면 **원문 전체 문장, stable message/entry ID, 화자/소비처 같은 원래 문맥 키**에 묶는다.

- 같은 한국어 표면 문자열이 여러 일본어 원문/화자/상황에서 나올 수 있으므로 한국어 결과 문자열만 보고 전역 치환하지 않는다.
- 검색 편의를 위해 장음표, 공백, 전각/반각, 표기 변형을 넓게 찾을 수는 있지만, 실제 패치 적용 전에는 target revision의 **정확한 원문/바이트 guard**를 다시 확인한다.
- 외부 대본이나 다른 지역판 표기는 discovery 보조일 뿐 target revision의 실제 원문을 덮어쓰는 기준이 아니다.

### 2.6 번역 승인 상태

대규모 프로젝트에서는 최소한 다음 상태를 구분하는 것을 권장한다.

- `needs_translation`
- `drafted`
- `needs_human_review`
- `approved`
- `excluded_non_user_facing`

자동 QA가 모두 통과해도 `drafted`가 사람 검수를 자동으로 의미하지 않는다. 이미 `approved`된 항목이라도 번역문, 원문 해석, 제어코드, 레이아웃 또는 소비 구조가 바뀌면 다시 `needs_human_review`로 돌린다.

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
- 원작·공식 작품에 이미 정착된 명칭이 있으면 프로젝트 용어 정책에서 그 명칭을 우선 검토하고, 임의 번역명은 별도 근거 없이 새 canonical로 만들지 않는다.

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
- 의존 명사는 한국어 문법에 맞게 띄어 쓴다. 예: `할 수 있다`, `한 것 같다`, `갈 줄 안다`.
- 보조 용언, 의존 명사, 조사 결합은 단순 공백 정규식으로 일괄 치환하지 말고 문맥과 프로젝트 문체를 함께 본다.
- `수밖에`, 고유명사+조사, 숫자+단위처럼 겉모양만 보고 분리/결합하면 오히려 틀릴 수 있는 구성은 형태소 단위로 확인한다.
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
   - 내부에 공백이 있는 다어절 고유명사의 중간 분리
   - 숫자+단위, 버튼명, 약어 분리
   - 한글 조합을 시각적으로 부자연스럽게 절단
   - 숨겨진 4번째 줄 생성
   - 우측 정렬/중앙 배치 침범

4. 자동 줄바꿈은 후보 생성용이다. 최종 개행은 문맥과 실제 화면을 기준으로 승인한다.

### 5.1 보호 토큰과 제어코드

번역문에 포함된 비텍스트 토큰은 일반 문자와 별도 계약으로 취급한다.

- 색상, 버튼, 속도, 대기, 음성/타이밍, 동적 치환, 페이지/종단 등 보호 토큰의 종류·순서·개수는 원문과 동일해야 한다.
- 줄바꿈 토큰은 한국어 레이아웃을 위해 이동할 수 있는 경우에도 다른 보호 토큰을 넘어 재배치하지 않는다.
- 동적 치환 토큰의 실제 표시 폭을 모르면 정적 추정만으로 레이아웃 PASS를 확정하지 않는다.
- 컴파일러/빌더는 가능한 경우 보호 토큰 불일치를 release-blocking failure로 처리한다.

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
7. 화면/스테이지/모듈별 subset font를 쓰는 게임에서는 **주소 가능한 glyph slot ceiling**과 **폰트 blob의 물리 tile 수**를 별도 한계로 추적한다.
   - 물리 blob에 타일을 추가할 수 있어도 런타임 로더가 그 타일까지 업로드/참조할 수 있다는 증거가 없으면 capacity가 늘었다고 판정하지 않는다.
   - 반대로 주소 가능한 슬롯이 남아 있어도 해당 화면이 실제로 로드하는 font blob이 짧으면 물리 타일 부족이 별도 blocker가 될 수 있다.
8. 같은 글리프/폰트가 여러 컨테이너에 복제되어 있으면 시각적으로 닮은 파일 하나만 수정하지 않는다.
   - 실제 화면/VRAM/RAM 캡처의 바이트나 런타임 로드 경로로 진짜 소비 자산을 확인한다.
   - 공유 슬롯을 다른 화면도 사용하는 경우 해당 consumer 전체의 문자 집합을 합쳐 충돌 여부를 검사한다.
   - 한 화면에서만 성공하는 isolated test용 slot reuse는 다른 consumer를 깨뜨릴 수 있으므로 release 근거로 사용하지 않는다.

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
6. `4bpp`라서 항상 `0..15` 전 인덱스를 안전하게 쓸 수 있다고 가정하지 않는다.
   - 자산별 실제 opaque/transparent index 범위와 CLUT ramp를 원본에서 확인한다.
   - 유효하지 않은 인덱스가 투명/예약 상태라면 재양자화 시 해당 인덱스를 사용하지 않는다.
7. 여러 상태(normal/selected/disabled 등)가 같은 atlas나 palette를 공유하면 한 상태만 보고 palette-safe라고 판정하지 않는다.
8. 다른 게임/전작/지역판의 검증된 자산을 재사용할 수는 있지만, **dimensions, texture format, bpp, palette/TLUT semantics, index vocabulary, tiling/swizzle, archive span**이 target consumer와 호환되는 경우에만 사용한다.
9. 다른 지역판/버전의 XML, 심볼, texture offset, archive layout은 힌트로만 사용한다. 실제 target revision의 consumer/segment/overlay/pointer 참조가 최종 권위다.
10. 재사용 가능한 검증 자산이 있는데 같은 모양을 임의 재생성하지 않는다. 반대로 호환성이 증명되지 않은 자산은 “같아 보인다”는 이유로 복사하지 않는다.

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

### 8.4.1 decoded/raw size도 런타임 계약이다

압축 컨테이너가 같은 physical window 안에 들어간다는 이유만으로 안전하다고 판정하지 않는다.

- decoded/raw size
- outer container raw size
- raw/packed descriptor
- allocator에 전달되는 크기
- decompression destination buffer

를 서로 별도 값으로 추적한다. repack/round-trip이 성공해도 decoded size 또는 raw-size descriptor 변경이 런타임 allocator/loader 계약을 깨뜨릴 수 있다. 구조가 증명되지 않았다면 known-good raw-size/descriptor floor 또는 exact value를 보존한다.

## 8.5 ISO / ROM 물리 레이아웃

GameCube FST, CD sector, PS1/PS2 LBA, Saturn file table 등 플랫폼별 물리 테이블을 사용하는 경우:

- 원본과 candidate의 file offset/size/LBA 비교
- overlap 0
- out-of-image 0
- raw table/FST 의도치 않은 변경 0
- 비대상 파일 offset/size 변경 0

을 검증한다.

가능하면 **고정 FST/LBA 위치에서 내용만 교체**한다.

Mode2/2352, XA/STR, CD-DA 혼합 트랙처럼 섹터 구조 자체가 재생/스트리밍 계약인 자산은 일반 ISO 재빌드 성공만으로 안전하다고 판정하지 않는다.

- same-size 교체가 가능하면 원본 sector 위치와 subheader/form 구조를 유지하는 in-place 패치를 우선 검토한다.
- XA/STR 등 interleaved stream은 sector count, channel/file number, coding info, subheader, EDC/ECC 등 실제 소비 필드를 검증한다.
- 한 채널/구간만 바꿀 때 다른 sector/stream이 byte-identical 또는 decoded-equivalent인지 확인한다.
- 재빌드 이미지가 부팅되더라도 해당 스트리밍 구간에서 stall/desync가 발생하면 물리 레이아웃 FAIL로 취급한다.

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
6. 같은 문구/그래픽이라도 소비처별 glyph index, codebook, palette, runtime family가 다르면 하나의 재생성 자산으로 강제로 통일하지 않는다.
7. 이미 런타임 PASS한 복제 자산이 있다면 새로 재인코딩한 “동일 내용” 자산보다 그 **검증된 바이트 계보(lineage)**를 우선한다. 다른 consumer에 복사할 때도 해당 consumer의 인코딩/로더 호환성을 별도로 증명한다.

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

## 8.10 단일 주 빌드 경로

최종 배포 산출물은 가능하면 하나의 canonical pipeline/entry point에서 만든다.

그 경로는 최소한 다음을 연결한다.

1. 지원 원본 리비전/해시 확인
2. 수정 대상 모집단과 구조 검증
3. 승인된 번역·그래픽·바이너리 변경 적용
4. 슬롯·포인터·오프셋·압축·정렬·크기 검증
5. 최종 아카이브/ROM/디스크 이미지 직렬화
6. 최종 이미지 readback과 기준선/보호 영역 검증

여러 독립 도구의 부분 성공을 합산해 최종 빌드 PASS로 판정하지 않는다.
개별 재작업 경로가 있더라도 최종 후보는 주 빌드 경로로 다시 통합해 검증한다.

## 8.11 런타임 생성 문자열과 코드 내 하드코딩

translation table에 모든 사용자 표시 문자열이 존재한다고 가정하지 않는다.

- 시간/수치 단위
- 화폐 단위
- 동적 아이템/플레이어 이름
- 버튼/상태 라벨
- 런타임 formatter가 조합하는 접미사

등은 실행 코드, overlay, formatter, decoder 안에 하드코딩될 수 있다. 정적 inventory에는 **table-driven text와 runtime-generated text를 별도 모집단**으로 기록하고, 알려진 dynamic control path를 전수 조사한다.

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
- cold boot / 일반 세이브 로드 / save state 로드 여부
- 해당 save state가 수정 자산의 초기 로드 전/후 어느 시점에 생성됐는지
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

- canonical unique population과 physical occurrence population의 수량/도달 가능성
- untranslated target-language source 잔존
- table-driven text와 runtime-generated/hardcoded text의 누락 여부
- 고유명사/용어 통일
- 화자/어투 conflict
- 종결/내부 문장부호 정책
- 호격 쉼표
- 띄어쓰기/의존 명사/보조 용언
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
- decoded/raw size 또는 raw descriptor의 예상 밖 변화
- glyph missing
- descriptor shrink
- decompression error
- archive entry mismatch
- runtime-family/codebook/glyph-index lineage mismatch
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

### 11.1 cold boot와 save state 오염 방지

디스크/ROM에서 자산을 초기화 시점에 한 번만 읽어 RAM/VRAM에 유지하는 게임은 save state가 새 후보의 자산 로드를 우회할 수 있다.

- 수정 대상이 부팅/장면/전투 초기화 때 로드된다면 최소 1회는 **새 후보 cold boot → 일반 게임 세이브 로드 또는 정상 진행 → 대상 화면 진입**으로 검증한다.
- 수정 자산이 이미 로드된 뒤 만든 save state는 구 RAM/VRAM/코드 상태를 복원할 수 있으므로 새 디스크/ROM의 runtime PASS 근거로 단독 사용하지 않는다.
- save state를 재현 도구로 사용할 때는 생성 빌드 SHA와 생성 시점을 기록하고, 해당 state가 어떤 자산의 reload를 우회하는지 명시한다.
- 정적 readback과 새 후보의 디스크 바이트가 맞는데 화면만 과거 상태를 보이면, 즉시 새 패치 실패로 단정하기 전에 stale save-state/RAM/VRAM 가능성을 분리한다.

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
- RC를 만든 뒤 source/translation/font/graphics/binary input이 바뀌면 기존 RC는 **stale**이다. 새 소스가 미반영된 후보를 최신 RC/canonical처럼 재사용하지 않는다.
- 패치/xdelta/배포 ZIP은 canonical 승격 이후 만든다.
- 여러 디스크 게임은 모든 디스크 PASS 이후 최종 배포 패치를 만든다.
- 배포 패치는 가능하면 **지원 retail 원본 → 최종 canonical**을 직접 변환한다. 이전 버전 패치를 먼저 적용해야 하는 증분 체인을 기본 배포 경로로 만들지 않는다.
- 패치 encode→decode 결과는 최종 canonical 이미지와 byte-identical인지 검증한다.
- 패키지 내용이 바뀌면 기존 다운로드 미러/링크는 새 패키지 해시가 실제로 업로드되기 전까지 최신 링크로 표기하지 않는다.
- 내부 QA/문구 보정마다 의미 없는 버전 번호를 올리지 않는다. 배포 버전은 실제 릴리스 또는 명확한 호환성/기능 경계에 맞추고, 중간 후보는 RC/checkpoint 식별자로 구분한다.

---

## 13. 보고서에 남길 최소 정보

가능하면 JSON/MD에 다음을 기록한다.

- source path + SHA-256
- source commit/revision 또는 번역/자산 입력의 기준 식별자
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
- runtime test provenance(cold boot / 일반 세이브 / save state와 생성 빌드)
- candidate stale 여부
- canonical promoted 여부
- release patch created 여부
- release package SHA 및 현재 미러가 그 해시를 가리키는지 여부

`PASS`라고 쓸 때는 반드시 어떤 범위의 PASS인지 적는다.

---

## 14. Git / 작업 디렉터리 안전 규칙

1. `git add .`를 기본 사용하지 않는다.
2. unrelated dirty files를 reset/clean/delete하지 않는다.
3. ISO/BIN/ROM, `.work`, 거대 임시 산출물은 명시적 정책 없이 커밋하지 않는다.
4. 중요한 런타임 PASS 시점은 checkpoint commit으로 남기는 것을 권장한다.
5. 커밋/푸시는 사용자가 요청하거나 프로젝트 규칙이 명시한 경우에만 수행한다.
6. 최종 빌드에 필요 없는 오래된 RC는 삭제할 수 있으나, 현재 canonical과 마지막 비교 기준은 보존한다.
7. 공개 저장소에는 원본 ROM/ISO/디스크 이미지, 패치된 전체 이미지, 원본에서 통째로 추출한 저작 자산, 생성된 게임 실행 파일을 기본적으로 포함하지 않는다.
8. 공개 저장소에는 번역 원본/메타데이터, 소스 형태의 툴체인, 재현 가능한 빌드·검증 스크립트, 합법적으로 재배포 가능한 차분 데이터, 지원 원본의 식별 해시 등 재현에 필요한 최소 자료를 우선한다.

---

## 15. 프로젝트 인계 규칙

새 세션/새 에이전트는 작업 시작 전에 최소한 다음을 읽는다.

1. 이 문서: 프로젝트가 고정한 `LOCALIZATION_QA_STANDARD.md` 또는 `https://github.com/gagnonjung/kr-patch-qa/blob/main/LOCALIZATION_QA_STANDARD.md`
2. 프로젝트의 `HANDOFF.md`
3. 프로젝트의 `WORKLOG.md` (있다면)
4. 프로젝트의 `.ai-bridge/current-plan.md`
5. 프로젝트별 `AGENTS.md` / voice policy / terminology policy

권장 인계 문구:

> 먼저 프로젝트가 지정한 `LOCALIZATION_QA_STANDARD.md` 또는 `kr-patch-qa`의 공통 규약을 읽고 QA·런타임 안전 기준으로 적용한다. 그 다음 이 프로젝트의 `HANDOFF.md`, `WORKLOG.md`, `.ai-bridge/current-plan.md`를 읽어 프로젝트별 예외와 현재 상태를 이어받는다. 공통 규약보다 프로젝트 규칙이 더 엄격하면 더 엄격한 쪽을 따른다.

---

## 16. 플랫폼별 추가 주의 예시

이 절은 공통 규약을 대체하지 않고 보강한다.

### 16.1 적용 범위는 비트 세대가 아니라 실제 저장·소비 구조로 나눈다

이 규약의 공통 불변식은 8/16/32/64비트 등 세대와 무관하게 적용한다.

- 원본/지원 리비전 고정
- 실제 게임 원문 우선
- 한국어 문장/화자/용어 QA
- 실제 소비자 기준 byte/display/glyph capacity 검증
- 변경 경계와 보호 영역 검증
- final readback
- 정적 QA와 runtime smoke 분리
- last-known-good 회귀 관리
- runtime PASS 이후 canonical 승격

반면 다음 항목은 해당 구조가 실제로 존재할 때만 적용한다.

- ISO extent, sector, LBA, FST, 파일시스템 테이블
- 대형 아카이브의 entry descriptor
- 압축 stream과 별도 allocation span/sector reservation
- 디스크 스트리밍 경계

따라서 “32비트 이전/이후” 자체를 QA 적용 경계로 삼지 않는다. 예를 들어 N64는 카트리지 기반이지만 DMA table·segmented pointer·압축 블록 검증이 중요하고, 8/16비트 카트리지 게임은 LBA 대신 bank mapping·pointer width·ROM window·VRAM/tile budget이 핵심일 수 있다.

### 8/16비트 카트리지 중심 플랫폼

SNES, Mega Drive/Genesis, Game Boy/GBC, Game Gear, NES 등에서는 해당 게임 구조에 따라 다음을 우선 확인한다.

- CPU 주소 공간과 ROM bank mapping
- bank crossing 허용 여부와 pointer/address width
- fixed bank / switchable bank / mapper 규칙
- 16-bit/24-bit pointer 및 bank byte 분리 저장
- free-space 사용 시 실제 runtime bank reachability
- text/font/tile data가 공유하는 고정 슬롯과 bank budget
- PPU/VDP tile index, tilemap, palette index, VRAM/CGRAM/CRAM 제약
- 압축 데이터가 있으면 decompressor의 출력 버퍼와 bank 경계
- ROM expansion을 할 경우 header/mapper/checksum 및 실제 매핑 변화 검증

이 플랫폼들에는 존재하지 않는 ISO/LBA/FST 검사를 억지로 적용하지 않는다. 대신 그 항목이 보호하려던 목적 — **실제 소비자가 읽는 위치·크기·경계가 바뀌지 않았는지 검증하는 것** — 을 해당 플랫폼의 bank/pointer/tile 구조로 치환한다.

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
- 필요한 사람 검수/승인 상태가 명시됨
- table-driven 및 known runtime-generated text 누락 0 또는 명시된 보류 목록 존재
- 용어/화자/어투 QA PASS
- 한국어 문장부호/조사/띄어쓰기/줄바꿈 QA PASS
- 화면 폭 overflow 0
- byte/block overflow 0
- missing glyph 0
- 포인터/오프셋/크기 및 decoded/raw-size 계약 검증 PASS
- 압축/아카이브 무결성 PASS
- 물리 layout overlap/out-of-range 0
- 보존 대상 회귀 0
- 빌드 readback PASS
- RC가 최신 source/input 기준임
- 사용자가 요구한 런타임 smoke PASS이며 필요한 경우 cold-boot provenance가 기록됨

정적 검사만 끝난 경우에는 `정적 완료`, `runtime smoke pending`처럼 명확히 표시하고 최종 완료라고 부르지 않는다.

---

## 18. 핵심 체크리스트 요약

- [ ] 원본 SHA / target revision 확인
- [ ] 번역 원문과 문맥 guard 확인
- [ ] canonical unique / physical occurrence 모집단 확인
- [ ] 번역 승인 상태 확인
- [ ] 화자/청자/어투 확인
- [ ] 원작/공식 용어 및 중복 소비처 전수 확인
- [ ] 문장부호 정책 확인
- [ ] 조사/띄어쓰기/의존 명사/보조 용언 확인
- [ ] 다어절 고유명사 줄바꿈 보호 확인
- [ ] runtime-generated/hardcoded text 확인
- [ ] 실제 렌더러 폭 확인
- [ ] encoded byte budget 확인
- [ ] record/block alignment 확인
- [ ] pointer/offset/size 확인
- [ ] decoded/raw size 및 raw/packed descriptor 확인
- [ ] 글리프 plan 및 실제 슬롯 확인
- [ ] 그래픽 palette/CLUT/index 및 target-revision layout 확인
- [ ] 재사용 자산 호환성과 verified lineage 확인
- [ ] 압축 descriptor/span 확인
- [ ] archive entry/order 확인
- [ ] LBA/FST/physical layout 확인
- [ ] readback hash 확인
- [ ] 보존 영역 hash 확인
- [ ] RC가 최신 source 기준인지 확인
- [ ] runtime smoke 시 cold boot/save-state provenance 확인
- [ ] canonical 승격 후 retail→canonical 패치 제작 및 decode 검증

이 체크리스트 중 런타임 안전 관련 항목을 “번역만 바꿨으니 괜찮다”는 이유로 생략하지 않는다.
