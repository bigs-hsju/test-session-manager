# 역할 권한 프리셋 시스템 구현

## 개요
Phase 1에서 정의한 도메인 모델을 기반으로 Infra/App 레이어 구현 및 기존 User.permissionIds → 프리셋 기반 시스템으로 전환.

## 구현 내용

### 1. Infra 레이어
- `RolePermissionPresetEntity` - JPA 엔티티 (@Version 낙관적 잠금, Auditing)
- `RolePermissionPresetItemEntity` - JPA 항목 엔티티
- `Mapper.kt` - Entity ↔ Domain 변환 함수 추가
- `JpaRolePermissionPresetRepository` - 저장소 구현체
- `JpaRolePermissionPresetQueryRepository` - Querydsl 조건 조회
- `JpaRolePermissionPresetItemRepository` - 항목 저장소 구현체
- `RolePermissionPresetEntityRepository` - Spring Data JPA
- `RolePermissionPresetItemEntityRepository` - Spring Data JPA

### 2. Domain Service
- `RolePermissionPresetService` - CRUD + permissionId 유효성 검증
  - create: 유효 권한만 필터링하여 저장
  - update: 기존 항목 삭제 후 재생성
  - delete: 항목 + 프리셋 함께 삭제
  - getPermissionIdsByPresetId: 프리셋의 권한 ID Set 반환

### 3. App 레이어
- `AdminRolePermissionPresetControllerV1` - REST API 5개 (CRUD + 목록)

### 4. User 전환 (핵심)
- `User.permissionIds` (MutableSet) 제거
- `User.rolePermissionPresetId` (Long?) 추가
- `updatePermissions()`, `updateUserAndPermissions()`, `hasPermission()` 메서드 제거
- `updateRolePermissionPresetId()` 메서드 추가
- `User.create()` 팩토리 메서드 파라미터 변경
- `CreateUserInput/UpdateUserInput` - permissionIds → rolePermissionPresetId
- `CreateUserRequest/UpdateUserRequest` - permissionIds → rolePermissionPresetId
- `CreateUserUseCase/UpdateUserUseCase` - 권한 처리 로직 간소화
- `BatchUserPermissionService` - 프리셋 기반으로 변경
- `UserPermissionsUpdatedEventHandler` - 이벤트 기반 권한 동기화 로직 제거

### 5. 인증 변경
- `PayUserDetailsServiceImpl` → `rolePermissionPresetId`로 프리셋 항목 조회 → `PayUserDetails.permissions`에 세팅
- 프리셋 없는 사용자는 빈 권한 Set

### 6. DDL
- `role_permission_preset.ddl.sql` - 프리셋 테이블
- `role_permission_preset_item.ddl.sql` - 프리셋 항목 테이블
- `user_permission_migration.ddl.sql` - users 테이블에 role_permission_preset_id 컬럼 추가

## 커밋 이력
- `3c44ecd52` feat: 역할 권한 프리셋 CRUD 구현
- `c68d18463` refactor: User.permissionIds 제거 및 프리셋 기반 권한 시스템 전환