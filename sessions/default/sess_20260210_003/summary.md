# 역할 권한 프리셋 시스템 구현

## 주요 변경사항
- **Infra 구현**: JPA Entity, Repository 구현체, Mapper, Querydsl 조회
- **Service**: RolePermissionPresetService (CRUD + 권한 검증)
- **Controller**: AdminRolePermissionPresetControllerV1 (REST API 5개)
- **User 전환**: `permissionIds` 제거 → `rolePermissionPresetId` 기반
- **인증 변경**: PayUserDetailsServiceImpl에서 프리셋 기반 권한 로딩
- **DDL**: 테이블 2개 + 마이그레이션 1개

## 변경 파일
- 신규 12개 (Entity, Repository, Mapper, Controller, Service, DDL)
- 수정 19개 (User 엔티티, UseCase, Controller, 인증 서비스 등)