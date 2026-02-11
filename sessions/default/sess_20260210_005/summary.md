# 정산 마감 통계 V2 API 개발 요약

## 목표
- 정산 마감 통계 v2 페이지용 API를 두 가지 데이터 소스 버전으로 개발
- 영업대행사(Agency) 수수료를 동적 AgencyPolicyItem 기반으로 계산

## 구현 결과

### API 엔드포인트
| 버전 | 경로 | 데이터 소스 |
|------|------|-----------|
| V1 | `GET /api/v1/closing-statistics/v2/cost` | TransactionCostSettlement |
| V2 | `GET /api/v1/closing-statistics/v2/closing` | DepartmentClosingFee |

### 계층 구조
테넌트 > 영업대행사 > 법인 > PG(계약) 으로 일별 매출/정산/수수료 집계

### 핵심 기능
- 영업대행사 수수료 동적 계산 (AgencyPolicyItem별 feeRate 적용)
- 프론트엔드 체크박스 연동용 항목별 금액 사전 계산
- N+1 방지를 위한 bulk 조회 (findAllByIds)

## 생성/수정 파일 (12개)
- Domain: Query, Projection, Port (V1/V2) - 4개
- Infra: Adapter (V1/V2), AgencyRepositoryImpl - 3개
- App: Controller, Request, Response, Service (V1/V2) - 5개
- Docs: settlement.md 섹션 9 추가