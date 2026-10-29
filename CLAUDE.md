# 천안IPC 통합현장 주간업무현황 대시보드 — Claude Code 작업 가이드

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
- 반응형: 파일 끝쪽 `/* ══ 반응형 */` 블록(1180/900/700px) + `태블릿 폭 제한`(701~1400px, 본문 최대 980px)

## 릴리스 절차(모든 변경 공통)
1. `index.html` 수정
2. `const VER = 'vX.Y  YYYY-MM-DD  HH:MM';` 갱신 + `<title>` 버전 갱신 (날짜는 시스템 시계)
3. `VERSION_HISTORY` 맨 위에 항목 추가 (`tag`: new|fix|change) — **문자열 안의 작은따옴표는 `\'` 로 이스케이프**(안 하면 스크립트 전체가 깨져 로그인 불가)
4. 메인 `<script>` 추출 후 `node --check` 로 문법 검사
5. 커밋·push → 30~60초 후 `curl "<사이트>/?t=$RANDOM" | grep "const VER"` 로 라이브 확인
6. 모바일/태블릿 변경은 Playwright로 390/820/1180px 렌더링 확인
- 토큰/비밀키를 HTML이나 저장소에 넣지 않는다(`grep ghp_ index.html` 은 0건이어야 함).

## 현재 상태 (v3.9, 2026-10-29)
- 완료: 단지별 취합(금주/차주 대비), 전주 복사 버튼, 글꼴 130%, 달력 클릭 일정 추가/수정(단지별 색·4건 초과 "+N건"), 분류·공종 순서/이름 편집, 사용자 관리(권한 변경·비번 초기화), 도움말·버전이력, 모바일/태블릿 반응형(v3.8, v3.8.1), Firebase Authentication 전환 코드(v3.9)
- **남은 일(사용자 콘솔 작업)**: ① Firebase 콘솔 Authentication → 이메일/비밀번호 사용 설정 ② 전 사용자 1회 로그인(자동 전환; 관리자 사용자 관리 창에서 미전환 안내 확인) ③ `database.rules.json` 을 실시간 DB 규칙에 게시. 이 전까지는 규칙이 `{".read":true,".write":true}` 로 열려 있음(롤백 시 이 값으로 복원).
- 비밀번호 초기화 방식: 클라이언트에서 타인 비번을 못 바꾸므로 새 Auth 계정을 발급해 `usernames` 에 재연결.
- 미갱신: 사용설명서 Word 파일(v3.1 기준) — 요청 시 갱신.
