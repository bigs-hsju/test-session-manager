# Git Auto Push 재테스트 - 상세 문서

## 수정 내용
- `standalone.ts`에서 `GitHubAuthService`를 초기화하고 `gitService.setAuthService()`를 호출하도록 수정
- `gitService` 초기화 시 `storagePath` 대신 `groupPath`를 사용하도록 수정

## 테스트 절차
1. MCP 서버 재시작
2. 새 세션 생성
3. saveDocuments로 문서 저장
4. updateSessionStatus로 completed 설정
5. git log / git status로 auto commit+push 확인