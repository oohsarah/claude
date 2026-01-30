# Implementation Plan: 예산 집행 현황 대시보드

**Branch**: `001-budget-dashboard` | **Date**: 2026-01-30 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-budget-dashboard/spec.md`

## Summary

예산 집행 현황을 실시간으로 조회하고 시각화하는 웹 대시보드 구현. Python FastAPI 백엔드와 React 프론트엔드로 구성하며, PostgreSQL에 데이터를 저장하고 WebSocket으로 실시간 업데이트를 제공한다. 시계열 분석 기반의 예측 기능을 포함한다.

## Technical Context

**Language/Version**: Python 3.11 (Backend), TypeScript 5.x (Frontend)
**Primary Dependencies**: FastAPI, SQLAlchemy, React, Recharts, pandas, statsmodels
**Storage**: PostgreSQL 15+
**Testing**: pytest (Backend), Vitest (Frontend)
**Target Platform**: 웹 브라우저 (Chrome, Firefox, Safari, Edge 최신 버전)
**Project Type**: Web Application (Frontend + Backend)
**Performance Goals**: 대시보드 초기 로딩 3초 이내, 데이터 갱신 5초 이내
**Constraints**: 내부 네트워크 환경, 기존 재무 시스템 연동 필요
**Scale/Scope**: 동시 사용자 100명, 예산 항목 1,000개, 월 집행 건수 10,000건

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| 원칙 | 검증 항목 | 상태 | 구현 방안 |
|------|----------|------|----------|
| **Principle 1: 신속 정확한 분석** | 최소 지연 시간 | ✅ Pass | WebSocket 실시간 업데이트, 인덱싱 최적화 |
| | 정확도 검증 | ✅ Pass | 데이터 무결성 체크 API, 합계 검증 로직 |
| | 오류 감지 및 알림 | ✅ Pass | 에러 바운더리, 토스트 알림 시스템 |
| | 데이터 무결성 검증 | ✅ Pass | DB 제약조건, 트랜잭션 관리 |
| **Principle 2: 예측 기반 의사결정** | 과거 패턴 반영 | ✅ Pass | 시계열 분석 (ARIMA/Prophet) |
| | 신뢰 구간 포함 | ✅ Pass | 예측 결과에 80%, 95% 신뢰구간 표시 |
| | 시나리오 분석 | ✅ Pass | 낙관/보수/현실 시나리오 모델링 |
| | 예측 정확도 검증 | ✅ Pass | 백테스트 기능, 정확도 리포트 |

**Gate Status**: ✅ PASSED - 모든 헌법 원칙 충족

## Project Structure

### Documentation (this feature)

```text
specs/001-budget-dashboard/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/           # Phase 1 output
│   └── openapi.yaml
├── checklists/
│   └── requirements.md
└── tasks.md             # Phase 2 output (/speckit.tasks)
```

### Source Code (repository root)

```text
backend/
├── src/
│   ├── models/          # SQLAlchemy 모델
│   │   ├── budget_item.py
│   │   ├── execution_record.py
│   │   ├── department.py
│   │   └── forecast_result.py
│   ├── services/        # 비즈니스 로직
│   │   ├── budget_service.py
│   │   ├── forecast_service.py
│   │   └── integrity_service.py
│   ├── api/             # FastAPI 라우터
│   │   ├── budgets.py
│   │   ├── executions.py
│   │   ├── forecasts.py
│   │   └── websocket.py
│   └── main.py
├── tests/
│   ├── contract/
│   ├── integration/
│   └── unit/
└── requirements.txt

frontend/
├── src/
│   ├── components/      # React 컴포넌트
│   │   ├── Dashboard/
│   │   ├── Charts/
│   │   ├── Filters/
│   │   └── Alerts/
│   ├── pages/
│   │   └── DashboardPage.tsx
│   ├── services/        # API 클라이언트
│   │   └── api.ts
│   ├── hooks/           # Custom hooks
│   │   ├── useBudgetData.ts
│   │   └── useWebSocket.ts
│   └── App.tsx
├── tests/
└── package.json
```

**Structure Decision**: Web Application 구조 선택. 대시보드 특성상 프론트엔드 시각화가 핵심이며, 백엔드는 데이터 제공 및 예측 분석을 담당한다.

## Complexity Tracking

> 헌법 위반 사항 없음 - 이 섹션은 해당 없음

## Implementation Phases

### Phase 0: Research (Completed)
- 기술 스택 결정 완료
- 예측 알고리즘 선정 (statsmodels ARIMA)
- 실시간 업데이트 방식 결정 (WebSocket)

### Phase 1: Design & Contracts (Current)
- 데이터 모델 정의 → `data-model.md`
- API 계약 정의 → `contracts/openapi.yaml`
- 개발 환경 설정 → `quickstart.md`

### Phase 2: Tasks (Next - /speckit.tasks)
- 구현 태스크 분해 → `tasks.md`
