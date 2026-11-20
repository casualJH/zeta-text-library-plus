# 제타 텍스트 보관함+ (Plus)

**개발·테스트 버전 v0.1.0** — 기존 [제타 텍스트 보관함 v1.0.2](https://github.com/casualJH/zeta-text-library)과 독립적으로 설치하는 확장판입니다.

## 설치

[Plus v0.1.0 유저스크립트 설치 (Raw)](https://raw.githubusercontent.com/casualJH/zeta-text-library-plus/main/zeta-text-library-plus.user.js)

Tampermonkey에서 설치한 뒤 제타를 새로고침하세요. 기존 버전을 사용 중이라면 **반드시 먼저 JSON 백업을 보관**하세요.

## 기능

- 기존 보관함의 텍스트·폴더·정렬·선택·검색·백업·테마·실행 버튼과 팝업 UI
- 제타 채팅창 `textarea[name="message"]`에 `/` 입력 시 저장 텍스트 검색 목록 표시
- 목록 선택 시 해당 텍스트를 입력창에 삽입 (**자동 전송 없음**)
- 채팅 입력 길이 **5,000자** 및 기존 `maxLength` 제한 초과 시 삽입 거부
- Plus 실행 버튼 ID: `text-library-plus-launcher`, `aria-label` 및 `title`: **텍스트 보관함+**
- 설정에서 슬래시 단축어 기능 켜기/끄기
- `frame.pingpong.ai/v숫자/index.html` 프레임만 채팅 대상으로 인식. 기타 `warm.html` 프레임은 제외
- 유효한 채팅 iframe과 실제 통신하지 못했을 때 Tampermonkey 접근 안내 팝업 제공 (Safari·Chrome)

## 저장 데이터 보호

기존 정식판의 저장 키는 변경하거나 쓰지 않습니다.

| 항목 | 정식판 | Plus |
| --- | --- | --- |
| 텍스트·폴더 | `zeta-text-vault-data-v1` | `zeta-text-library-plus-data-v1` |
| UI 설정 | `zeta-text-vault-ui-v1` | `zeta-text-library-plus-ui-v1` |

**기존 저장 내용은 자동으로 Plus에 복사되지 않습니다.** 정식판에서 `백업 → JSON 전체 백업 다운로드`를 실행하고 Plus의 `백업 → 가져오기`에서 해당 JSON을 선택하세요. 두 버전은 이후 자동으로 동기화되지 않습니다.

Plus의 기존 데이터에 **합치기**를 사용하면 동일한 텍스트나 폴더가 중복될 수 있습니다. 비어 있는 Plus를 처음 사용할 때 기존 JSON을 **교체** 방식으로 가져오는 것을 권장합니다.

## 현재 검증 상태

GitHub 업로드 및 JavaScript 문법 검사를 완료했습니다. **실제 모바일 Safari, PC Chrome, Dock 연동 및 제타 입력창 삽입의 정상 동작은 아직 검증되지 않았습니다.** 문제가 생기면 기존 정식판의 백업은 삭제하지 마세요.

[기획·기술 설계안 v0.1.0](docs/DESIGN-v0.1.0.md)

---

기존 `zeta-text-library`의 `main` 파일은 Plus 개발 과정에서 수정하지 않습니다.
