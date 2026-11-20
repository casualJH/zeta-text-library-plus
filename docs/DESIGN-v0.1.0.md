# 제타 텍스트 보관함+ v0.1.0 — 기획·기술 설계안

> 상태: **설계 완료 / 구현·실기기 검증 전**  
> 기준: 기존 **zeta-text-library v1.0.2** 유지, Plus 신규 프로젝트 0.1.0  
> 목적: 기존 텍스트 보관함에 저장한 문구를 제타 채팅창에서 `/` 검색으로 찾아 **입력창에 삽입**한다. **자동 전송은 하지 않는다.**

## 1. 제품 구분 및 배포 원칙

| 항목 | 기존 정식판 | Plus |
| --- | --- | --- |
| GitHub | `casualJH/zeta-text-library` | `casualJH/zeta-text-library-plus` |
| 배포 상태 | v1.0.2 정식 배포 유지 | v0.1.0 개발안 |
| Tampermonkey `@name` | 제타 텍스트 보관함 | **제타 텍스트 보관함+** |
| Dock 표시용 이름 | 기존 표시 유지 | **텍스트 보관함+** |
| `@namespace` | `zeta-text-library` | `zeta-text-library-plus` |
| 스크립트 파일 | `zeta-text-library.user.js` | `zeta-text-library-plus.user.js` |
| 데이터 저장 | 기존 키 유지 | 별도의 Plus 키 사용 |
| 자동 업데이트 | 기존 저장소 main | Plus 저장소 main |

- 기존 정식판 `main` 및 배포 파일은 **수정하지 않는다**.
- Plus의 개발 0.x.x는 자체 저장소 `main`에서 관리하고, 최초 정식 배포 1.0.0 이후 기능 개발은 `dev`에서 `-beta.N`으로 진행한다.
- Plus 코드 생성 시 `@homepageURL`, `@updateURL`, `@downloadURL`을 Plus 저장소 기준으로 지정한다.
- Dock에서 명칭으로 구분할 수 있게 `@name`뿐 아니라 실제 Dock에 노출되는 표시 텍스트와 등록 메타데이터도 **텍스트 보관함+**로 지정한다. Dock의 인식 기준은 실제 Dock 구현을 확인한 뒤 결정한다.
- **기존 정식판과 Plus를 동시에 설치한 경우**에도 각각 고유 DOM ID, CSS 호스트, 프로토콜, 저장 키를 사용하여 서로 덮어쓰지 않게 한다.

### 예정 메타데이터 (참고, 아직 실행 파일 아님)

```javascript
// ==UserScript==
// @name         제타 텍스트 보관함+
// @namespace    zeta-text-library-plus
// @version      0.1.0
// @description  제타 텍스트 보관함에 채팅 / 단축어 검색·삽입 기능을 더한 Plus 버전
// @match        https://zeta-ai.io/*
// @match        https://*.zeta-ai.io/*
// @match        https://m.dcinside.com/*
// @match        https://gall.dcinside.com/*
// @match        https://frame.pingpong.ai/*
// @run-at       document-idle
// @grant        GM_getValue
// @grant        GM_setValue
// @updateURL    https://raw.githubusercontent.com/casualJH/zeta-text-library-plus/main/zeta-text-library-plus.user.js
// @downloadURL  https://raw.githubusercontent.com/casualJH/zeta-text-library-plus/main/zeta-text-library-plus.user.js
// @homepageURL  https://github.com/casualJH/zeta-text-library-plus
// ==/UserScript==
```

기존 `@match` 항목을 그대로 둔 채 **마지막 `@match` 다음 줄**에 iframe 항목을 넣는다. `@match` 줄은 연속 배치한다. 기존 코드의 `@noframes`는 Plus에서는 제거하고, 역할 분기로 UI 중복을 방지한다. `@run-at`은 특별한 이유가 검증되지 않으면 원본의 `document-idle`을 유지한다.

## 2. 사용자 경험: 슬래시 단축어

**사용 흐름**
1. 사용자가 제타 채팅 입력창에서 `/` 입력.
2. 사용자 입력을 방해하지 않는 위치에 **텍스트 보관함+ 추천 목록** 표시.
3. `/편지`, `/커뮤반응`처럼 이어서 입력하면 **저장 텍스트의 제목 및 검색용 별칭**으로 목록 필터링.
4. 항목 선택 시 해당 본문을 **현재 활성 채팅 입력창**에 삽입. **자동 전송 금지**.
5. `Esc`, 목록 바깥 클릭, 채팅방 변경, 입력창 이탈 시 목록 닫기. 터치·키보드 모두 지원.

**목록 예시**

```text
채팅 입력창: /커

┌ 텍스트 보관함+ ──────────────────┐
│ /커뮤반응    OOC / 문체            │
│  커뮤니티 반응 형식 지침            │
│ /커뮤요약    OOC / 요약            │
│  최근 대화 요약 지침                │
└──────────────────────────────────┘
```

- 목록은 저장된 텍스트의 **제목·폴더 경로·짧은 본문 미리보기**를 보여준다.
- 슬래시 입력을 전혀 원하지 않는 사용자에게는 **단축어 기능 켜기/끄기** 설정을 제공한다.
- 제타 **기존 / 단축어 메뉴를 제거하거나 재구성하지 않는다.** 기존 메뉴와의 충돌 여부를 모바일 Safari 및 PC Chrome에서 실측하고, 동시에 표시된다면 사용자의 입력/선택을 가로채지 않는 형태로 표시 위치나 트리거 조건을 조정한다. 임의로 플랫폼 기존 명령을 덮어쓰지 않는다.
- 삽입 시 슬래시 검색어를 **어떻게 치환할지**는 현재 입력창을 분석한 뒤 커서 위치·기존 본문 보존을 확인하고 구현한다. 삽입 실패 시 입력 내용을 잃지 않고 오류를 표시한다.
- 한글 IME 조합 중에는 강제 선택/삽입을 하지 않는다.
- 입력 길이 제한, 비활성 입력창, 메시지 전송 상태를 점검한다.

## 3. 기존 보관함 기능 및 UI 유지

Plus는 기존 v1.0.2의 텍스트/폴더/검색/정렬/복사/선택 이동·삭제/JSON 백업·복원/테마/버튼·팝업 위치·크기 조절을 **원형에 가깝게 계승**한다.

- 실행 버튼: 드래그 위치 이동, 글자·이모지 변경, 크기 슬라이더, 드래그와 클릭 구분.
- 보관함 팝업: 헤더 드래그 이동, 오른쪽 아래 ◢로 크기 조절, 최소 크기, 내부 스크롤, 화면 이탈 보정.
- 화면 모드: **라이트 / 다크 / 기기 설정**의 가로 세그먼트 선택, 초기값 **기기 설정**. 시스템 모드 변경 즉시 반영.
- 브라우저 재실행 후 복원, UI 설정만 초기화 가능. 텍스트 본문·폴더 데이터까지 UI 초기화에 포함하지 않음.
- Pointer Events로 모바일/PC를 함께 지원.
- Dock 명칭 **텍스트 보관함+**는 저장 텍스트의 제목/폴더명이 아니라 스크립트 식별/표시 이름에만 적용한다.

## 4. 데이터 보존과 정식판 → Plus 이전

### 저장 격리
- 기존 정식판 저장 키: `zeta-text-vault-data-v1`, `zeta-text-vault-ui-v1`. **절대 수정하거나 삭제하지 않음**.
- Plus 전용 예시 키: `zeta-text-library-plus-data-v1`, `zeta-text-library-plus-ui-v1`, `zeta-text-library-plus-shortcuts-v1`.
- Plus `@namespace` 및 UI 호스트 ID도 독립화. `GM_getValue` / `GM_setValue`를 우선 사용.
- 정식판에서 Plus의 다른 namespace 저장소에 직접 접근할 수 있다고 가정하지 않는다.

### 이전 과정
1. 기존 정식판에서 **JSON 백업** 파일 생성(사용자 수행).
2. Plus에서 **기존 보관함 백업 가져오기** 선택.
3. JSON의 `format: 'zeta-text-vault'` 및 `data.folders`, `data.entries`, `data.orders`, `ui`를 검증.
4. 원본을 수정하지 않고 **Plus 저장소로 복사**. 보관함 내용, 폴더 ID/부모관계, 정렬 순서, 등록일을 보존.
5. 성공 여부와 가져온 항목 수를 표시하고, 실패 시 Plus의 기존 데이터를 복원할 수 있도록 **쓰기 전 스냅샷과 롤백**을 준비.

- **자동으로 기존 보관함을 지우거나 초기화하지 않는다.**
- 가져오기 방식은 **기본값 새 보관함에서 최초 1회 복사**를 우선 제공한다. 기존 데이터에 합치기는 중복 폴더/중복 텍스트/정렬 충돌 위험을 별도 경고하고 초기 버전에서 무리하게 자동 병합하지 않는다.
- Plus 데이터와 정식판 데이터는 **이후 자동 동기화되지 않는다**. 이를 사용자에게 명확히 안내한다.
- JSON 백업은 Plus 전용임을 구분하되 기존 정식판 형식 가져오기와 호환한다. 가져오기가 실패하더라도 정식판 데이터에는 영향 없음.
- 채팅 iframe에는 필요한 목록(제목/경로/별칭과 불필요한 개인정보 없는 미리보기)만 전달하고 본문은 선택 시에만 전송한다.

## 5. iframe 구동, URL 및 역할

```javascript
const frameOrigin = 'https://frame.pingpong.ai';
function isChatFrameUrl(url) {
  return url.origin === frameOrigin
    && /^\/v\d+\/index\.html$/.test(url.pathname);
}
```

- 제타 최상위 사이트(및 기존 허용 사이트): **보관함 실행 버튼, 설정, 데이터 저장, iframe 탐색, 접근 안내 팝업** 관리.
- `frame.pingpong.ai`의 URL이 `/^\/v\d+\/index\.html$/`를 만족하는 **채팅 iframe**: 활성 채팅 입력창 탐지, `/` 검색 UI와 선택 이벤트, 입력창 삽입 처리, 메시지 응답.
- 다른 iframe·잘못된 프레임 경로에서는 어떤 Plus UI도 생성하지 않는다.
- iframe 순서나 고정 인덱스를 쓰지 않는다. 바깥과 안쪽이 동일한 URL 판별 함수를 사용.
- 버전 경로가 실제로 다른 것으로 관찰되면 추측해서 지원 범위를 넓히지 않고 실측 후 수정한다.
- 기존 정식판처럼 시작부터 `window.top !== window.self`로 무조건 종료하는 로직은 Plus에서 **역할별 분기**로 바꾼다.

## 6. 메시지 연결 및 보안 프로토콜

참고: [zeta-output-editor](https://github.com/casualJH/zeta-output-editor/blob/main/zeta-output-editor.user.js)의 `isChatFrameUrl`, `chatFrame`, `callEditor`, iframe 이벤트 수신 처리. 직접 기존 식별자를 재사용하지 않고 Plus 전용으로 설계한다.

- Plus 전용 프로토콜 예: `zeta-text-library-plus/v1`; `MessageChannel` 요청별 1개 포트 사용.
- 요청 타입 예시: `ping`, `search`, `get-text`, `insert-text`. 실제로 필요한 명령은 동작 검증 후 확정.
- 수신 시 **origin / source / 프로토콜 / 요청 ID / 세션 ID / 포트 개수 / 현재 iframe** 모두 검증. 허용하지 않은 타입·인자는 거부.
- 상위에서 iframe으로 보내는 `postMessage`는 정확히 `https://frame.pingpong.ai`를 사용. iframe이 상위 페이지에 보내는 응답도 검증된 현재 상위 origin을 사용하고 `*`는 금지.
- 통신 응답과 **실제 입력창 삽입 성공**은 별도로 확인한다.
- iframe 교체/채팅방 이동 때 이전 세션 무효화. 오래된 응답은 폐기.
- 요청 시간 제한, 로딩 중 재시도, 채널/타이머 정리. iframe 존재만으로 성공 표시 금지.
- 프레임을 찾지 못했거나 통신 불가한 경우와 **프레임은 연결되지만 입력창 요소가 변경된 경우**를 별도 오류로 처리한다.

## 7. iframe 접근 안내 팝업

**표시 조건**
- 사용자가 채팅 기능을 실행할 때 현재 iframe을 대상으로 실제 응답을 확인하고, 적절한 재시도 후에도 연결이 실패하면 표시.
- 정상 응답이 온 뒤 **기능만 실패**하면 안내 팝업은 표시하지 않음.
- 같은 실패에 팝업이 중복 생성되지 않게 관리. 팝업을 닫아도 본래 보관함 기능과 저장 데이터는 영향 없음.

**기준 구현**
- [zeta-profiles-mob v1.0.5의 `showSceneAccess`](https://github.com/casualJH/zeta-profiles-mob/blob/1befb93e9160ec956a55ec1227fde9b48dc1936b/zeta-profiles-mob.user.js#L48)
- Safari·Chrome **원본 안내 이미지(data URL)를 그대로 재사용**. 임의 교체 금지.
- 화면 가운데, Safari 다음 Chrome, 두 항목 최초 접힘, 아래 닫기 버튼은 전체 너비 및 가운데 정렬.
- 아래 **문구를 그대로 사용**.

```text
Tampermonkey 사이트 접근 안내

스크립트 실행을 위해, 아래 링크에서 Tampermonkey 접근 허용 설정을 해주세요.

frame.pingpong.ai 열기 ↗

Safari
[기준 소스의 Safari 이미지]
링크를 눌러 새 탭에서 사이트를 열고, Tampermonkey 메뉴를 열어 [이 웹사이트에서 항상 허용]을 누르세요.

Chrome
[기준 소스의 Chrome 이미지]
링크를 눌러 새 탭에서 사이트를 열고, Tampermonkey 메뉴를 열어 [사이트 데이터를 읽고 변경할 수 있음]-[frame.pingpong.ai에서]를 누르세요.

닫기
```

- 안내 링크는 현재 확인한 `frame.pingpong.ai` 프레임의 **origin + pathname**. 검색어/해시 제거.
- 유효 경로를 감지하지 못했다면 임의의 버전을 추측하지 않고 `https://frame.pingpong.ai/` 사용.
- 링크: `target="_blank"`, `rel="noopener noreferrer"`; 상단 제목·문구·링크 배치 유지.

## 8. 개발 및 검증 단계

### 0.1.0 범위
- 기존 보관함 Plus 복제 및 데이터·UI·Dock 식별 분리
- JSON 원본 복사 가져오기
- `/` 검색 목록, 항목 검색·선택·본문 삽입(자동 전송 없음)
- 현재 채팅 iframe을 실제 연결 응답으로 확인
- 접근 안내 팝업(기준 이미지·문구·디자인)
- 기존 기능 회귀 및 데이터 보호 검증

### 구현 전에 실측해야 할 사항
- 현재 제타의 채팅 iframe 실제 URL 및 바깥 사이트 origin
- Safari/Chrome의 **활성 입력창 DOM 및 편집 이벤트**, 제타의 자체 `/` 목록과 충돌 여부
- iOS 한국어 조합(IME), 모바일 키보드 열림·닫힘, 커서 위치·문자 선택, 채팅방 전환
- Dock의 **스크립트 식별 및 표시명 규칙** (`@name`만으로 인식하는지 등)

### 완료 판정 체크리스트
- [ ] 정식판 v1.0.2 및 `main` 불변
- [ ] Plus의 `@name`/Dock 표기가 `텍스트 보관함+`로 구별됨
- [ ] 모든 `@match` 연속 배치, `frame.pingpong.ai/*` 포함, `@noframes` 없음
- [ ] iframe 판별이 `/^\/v\d+\/index\.html$/`에 일치
- [ ] 메인/iframe 역할 분리 및 UI 중복 없음
- [ ] 엄격한 origin/source/protocol/포트 검증과 실제 요청·응답 확인
- [ ] 새 iframe 교체 및 오래된 응답 폐기
- [ ] 연결 실패 / 기능 실패 구분
- [ ] 안내 팝업은 연결 불가 때만, 원본 이미지·문구·링크 조건 준수
- [ ] 원래 제타의 `/` 기능과 충돌 없음
- [ ] 본문 입력 성공 확인, 자동 전송 없음
- [ ] 기존 JSON 데이터·폴더·순서·UI 설정 Plus에 정상 **복사**, 기존 정식판 데이터 불변
- [ ] Plus의 백업·복원 및 브라우저 재실행 뒤 보존
- [ ] 모바일 Safari / PC Chrome 실기기 회귀 테스트
- [ ] 최종 출시 전 코드 diff, JavaScript 문법 및 메타데이터 URL 검증

## 9. 문서의 한계

이 문서는 **계획 및 기술 설계**이다. 제타의 현재 채팅 입력창 DOM, Dock의 인식 코드, 원래 슬래시 메뉴와의 공존은 아직 실기기 검증되지 않았다. 따라서 실제 삽입 성공, iframe 연결 성공, 접근 안내 팝업 표시를 **구현 완료라고 주장하지 않는다**.

### 참고 소스
- [기존 정식판 v1.0.2](https://github.com/casualJH/zeta-text-library/blob/main/zeta-text-library.user.js)
- [iframe 통신 기준 zeta-output-editor](https://github.com/casualJH/zeta-output-editor/blob/main/zeta-output-editor.user.js)
- [접근 안내 팝업 기준 zeta-profiles-mob v1.0.5](https://github.com/casualJH/zeta-profiles-mob/blob/1befb93e9160ec956a55ec1227fde9b48dc1936b/zeta-profiles-mob.user.js#L48)
