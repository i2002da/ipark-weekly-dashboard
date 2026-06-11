# Gist 연동 설정 가이드

## 1단계 — `gist` 권한 포함 토큰 발급

1. https://github.com/settings/tokens 접속
2. **Generate new token (classic)** 클릭
3. Note: `ipark-weekly-gist`
4. Expiration: No expiration (또는 원하는 기간)
5. Select scopes:
   - ☑ **gist** ← 반드시 체크
6. **Generate token** → 토큰 복사

## 2단계 — 대시보드에서 자동 Gist 생성

1. https://i2002da.github.io/ipark-weekly-dashboard/ 접속
2. 발급한 토큰 입력 후 **접속하기**
3. 최초 접속 시 현장별 Gist 6개 자동 생성됨

## 3단계 — 팀원 공유

- 팀원에게 동일한 토큰 배포
- 모든 팀원이 같은 Gist에 연결 → 실시간 공유

## 데모 모드

토큰 없이 **데모 모드로 보기** 클릭 시  
샘플 데이터로 모든 기능 체험 가능 (저장 안 됨)
