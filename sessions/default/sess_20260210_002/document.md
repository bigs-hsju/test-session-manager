# Role Permission Preset 매핑 테이블 구현

## 개요
역할(Role)별 권한 프리셋을 관리하는 도메인 모델과 인터페이스를 정의하는 Phase 1 작업.

## 구현 내용

### 1. 도메인 모델
- `RolePermissionPreset` - 프리셋 본체 (id, tenantId, role, name, description, 감사 필드)
- `RolePermissionPresetItem` - 프리셋에 매핑된 개별 권한 항목

### 2. Repository 인터페이스
- `RolePermissionPresetRepository` - 기본 CRUD (findById, save, delete)
- `RolePermissionPresetQueryRepository` - 조건 조회 (tenantId, role 필터)
- `RolePermissionPresetItemRepository` - 항목 CRUD (findByPresetId, saveAll, deleteByPresetId)

### 3. 요청 DTO
- `CreateRolePermissionPresetRequest` - 생성 (tenantId, name, role, permissionIds, description, reason)
- `UpdateRolePermissionPresetRequest` - 수정 (name?, permissionIds?, description?, reason)

## 커밋
- `b1bc6f6f0` feat: 역할 권한 프리셋 도메인 모델 및 인터페이스 정의