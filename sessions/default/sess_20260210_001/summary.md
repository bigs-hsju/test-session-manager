# 영업대행사 하위 법인 계약 3단 조회 API + 수수료 합계 노출

## 주요 변경사항
- **3단 계층 조회 API**: `GET /api/v1/agencies/corporation-contracts` - 영업대행사 → 법인 → 계약 단계별 조회
- **수수료 합계 노출**: 계약 상세/목록에서 영업대행사 수수료 합계 표시

## 변경 파일
- 신규 4개: Service interface/impl, Response DTO, AgencyContractMappingOutput
- 수정 8개: Controller, UseCase 2개, Repository 4개