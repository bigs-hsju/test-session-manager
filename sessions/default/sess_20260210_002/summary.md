# Role Permission Preset 매핑 테이블 구현

## 주요 변경사항
- **도메인 모델**: `RolePermissionPreset`, `RolePermissionPresetItem` 모델 정의
- **Repository 인터페이스**: CRUD + 조건 조회 인터페이스 3개 정의
- **DTO**: Create/Update 요청 DTO 정의

## 변경 파일
- 신규 7개: Domain 모델 2개, Repository 인터페이스 3개, DTO 2개