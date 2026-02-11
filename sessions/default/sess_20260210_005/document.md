# 정산 마감 통계 V2 API 구현 문서

## 1. 배경

정산 마감 통계 v2 페이지는 **테넌트 > 영업대행사 > 법인 > PG(계약)** 계층 구조로 일별 매출/정산/수수료를 보여주는 대시보드다. 두 가지 데이터 소스를 비교 분석할 수 있도록 별도 API를 제공한다.

## 2. 아키텍처 설계

### 2.1 공통 도메인 인터페이스 (Interface-First)

```
ClosingStatisticsV2Query.Search   ← 공통 쿼리
ClosingStatisticsV2Projection     ← 공통 Projection
ClosingStatisticsV2Response       ← 공통 응답 DTO
```

V1/V2 각각 별도 Port 인터페이스:
- `ClosingStatisticsV2ByCostPort` (V1)
- `ClosingStatisticsV2ByClosingPort` (V2)

### 2.2 Hexagonal Architecture 적용

```
Domain Layer (Port Interface)
    ↓
Infra Layer (Adapter - Querydsl 구현)
    ↓
App Layer (Service - 비즈니스 로직, Controller - REST API)
```

## 3. 데이터 소스 비교

### V1: TransactionCostSettlement 기반
- 매출액: `SUM(transactionTotalAmount)`
- 정산금: `SUM(pgOutflowAmount)`
- PG수수료: `SUM(pgInflowAmount)`
- 조건: `transactionDate BETWEEN startDate AND endDate`

### V2: DepartmentClosingFee 기반
- 매출액: `SUM(baseAmount)`
- 정산금: `SUM(baseAmount - feeAmount)`
- PG수수료: `SUM(feeAmount)`
- 조건: `closureDate BETWEEN`, `costId = 'COST01'`
- PgContract JOIN으로 corporationId 매핑

## 4. 영업대행사 수수료 계산

### 계산 흐름
1. `AgencyContractMapping`으로 contractId → agencyId 매핑
2. `Agency.policyItems` (동적 수수료 정책) 조회
3. 기본금 = salesAmount - pgSourceFee
4. 각 policyItem별: fee = 기본금 × feeRate / 100
5. 프론트엔드는 체크박스로 항목별 가감

### PolicyInfo 제공
- policyId, index, description, required, feeRate
- `required=true`인 항목은 기본 체크 상태

## 5. N+1 방지

### 문제
- 영업대행사 조회 시 `agencyIds.forEach { findById(it) }` 패턴 → N+1

### 해결
- `AgencyRepository.findAllByIds(ids)` 추가
- 내부적으로 `findAllById`, `findAllByAgencyIdIn` bulk 쿼리 사용
- 결과를 `associateBy { it.id }` Map으로 변환

## 6. 코드 리뷰 이슈 및 수정

| 이슈 | 위치 | 수정 내용 |
|------|------|----------|
| 중복 어노테이션 | ByCostAdapter | `@Component` 제거 (`@Repository` 충분) |
| N+1 | ByCostQueryService | `findById` → `findAllByIds` |
| N+1 | ByClosingQueryService | `findById` → `findAllByIds` |

## 7. 파일 목록

### Domain (4개)
- `settlement/query/ClosingStatisticsV2Query.kt`
- `settlement/model/query/ClosingStatisticsV2Projection.kt`
- `settlement/port/ClosingStatisticsV2ByCostPort.kt`
- `settlement/port/ClosingStatisticsV2ByClosingPort.kt`

### Infra (3개)
- `settlement/adapter/ClosingStatisticsV2ByCostAdapter.kt`
- `settlement/adapter/ClosingStatisticsV2ByClosingAdapter.kt`
- `agency/repository/AgencyRepositoryImpl.kt` (findAllByIds 추가)

### App (5개)
- `controller/settlement/ClosingStatisticsV2ControllerV1.kt`
- `controller/settlement/request/ClosingStatisticsV2Request.kt`
- `controller/settlement/response/ClosingStatisticsV2Response.kt`
- `application/settlement/service/ClosingStatisticsV2ByCostQueryService.kt`
- `application/settlement/service/ClosingStatisticsV2ByClosingQueryService.kt`

### 문서 (1개)
- `.claude/docs/domains/settlement.md` (섹션 9 추가)