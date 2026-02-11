# Git Push 자동화 테스트 - 상세 문서

## 목적
- session-manager-v3의 team 그룹 git 자동 push 기능 검증

## 테스트 절차
1. `git-test` 그룹을 활성화
2. 새 세션 생성 (`sess_20260211_001`)
3. `saveDocuments`로 summary.md, document.md 저장
4. 자동 git add + commit + push 확인

## 기대 결과
- 세션 파일이 로컬 git 저장소에 commit되고
- origin/main에 자동으로 push됨