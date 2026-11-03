# 같은 도시, 다른 이용환경 — 실행·설정 가이드

성별 상대 우세 따릉이 대여소 주변의 실사 가상 탐색 (네이버 지도 Panorama API)

- `index.html` 하나에 HTML·CSS·JavaScript가 모두 들어 있습니다.
- **네이버 지도 API와 인터넷 연결이 필요한 웹페이지**입니다. 오프라인 파일로는 거리뷰가 표시되지 않습니다.
- 수정할 곳은 파일 상단의 `APP_CONFIG`, `RESEARCH`, `STATIONS` 세 객체뿐입니다.

---

## 1. 네이버 지도 API 준비 (2026-11-03 공식 문서 확인 기준)

### 1-1. Application 등록
1. 네이버 클라우드 플랫폼(NCP)에 로그인 → 콘솔 `Services > Application Services > Maps`에서 **이용 신청**.
2. `Maps > Application` → **Application 등록** (https://console.ncloud.com/maps/application)
   - Application 이름: 20자 이내 (10자 이하 권장)
   - **API 선택: `Dynamic Map` 반드시 체크.** 파노라마는 Web Dynamic Map의 `panorama` 서브 모듈로 제공됩니다. 체크하지 않으면 429(Quota Exceed) 오류가 납니다.
   - **Web 서비스 URL** 등록 (아래 1-2 참고)
3. 등록한 Application을 선택해 **Client ID**를 확인합니다.

> 공식 문서상 Maps API가 개편되면서 스크립트 인증 파라미터가 `ncpClientId` → **`ncpKeyId`** 로 바뀌었고, 신규 클라이언트 아이디 발급이 필요하다고 안내합니다. 예전 블로그 글의 `ncpClientId` 예제는 쓰지 마세요.

### 1-2. Web 서비스 URL 등록 규칙 (공식 가이드)
- 최대 10개, `http://`로 시작해 입력. http/https는 구분하지 않고, `www`는 빼고 입력
- 하위 도메인이 있으면 메인 도메인만 입력
- 실제 사용하는 사이트 주소와 다르면 인증이 실패할 수 있음

이 프로젝트에서 등록할 주소:

| 용도 | 등록 예 |
|---|---|
| 로컬 테스트 | `http://localhost` |
| 배포 후 | 배포 주소 (예: `http://아이디.github.io`) |

⚠️ **포트 번호(`:8000`) 처리 규칙은 공식 문서에서 확인하지 못했습니다.** `http://localhost`로 인증이 실패하면 `http://localhost:8000`도 추가해 보세요. 페이지의 **설정 도우미 > 현재 주소**에 등록해야 할 주소가 표시됩니다.

### 1-3. 인증정보 넣기 — 어떤 값이 공개되어도 되는가

| 값 | 넣는 곳 | 비고 |
|---|---|---|
| **Client ID** | `APP_CONFIG.naver.ncpKeyId` | 공식 예제대로 스크립트 주소(`maps.js?ncpKeyId=...`)에 들어가므로 페이지 소스에서 보입니다. 등록한 Web 서비스 URL에서만 인증되므로, 쓰지 않는 URL은 등록하지 마세요. |
| **Client Secret** | **어디에도 넣지 않음** | 웹 지도·파노라마 표시에는 필요하지 않습니다. HTML·GitHub에 올리지 마세요. 노출이 의심되면 콘솔에서 재발급하세요. |

```js
const APP_CONFIG = {
  naver: {
    ncpKeyId: "여기에_Client_ID",
    ...
```

---

## 2. Windows에서 로컬 실행

`index.html`을 더블클릭(`file://`)하면 등록한 URL과 주소가 달라 인증이 실패합니다. 반드시 로컬 서버로 여세요.

### 방법 A — Python (권장)
1. Python 설치 확인: PowerShell에서 `py --version` (없으면 python.org 또는 Microsoft Store에서 설치)
2. `index.html`이 있는 폴더에서 PowerShell 열기 (탐색기 주소창에 `powershell` 입력 후 Enter)
3. 실행:
   ```
   py -m http.server 8000
   ```
4. 브라우저에서 `http://localhost:8000` 접속 (Chrome·Edge 권장)
5. 종료: PowerShell에서 `Ctrl + C`

### 방법 B — VS Code Live Server
확장 프로그램 Live Server 설치 → `index.html` 우클릭 → *Open with Live Server*. 주소가 `http://127.0.0.1:5500`으로 열리면 Web 서비스 URL에 `http://127.0.0.1`도 등록하거나 주소창을 `localhost`로 바꿔 여세요.

---

## 3. 배포 (학술제 현장·외부 공개용)

정적 파일 하나라서 GitHub Pages 같은 무료 정적 호스팅으로 충분합니다.

1. GitHub에서 새 공개 저장소 생성 → `index.html` 업로드 (Client Secret이 없는지 다시 확인)
2. 저장소 `Settings > Pages` → Branch `main` / `/(root)` → Save
3. 몇 분 뒤 `https://아이디.github.io/저장소이름/` 주소가 생성됨
4. NCP 콘솔 Web 서비스 URL에 `http://아이디.github.io` 추가 (규칙상 메인 도메인만)
5. 전시 당일 설정:
   ```js
   ui: { showSetupTools: false, ... }
   ```
6. 현장 PC에서 인터넷 연결과 거리뷰 로딩을 미리 확인 (현장 Wi-Fi가 방화벽으로 네이버 지도 호스트를 막을 수 있음)

---

## 4. 자료 교체 방법

### 4-1. 1단계 — 파노라마 ID 확보 (대여소 한 곳, 지점 3~5개)
좌표만 넣으면 의도하지 않은 파노라마가 선택될 수 있으므로, 화면으로 확인한 **파노라마 ID**로 지점을 고정합니다.

1. `STATIONS[0].coord`에 대여소 위도·경도 입력 (또는 직접 입력)
2. 로컬 서버로 페이지 열기 → 오른쪽 패널 **설정 도우미** 펼치기
3. **선택한 대여소 좌표 넣기 → 찾기** — `setPosition()`으로 반경 300m 안 가장 가까운 파노라마가 표시됨
4. 화면을 끌어 원하는 방향을 맞추고, 필요하면 화면 안에서 인접 지점으로 이동
5. **현재 파노라마 ID·시야 가져오기 → 복사**
6. 복사한 코드를 `stops` 배열에 붙여 넣고 `title`, `description`, `dwellMs` 수정
7. 3~6을 지점 수만큼 반복 → 새로고침해서 순서대로 확인

복사되는 형식:
```js
{
  panoId: "실제_파노라마_ID",  // 촬영일 … · 주소 …
  pov: { pan: 35.2, tilt: -3.1, fov: 90 },
  title: null,
  description: null,
  dwellMs: 6000
},
```

### 4-2. 필드 설명

**지점 (`stops[]`)**

| 필드 | 설명 |
|---|---|
| `panoId` | 확인한 파노라마 ID (문자열). `null`이면 "자료 미등록" 표시 |
| `pov` | 기본 시야. `pan` -180~180(정북 기준), `tilt` -90~90, `fov` 20~100. 범위를 벗어나면 자동 보정하고 설정 점검에 표시 |
| `title`, `description` | 지점 설명 제목·본문 |
| `dwellMs` | 자동 탐색 시 표시 시간(ms). **로딩 완료 신호를 받은 뒤부터** 계산 |

같은 `panoId`를 연속 지점에 넣고 `pov`만 다르게 하면, 다시 조회하지 않고 시야만 돌립니다.

**대여소 (`STATIONS[]`)**

| 필드 | 설명 |
|---|---|
| `id`, `name` | 대여소 번호·이름 |
| `dominance` | `"male_relative"` / `"female_relative"` |
| `zScoreDiff` | 남성 Z − 여성 Z (부호가 분류와 다르면 경고) |
| `color` | `"navy"`(남성 상대 우세, 남색) / `"burgundy"`(여성 상대 우세, 버건디) |
| `researchPeriod`, `source` | 대여소 자료 기준 시점·출처 |
| `shortNote` | 발표 모드 하단의 짧은 설명 (측정 범위를 함께 적기) |
| `metrics[]` | `{ name, value, unit, scope, source }` — **`scope`(측정 범위) 필수**. 비면 경고 |
| `vlm` | `score`, `scaleMax`, `rubric`, `explanation`, `evaluatedImageReference`, `evaluatedPanoId` |

`vlm.evaluatedPanoId`가 현재 지점 `panoId`와 같지 않으면 "VLM이 평가한 이미지와 현재 화면은 다른 이미지"라는 안내가 자동으로 표시됩니다.

**연구 공통 (`RESEARCH`)** — 연구 제목, 연구진, 이용자료 기간·출처, 분류 방법, `groupFindings`(집단 평균 비교: `{ name, unit, scope, maleRelative, femaleRelative, statistic, pValue }`). 집단 결과는 개별 대여소 값과 다른 섹션에 표시됩니다.

---

## 4-3. 전체 대여소 탐색 모드

상단 **전체 대여소** 버튼을 누르면 연구 자료의 대여소 2,766곳 중에서 골라 볼 수 있습니다.

- 정보 패널에서 번호·이름 검색, 자치구·분류 필터, **조건 안에서 무작위로 고르기**를 쓸 수 있습니다.
- **이전 대여소 / 다음 대여소** 버튼(하단 조작부·정보 패널·발표 모드)으로 현재 검색·필터 목록을 대여소 번호 순서대로 넘겨 봅니다. 끝에 닿으면 처음으로 돌아갑니다.
- 패널의 **"4방향을 다 보면 다음 대여소로 이어서 진행"**을 켜고 자동 탐색을 시작하면, 대여소마다 4방향을 보여 준 뒤 목록의 다음 대여소로 넘어갑니다. 파노라마가 없는 대여소는 3초 뒤 건너뜁니다.
- 색: 남성 상대 우세 대여소는 남색, 여성 상대 우세 대여소는 버건디로 표시합니다.
- 대여소를 고르면 대여소 좌표에서 가장 가까운 네이버 파노라마(반경 300m)를 `setPosition()`으로 찾고, 같은 파노라마에서 북·동·남·서 4방향을 차례로 보여 줍니다.
- 자동으로 고른 파노라마는 사람이 검수하지 않았습니다. 화면 오른쪽 위에 대여소와의 거리가 표시되며, `explore.distanceWarnM`(기본 30m)보다 멀면 경고합니다.
- 패널 자료: 분류·Z점수 차·대여 건수·400m 지표(`Bike_400m_Final.ver.1.2`), VLM 점수·인간 평가자 평균(`BikeDock-HumanAudit`). VLM 설명문은 대표 사례 A·B에만 있습니다.
- 인간 평가 0점(평가 불가) 대여소는 "자료 미등록"으로 표시합니다. 평가자 개인별 점수는 공개 페이지에 넣지 않았습니다.

**자료 갱신:** `index.html` 안의 `<script id="allStationsData">` 블록은 두 CSV에서 생성한 것입니다. 직접 고치지 말고 원본 CSV가 바뀌면 다시 생성하세요.

**끄기:** `APP_CONFIG.explore.enabled = false`

---

## 5. 발표 모드와 녹화

- 상단 **발표 모드**: 설정 UI·패널·조작부를 숨기고 파노라마를 크게, 하단에 대여소명·현재 지점·짧은 설명·출처를 표시
- 마우스를 2.5초 멈추면 상단 조작 버튼이 사라짐 (녹화 화면 정리용)
- 키보드: `Space` 자동 탐색/일시정지, `PageDown`/`→` 다음, `PageUp`/`←` 이전, `Home` 처음, `Esc` 발표 모드 종료 (프레젠터 리모컨의 PageUp/Down 사용 가능)
- 녹화: Windows `Win + Alt + R`(Xbox Game Bar) 또는 OBS
- **녹화한 결과는 일반 동영상이라 시점을 돌릴 수 없습니다.** 관람객이 직접 둘러보는 기능은 웹페이지에서만 가능하며, 발표 모드 캡션에 이 안내가 함께 표시됩니다 (`ui.presentationRecordingNote`로 문구 수정).

---

## 6. 이용요금·공개 전시 조건 (공식 문서에서 확인한 범위)

확인한 내용:
- Maps는 **종량 요금제(구간 요금제)**이며, 일정 사용량을 넘으면 과금됨
- 무료 이용량은 **대표 계정** 기준으로 제공되며, 대표 계정이 아니면 호출 시 과금될 수 있음
- 콘솔에서 **이용 한도 설정·한도 초과 통보**로 사용량을 관리하도록 안내
- 파노라마는 Web Dynamic Map에 포함된 `panorama` 서브 모듈

확인하지 못한 내용:
- **구체적인 단가·무료 제공량 수치, 파노라마 호출의 과금 단위** — 요금 페이지를 직접 열람하지 못했습니다. 콘솔의 요금 안내에서 확인하세요.
- **공개 전시 시 표기·이용 조건** — 문서에는 이용약관(Terms of Service) 링크만 있습니다. 공개 전에 약관을 확인하세요. 이 페이지는 네이버 로고(`logoControl: true`)를 끄지 않습니다.

전시 전 권장: 콘솔에서 **이용 한도와 초과 통보를 설정**해 예상치 못한 과금을 막으세요.

---

## 7. 검증 현황

### A. 네이버 API 연결 전 검증 — 실행 완료
헤드리스 Chromium(데스크톱 1440×900, 모바일 390×844)에서 실행. 실제 네이버 서버에는 접속하지 못해 **모의(mock) `naver.maps` 객체**로 앱의 제어 로직만 확인했습니다. 56개 항목 통과.

- 빈 설정으로 오류 없이 열림, Client ID 없으면 네이버 스크립트를 요청하지 않고 안내 표시
- 대여소 A/B 전환, 색상 구분, 정보 패널 접기·펼치기, 발표 모드 진입·Esc 종료
- 모바일 폭에서 가로 스크롤 없음, 정보 패널이 파노라마 아래로 이동
- 스크립트 요청 실패 안내 / `navermap_authFailure` 호출 시 인증 실패 안내와 현재 주소 표시
- `panoId`로 생성, `size`·`logoControl: true` 전달
- 머무는 시간이 로딩 완료 신호 이후부터 계산됨 (설정 1000ms → 측정 1002ms)
- `pano_changed`가 `pano_status`보다 먼저 오는 순서도 처리
- 같은 `panoId` 연속 지점은 `setPov`만 호출
- `pano_status: ERROR` → 오류 카드 + 자동 탐색 일시정지 + 재시도(새 인스턴스)·다음 지점
- `panoId` 없는 지점 → "자료 미등록" + 파노라마 숨김
- 로딩 지연 안내(테스트 2초) 후 늦게 온 완료 신호 반영
- 파노라마 위 마우스 조작 시 자동 탐색 일시정지, 재개 시 남은 시간만 대기
- 인접 파노라마로 이동 시 이탈 안내·ID 불일치 표시, 지정 지점 복귀
- 대여소 전환 시 기존 타이머가 남지 않음, 마지막 지점에서 자동 정지, 끝난 뒤 시작하면 첫 지점부터
- 패널 접기·발표 모드 시 `setSize`로 크기 반영
- 설정 도우미 좌표 검색(`setPosition`)과 지점 코드 생성

### B. 실제 거리뷰 연결 후 확인할 항목 — 아직 검증하지 못함
- [ ] `ncpKeyId` 인증 성공, localhost 포트 등록 필요 여부
- [ ] 실제 `pano_status` / `pano_changed` 발생 순서와 시점 (모의 객체의 순서는 가정)
- [ ] 생성 직후 첫 로딩에서 이벤트가 정상 발생하는지
- [ ] 머무는 시간 시작 시점: 문서상 `pano_changed`는 "조회 완료" 이벤트이며 **이미지 렌더링 완료 이벤트는 문서에 없음** → 화면이 다 그려지기 전에 시간이 흐르기 시작할 수 있음
- [ ] 지점 간 전환이 실제로 얼마나 자연스러운지 (빈 화면·깜빡임 여부) — **부드러운 연속 주행을 보장하지 않습니다**
- [ ] 마우스 드래그·휠이 파노라마 영역의 `pointerdown`·`wheel`로 감지되어 자동 탐색이 멈추는지
- [ ] 화면 안 인접 지점 이동 시 `pano_changed`로 이탈이 감지되는지
- [ ] `getLocation().photodate`의 표기 형식
- [ ] 네이버 로고·저작권 표기 위치가 지점 배지·이탈 안내와 겹치지 않는지
- [ ] 방향키를 파노라마가 함께 사용하는지 (충돌 시 PageUp/Down 사용)
- [ ] `flightSpot: false` 등 옵션 적용 모습
- [ ] 배포 주소에서의 인증, 현장 네트워크에서의 로딩 속도

---

## 참고한 공식 문서
- Panorama 클래스: https://navermaps.github.io/maps.js.ncp/docs/naver.maps.Panorama.html
- 파노라마 튜토리얼: https://navermaps.github.io/maps.js.ncp/docs/tutorial-Panorama.html
- Hello, World(인증 파라미터·인증 실패 콜백): https://navermaps.github.io/maps.js.ncp/docs/tutorial-2-Getting-Started.html
- 서브 모듈 로딩: https://navermaps.github.io/maps.js.ncp/docs/tutorial-4-Submodules.html
- 클라이언트 아이디 발급: https://navermaps.github.io/maps.js.ncp/docs/tutorial-1-Getting-Client-ID.html
- naver.maps.Event: https://navermaps.github.io/maps.js.ncp/docs/naver.maps.Event.html
- NCP Maps Application 등록: https://guide.ncloud-docs.com/docs/application-maps-app-vpc
- NCP Maps 개요: https://guide.ncloud-docs.com/docs/application-maps-overview
- NCP Maps 시나리오: https://guide.ncloud-docs.com/docs/application-maps-procedure
- NCP Maps 사전 준비(요금제 형태): https://guide.ncloud-docs.com/docs/maps-spec
