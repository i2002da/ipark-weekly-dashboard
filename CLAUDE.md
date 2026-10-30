# 천안IPC 통합현장 주간업무현황 대시보드 — Claude Code 작업 가이드

> 요청 이력·설계 결정·시행착오 상세는 `docs/HISTORY.md` 참조.

HDC현대산업개발 천안IPC 현장(3단지·4단지·5단지·6단지·기반시설·DX TFT)의 주간 업무(금주 진행/차주 예정)를 취합하는 대시보드.
사용자는 총괄소장이며 **한국어**로 짧은 번호 목록으로 요청한다. 응답·커밋 메시지·버전 이력은 한국어.

## 구조
- 단일 파일 `index.html` (CSS+HTML+JS 한 파일), GitHub Pages 배포: https://i2002da.github.io/ipark-weekly-dashboard/
- 저장소: `i2002da/ipark-weekly-dashboard` (main 브랜치 push = 배포, 반영까지 30~60초)
- 데이터: Firebase Realtime DB (compat SDK 10.12.2), 프로젝트 `ipc-dashboard-61be5`, 루트 `ipark_weekly_v1`
- 라이브러리: marked@9, turndown(+gfm) — WYSIWYG(contenteditable) 편집기
- `database.rules.json`: 보안 규칙(콘솔에 수동 게시 — 배포되지 않음)

## Firebase 경로
- `sites/{site}/{year}/{weekKey}` 주간 데이터 · `categories` 단지별 공종(주차별 스냅샷) · `calendar` 일정 · `event_types` 일정 분류
- 인증(v3.9~): `profiles/{uid}` {id,name,role,site} · `usernames/{hex(id)}` {email,uid} · (기존) `users` 배열 — 전환 전 평문 pw 보관용, 전환되면 pw 삭제
- 로그인: 아이디 → 가상 이메일 `u<hex>@ipark.local` → Firebase Auth. 기존 계정은 첫 로그인 때 자동 전환(`migrateLegacyUser`). Authentication 미사용 상태면 `legacyLogin` 호환 모드.
- 권한: admin / manager(작성·저장) / viewer(열람)

## 주요 코드 위치(검색 키워드)
`const VER` · `VERSION_HISTORY` · `HELP_SECTIONS` · `showView` · `openSiteOverview` · `renderOverview` · `renderWYEditor` · `getCats/ensureCurrentSnapshot`(공종 변경은 해당 주 이후에만 적용) · `loadCalEventsFromFirebase`(on('value') 하나만, 로컬 변이는 IS_DEMO일 때만) · `doLogin/enterApp/initApp` · `openUP/addUser/resetPassword`
- 반응형: 파일 끝쪽 `/* ══ 반응형 */` 블록(1180/900/700px). 태블릿은 `<head>` 의 `TABLET_SCALE`(viewport 폭을 줄여 확대, 현재 2)로 처리하고 본문 폭 제한 없음(v3.16에서 980px 제한 제거)
- 메인(단지별 취합) 화면은 현장 관제형(v3.13~): 남색 상단바+현황 띠(`renderOvBand`), 주차 선택 드롭다운은 상단바(`renderWeekSelect`/`pickWeek`), 단지 선택은 현황 띠의 타일만(v3.17 상단 단지 탭 제거, 메뉴는 현황판·주요 일정·도움말)

## 릴리스 절차(모든 변경 공통)
1. `index.html` 수정
2. `const VER = 'vX.Y  YYYY-MM-DD  HH:MM';` 갱신 + `<title>` 버전 갱신 (날짜는 시스템 시계)
3. `VERSION_HISTORY` 맨 위에 항목 추가 (`tag`: new|fix|change) — **문자열 안의 작은따옴표는 `\'` 로 이스케이프**(안 하면 스크립트 전체가 깨져 로그인 불가)
4. 메인 `<script>` 추출 후 `node --check` 로 문법 검사
5. 커밋·push → 30~60초 후 `curl "<사이트>/?t=$RANDOM" | grep "const VER"` 로 라이브 확인
6. 모바일/태블릿 변경은 Playwright로 390/820/1180px 렌더링 확인
- 토큰/비밀키를 HTML이나 저장소에 넣지 않는다(`grep ghp_ index.html` 은 0건이어야 함).

## 현재 상태 (v3.10, 2026-10-29)
- 완료: 단지별 취합(금주/차주 대비), 전주 복사 버튼, 글꼴 130%, 달력 클릭 일정 추가/수정(단지별 색·4건 초과 "+N건"), 분류·공종 순서/이름 편집, 사용자 관리(권한 변경·비번 초기화), 도움말·버전이력, 모바일/태블릿 반응형(v3.8, v3.8.1), Firebase Authentication 전환(v3.9), 인사정보 엑셀 일괄 등록·초기 비번 변경 강제(v3.10)
- 운영 상태: Authentication 이메일/비밀번호 사용 중, 실시간 DB 규칙은 `database.rules.json`(잠금) 게시 상태. 기존 계정 자동 전환은 사용 안 함(규칙상 `users` 읽기·`profiles` 쓰기 불가, `users` 가 희소 배열이라 `Array.isArray` 도 실패) → 2026-10-29 인사정보 엑셀로 77명 일괄 재등록.
- admin 계정: Auth 이메일은 관리자 업무 메일(콘솔에서 직접 생성, 기본 형식 아님), `usernames/61646d696e` 로 아이디 admin 에 연결.
- 일괄 등록 규칙(`parseHRRows`): 아이디=사번, 초기 비번=`initialPw`(사번, 6자 미만은 두 번), 세부직책·담당업무에 '공무' → admin 그 외 manager, 담당 현장=(겸) 아닌 본 소속 부서(N단지 / 부지조성→기반시설 / 통합현장→전체). 직원 정보는 저장소에 넣지 않는다(엑셀은 브라우저에서만 읽음).
- 기존 `users` 노드(base64 옛 비번)는 2026-10-29 삭제함. 남은 일: 소장급 권한(일괄 등록 시 매니저) 조정 여부 확인.
- 비밀번호 초기화 방식: 클라이언트에서 타인 비번을 못 바꾸므로 새 Auth 계정을 발급해 `usernames` 에 재연결.
- 미갱신: 사용설명서 Word 파일(v3.1 기준) — 요청 시 갱신.
