# Data Model: 예산 집행 현황 대시보드

**Feature**: 001-budget-dashboard
**Date**: 2026-01-30

## Entity Relationship Diagram

```
┌─────────────────┐       ┌─────────────────────┐
│   Department    │       │     BudgetItem      │
├─────────────────┤       ├─────────────────────┤
│ id (PK)         │◄──┐   │ id (PK)             │
│ code            │   │   │ code                │
│ name            │   │   │ name                │
│ parent_id (FK)  │───┘   │ allocated_amount    │
│ budget_manager  │   ┌──►│ department_id (FK)  │
└─────────────────┘   │   │ fiscal_year         │
        ▲             │   └─────────────────────┘
        │             │             │
        │             │             │
        └─────────────┼─────────────┤
                      │             ▼
┌─────────────────────┤   ┌─────────────────────┐
│   ForecastResult    │   │  ExecutionRecord    │
├─────────────────────┤   ├─────────────────────┤
│ id (PK)             │   │ id (PK)             │
│ budget_item_id (FK) │───│ budget_item_id (FK) │
│ forecast_period     │   │ execution_date      │
│ predicted_amount    │   │ amount              │
│ confidence_80_lower │   │ description         │
│ confidence_80_upper │   │ approved_by         │
│ confidence_95_lower │   │ created_at          │
│ confidence_95_upper │   └─────────────────────┘
│ scenario_type       │
│ created_at          │
└─────────────────────┘
```

## Entities

### 1. Department (부서)

조직의 부서 단위를 나타내며, 계층 구조를 지원한다.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK | 고유 식별자 |
| code | VARCHAR(20) | UNIQUE, NOT NULL | 부서 코드 (예: "FIN001") |
| name | VARCHAR(100) | NOT NULL | 부서명 |
| parent_id | UUID | FK(Department.id), NULL | 상위 부서 (NULL이면 최상위) |
| budget_manager | VARCHAR(100) | NULL | 예산 책임자 이름 |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | 생성 일시 |
| updated_at | TIMESTAMP | NOT NULL | 수정 일시 |

**Validation Rules**:
- code는 영문 대문자와 숫자 조합
- 자기 자신을 parent로 가질 수 없음
- 순환 참조 불가

---

### 2. BudgetItem (예산 항목)

예산 단위를 나타내며, 부서와 회계연도에 속한다.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK | 고유 식별자 |
| code | VARCHAR(20) | NOT NULL | 예산 코드 (예: "OP-2026-001") |
| name | VARCHAR(200) | NOT NULL | 예산 항목명 |
| allocated_amount | DECIMAL(15,2) | NOT NULL, >= 0 | 배정 금액 |
| department_id | UUID | FK(Department.id), NOT NULL | 소속 부서 |
| fiscal_year | INTEGER | NOT NULL | 회계 연도 (예: 2026) |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | 생성 일시 |
| updated_at | TIMESTAMP | NOT NULL | 수정 일시 |

**Validation Rules**:
- allocated_amount >= 0
- fiscal_year는 합리적 범위 (2000-2100)
- (code, fiscal_year) 조합은 UNIQUE

**Computed Fields** (API 레벨):
- executed_amount: 해당 항목의 집행 합계
- remaining_amount: allocated_amount - executed_amount
- execution_rate: (executed_amount / allocated_amount) * 100

---

### 3. ExecutionRecord (집행 내역)

개별 예산 집행 기록을 나타낸다.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK | 고유 식별자 |
| budget_item_id | UUID | FK(BudgetItem.id), NOT NULL | 예산 항목 |
| execution_date | DATE | NOT NULL | 집행 일자 |
| amount | DECIMAL(15,2) | NOT NULL, > 0 | 집행 금액 |
| description | VARCHAR(500) | NOT NULL | 집행 사유 |
| approved_by | VARCHAR(100) | NOT NULL | 승인자 |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | 생성 일시 |

**Validation Rules**:
- amount > 0 (음수 집행 불가)
- execution_date는 해당 예산 항목의 fiscal_year 범위 내
- 집행 합계가 배정 금액 초과 시 경고 (차단은 아님)

**Indexes**:
- (budget_item_id, execution_date): 항목별 기간 조회 최적화
- (execution_date): 일별 집계 최적화

---

### 4. ForecastResult (예측 결과)

예산 소요 예측 분석 결과를 저장한다.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK | 고유 식별자 |
| budget_item_id | UUID | FK(BudgetItem.id), NOT NULL | 예산 항목 |
| forecast_period | VARCHAR(7) | NOT NULL | 예측 대상 기간 (예: "2026-04") |
| predicted_amount | DECIMAL(15,2) | NOT NULL | 예상 집행액 |
| confidence_80_lower | DECIMAL(15,2) | NOT NULL | 80% 신뢰구간 하한 |
| confidence_80_upper | DECIMAL(15,2) | NOT NULL | 80% 신뢰구간 상한 |
| confidence_95_lower | DECIMAL(15,2) | NOT NULL | 95% 신뢰구간 하한 |
| confidence_95_upper | DECIMAL(15,2) | NOT NULL | 95% 신뢰구간 상한 |
| scenario_type | ENUM | NOT NULL | 시나리오 유형 |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | 생성 일시 |

**Scenario Types**:
- `REALISTIC`: 현실적 시나리오
- `OPTIMISTIC`: 낙관적 시나리오 (예상보다 적게 집행)
- `CONSERVATIVE`: 보수적 시나리오 (예상보다 많이 집행)

**Validation Rules**:
- confidence_80_lower <= predicted_amount <= confidence_80_upper
- confidence_95_lower <= confidence_80_lower
- confidence_95_upper >= confidence_80_upper

**Indexes**:
- (budget_item_id, forecast_period, scenario_type): 예측 조회 최적화

---

## Aggregation Tables (성능 최적화)

### DailyExecutionSummary

일별 집행 집계 테이블 (materialized view 또는 트리거 기반)

| Field | Type | Description |
|-------|------|-------------|
| summary_date | DATE | 집계 일자 |
| department_id | UUID | 부서 |
| total_executed | DECIMAL(15,2) | 일별 집행 합계 |
| record_count | INTEGER | 집행 건수 |

### MonthlyExecutionSummary

월별 집행 집계 테이블

| Field | Type | Description |
|-------|------|-------------|
| summary_month | VARCHAR(7) | 집계 월 (YYYY-MM) |
| budget_item_id | UUID | 예산 항목 |
| total_executed | DECIMAL(15,2) | 월별 집행 합계 |
| record_count | INTEGER | 집행 건수 |

---

## State Transitions

### BudgetItem Lifecycle

```
Created → Active → Closed
           │
           └──→ Suspended (예외 상황)
```

### ExecutionRecord Lifecycle

```
Pending → Approved → Recorded
             │
             └──→ Rejected
```

*Note: 현재 스펙에서는 상태 관리가 명시되지 않았으므로, 모든 레코드는 즉시 Recorded 상태로 생성된다고 가정*
