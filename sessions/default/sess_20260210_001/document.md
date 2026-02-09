# 영업대행사 하위 법인 계약 3단 조회 API + 수수료 합계 노출

## 개요
영업대행사 관리 화면에서 영업대행사 → 법인 → 계약을 단계별로 선택할 수 있는 3단 계층 조회 API와, 계약 수수료 노출 시 영업대행사 수수료 합계를 함께 보여주는 기능을 구현했다.

## 1. 3단 계층 조회 API

### 엔드포인트
```
GET /api/v1/agencies/corporation-contracts
```

### 파라미터
| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| agencyId | String | N | 영업대행사 ID |
| corporationId | String | N | 법인 ID |

### 동작 방식
1. **파라미터 없음** → `agencies` 목록만 반환
2. **agencyId만** → `agencies` + 해당 영업대행사에 매핑된 `corporations` 반환
3. **agencyId + corporationId** → `agencies` + `corporations` + 해당 법인의 `contracts` 반환

### 응답 구조
```kotlin
data class AgencyCorporationContractResponse(
    val agencies: List<AgencySummary>,
    val corporations: List<CorporationSummary>?,
    val contracts: List<ContractSummary>?,
)
```

### 데이터 흐름
```
Agency → AgencyContractMapping → PgContract → Corporation
         (agencyId로 조회)      (contractId로 조회)  (corporationId distinct 후 조회)
```

### 생성 파일
- `AgencyCorporationContractQueryService.kt` - interface
- `AgencyCorporationContractQueryServiceImpl.kt` - 구현체
- `AgencyCorporationContractResponse.kt` - 응답 DTO (AgencySummary, CorporationSummary, ContractSummary 내장)

### 수정 파일
- `AgencyControllerV1.kt` - 엔드포인트 추가

## 2. 계약 수수료 영업대행사 수수료 합계 노출

### 변경 내용
계약 상세 조회(`GetPgContractDetailUseCase`)와 계약 목록 조회(`GetPgContractsForManageViewUseCase`)에서 해당 계약에 매핑된 영업대행사의 수수료 합계를 함께 반환하도록 수정.

### 데이터 흐름
```
PgContract.id → AgencyContractMapping (contractId 매핑)
             → Agency (agencyId로 조회)
             → AgencyPolicyItem (agencyId로 수수료 항목 조회)
             → feeRate 합계 계산
```

### 생성 파일
- `AgencyContractMappingOutput.kt` - 영업대행사-계약 매핑 조회 결과 DTO

### 수정 파일
- `GetPgContractDetailUseCase.kt` - 계약 상세에 수수료 합계 추가
- `GetPgContractsForManageViewUseCase.kt` - 계약 목록에 수수료 합계 추가
- `AgencyRepository.kt` - 인터페이스에 메서드 추가
- `AgencyRepositoryImpl.kt` - 구현체
- `JpaAgencyContractMappingRepository.kt` - JPA 쿼리 추가
- `JpaAgencyEntityRepository.kt` - JPA 쿼리 추가
- `JpaAgencyPolicyItemRepository.kt` - JPA 쿼리 추가

## 커밋 이력
- `fc1cdc803` feat: 영업대행사 하위 법인 계약 조회
- `6631fae74` feat: 계약 수수료 노출 시 영업대행사 수수료 합계 노출