# 작업 이력 및 결정 사항 (상세)

`CLAUDE.md`의 보충 문서. 사용자 요청 순서대로 무엇을·왜·어떻게 바꿨는지와 시행착오를 기록한다.

## 1. 요청 이력 (시간순)
1. **화면 정리 (v2.x 초반)** — H1~H4 글자 크기·서식 차이 부여 / 각 단지 화면 상단의 "인허가, 정상" 삭제 / 단지명 앞의 "착공, 착공예정, 기반시설, DX TFT" 글씨 삭제 / 전체 단지를 보여주는 화면 삭제 / 각 단지 취합 화면을 **금주 진행사항 ↔ 차주 예정사항 2열 대비**로 변경
2. **전주 복사·글꼴** — 금주 작성 시 전주 차주 예정사항을 복사해 오는 버튼(`copyFromPrevWeek`, `prevWeekInfo`, `fetchSiteDataForWeek`) / 차주 작성 시 금주 내용 복사 버튼(`copyFromCur`) / 전체 글꼴 130% 확대(CSS 12~29px)
3. **주요일정** — 일정추가 버튼 대신 달력 날짜 클릭으로 추가, 일정 클릭으로 수정 / 대외·대내·DX·공정 등 분류 항목 편집(`event_types`, Firebase 저장) / 공종 추가·변경은 **해당 주 이후에만** 적용(주차별 스냅샷 `categories`, 기준키 `1900-W01`)
   - 확인 질문 답변: Firebase에 저장해 영구 동기화 / 위 해석 그대로 진행
4. **일정 중복 저장 버그(v2.9.1)** — 저장 시 2건씩 생기던 원인: 실시간 리스너가 반영하는데 로컬 배열에도 push. 해결: `on('value')` 하나만 진실 공급원, 로컬 변이는 `IS_DEMO` 일 때만
5. **달력 표시** — 일정 앞에 (3단지)/(통합)/(4단지) 등 단지 표시, 단지별 색상 구분 / 기본 화면을 이번 달로
6. **공종 순서 변경** — 탭 드래그 이동(`reorderCat`, `initTabDragDrop`), 변경은 현재 주부터
7. **사용설명서** — Word 매뉴얼 작성(`ipark_weekly_사용설명서_v3.1.docx`, docx@9.6.1, v3.1 기준·이후 미갱신) → 대시보드 내 도움말(`HELP_SECTIONS`)과 버전 이력(`VERSION_HISTORY`)으로도 추가
8. **로그인 장애** — 다른 PC에서 로그인 불가. 최초 진단(방화벽)은 오진(v3.3 배너·재시도 코드는 무해하지만 원인 아님). **진짜 원인: 버전이력 문자열의 이스케이프 안 된 작은따옴표 → 스크립트 전체 SyntaxError**(v3.3.1에서 `\'` 처리)
9. **관리자 기능** — 타 사용자 비밀번호 초기화(v3.4) / 사용자 권한 변경(v3.5)
10. **버전 관리 자동화** — 변경 때마다 도움말 버전이력 추가 + 화면 버전 갱신. 날짜 오류(실제 시스템 날짜와 1년 이상 차이) 수정(v3.5.1, 시스템 시계 사용)
11. **달력 일정 표시 개수** — 날짜당 4건까지 보이고 초과분은 "+N건"(v3.6)
12. **Firebase 만료 메일 대응** — 클라이언트 액세스 만료 안내 메일을 받고 규칙 게시 방법 문의. 임시 조치: `{"rules":{"ipark_weekly_v1":{".read":true,".write":true}}}` 게시(날짜 만료 없음, 보안은 약함) → 인증 기반 강화 질문
13. **v3.7** — "데모 모드로 보기" 버튼 삭제 / 금주 진행사항 초기 입력(예정 문구) 삭제 후 "금주 진행 사항을 입력하세요" 안내문 / 공종 옆 숫자 삭제. 이후 라이브 반영 여부 확인 → 로컬에만 있던 변경을 발견해 배포
14. **v3.8 / v3.8.1 반응형** — 폰·패드에서 비율 맞춤. 기존엔 `@media` 0개(viewport 메타만 있음). 아이패드 에어에서 너무 넓다는 피드백 → 701~1400px에서 본문 최대 980px 가운데 정렬
15. **v3.9 Firebase Authentication 전환(B안)** — 아래 3절 참조

## 2. 핵심 설계 메모
- 상단바 grid(auto 1fr auto) 높이 58px. 900px 이하는 2단 배치, 페이지 스크롤형(편집창 위아래 적층, `#gj-ed-wrap` 높이 고정 해제)
- 달력: 7열 `minmax(0,1fr)`, 폰에서 셀 68px·글자 10px
- 모달: `showModal(inner,onSave)`, `.modal-bg` z-index 700(사용자 관리 창 600 위)
- 편집기: `setHeading()` 은 execCommand 가 아닌 직접 DOM 조작, `ceExec`, turndown/marked 로 md 변환
- 공종 스냅샷: `CATS_CACHE[site][weekKey]=[공종…]`, `getCats(site,refWeekKey)` 는 기준 주 이하의 가장 최근 스냅샷, `ensureCurrentSnapshot` 은 메모리만 변경
- 계정 기본값(전환 전): admin/admin1234, manager1/pass1234, viewer1/view1234

## 3. Firebase Authentication 전환(v3.9) 상세
- 선택 사유: A안(익명 인증+`auth!=null`)은 사실상 보안 효과 없음 → B안(이메일/비밀번호) 채택
- 아이디→이메일: `u<hex(UTF-8 아이디)>@ipark.local` (admin → `u61646d696e@ipark.local`). 비밀번호 초기화 시 `u<hex>r<time36>@ipark.local` 새 계정을 만들고 `usernames/<hex>` 를 재연결, 이전 `profiles/<oldUid>` 삭제
- 관리자가 다른 사용자 계정을 만들 때는 보조 앱(`firebase.initializeApp(cfg,'secondary')`) 사용 → 관리자 세션 유지
- 기존 계정 자동 전환: 로그인 실패 시 `users` 배열의 `btoa(pw)` 일치하면 Auth 계정 생성 + `profiles`/`usernames` 기록 + 기존 pw 삭제. 6자 미만 비번은 전환 불가(관리자 초기화 필요)
- 호환 모드: `auth/configuration-not-found`(콘솔에서 Authentication 미사용) 시 `legacyLogin` 으로 기존 방식 로그인, 사용자 관리 아이콘 숨김
- 세션: Firebase Auth 영속 세션 + `localStorage.ipark_session {uid,ts}` 8시간 만료
- 로그아웃: `off()` 로 리스너 해제, 캐시 초기화, `signOut`
- 규칙(`database.rules.json`): `usernames` 공개 읽기·관리자 쓰기 / `profiles` 본인 읽기·관리자 전체 읽기쓰기 / `sites,categories,calendar,event_types` 로그인 읽기·admin·manager 쓰기. 기존 `users` 노드는 규칙에 없어 잠금 후 접근 불가(의도)
- 테스트: 가짜 Firebase(메모리 DB/Auth)로 마이그레이션·재로그인 복원·사용자 추가·비번 초기화·본인 비번 변경·호환 모드 통과. **실제 Firebase 에서의 검증은 아직 안 함**(Authentication 미사용 상태에서 `auth/configuration-not-found` 응답을 확인했을 뿐)
- 완전 초기화 절차(기존 비번 일괄 삭제): 방법 A(admin 만 남기고 `users/0/pw` 를 새 비번 base64 로 교체 후 로그인 시 자동 전환) / 방법 B(콘솔에서 admin Auth 계정과 `profiles`, `usernames` 직접 생성)

## 4. 시행착오와 교훈
- 문자열 안 작은따옴표 미이스케이프 = 전체 앱 마비 → 배포 전 `node --check` 필수
- 원인 불명 장애에서 네트워크 탓으로 단정하지 말 것(먼저 콘솔 에러/문법 확인)
- 로컬 작업본과 라이브가 다를 수 있음 → 시작 시 라이브를 내려받아 작업
- 패치 시 대상 문자열이 실제로 존재하는지 확인(`count==1` assert)
- 환경 메모: 세션 환경에서는 GitHub API 토큰 방식이 403으로 막혀 `add_repo`(push 권한)로 저장소를 연결해 `git clone/push` 로 배포했다. 클로드 코드에서도 저장소 접근 권한 연결 후 git push 로 배포하면 된다.

## 5. 남은 작업 / 후보
1. (사용자) Firebase 콘솔: Authentication 이메일/비밀번호 사용 설정 → 전 사용자 1회 로그인 → `database.rules.json` 게시
2. 실제 Firebase 환경에서 v3.9 로그인·전환·사용자 관리 검증(문제 시 규칙을 `{".read":true,".write":true}` 로 롤백)
3. 사용설명서 Word 파일을 최신 버전(v3.9)으로 갱신(요청 시)
4. 폰에서는 공종 탭 드래그 순서 변경 불가(터치에서 HTML5 드래그 미지원) — 필요하면 위/아래 버튼 방식 추가
