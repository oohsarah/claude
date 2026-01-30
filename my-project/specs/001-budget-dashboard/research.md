# Research: 예산 집행 현황 대시보드

**Feature**: 001-budget-dashboard
**Date**: 2026-01-30

## 1. 기술 스택 결정

### Backend Framework

**Decision**: FastAPI (Python 3.11)

**Rationale**:
- 비동기 지원으로 WebSocket 실시간 통신에 적합
- 자동 OpenAPI 문서 생성
- Pydantic 기반 데이터 검증으로 무결성 보장
- 데이터 분석 라이브러리(pandas, numpy)와 자연스러운 통합

**Alternatives Considered**:
- Django REST Framework: 기능이 풍부하나 비동기 지원이 제한적
- Flask: 경량이나 비동기 및 타입 힌팅 지원 부족
- Node.js/Express: Python 데이터 분석 생태계 활용 어려움

### Frontend Framework

**Decision**: React 18 + TypeScript

**Rationale**:
- 컴포넌트 기반 아키텍처로 대시보드 위젯 재사용 용이
- 풍부한 차트 라이브러리 생태계 (Recharts, Chart.js)
- TypeScript로 타입 안전성 확보

**Alternatives Considered**:
- Vue.js: 좋은 선택이나 React 생태계가 더 풍부
- Angular: 대시보드에 비해 과도한 복잡성
- Svelte: 생태계가 아직 성숙하지 않음

### 차트 라이브러리

**Decision**: Recharts

**Rationale**:
- React 네이티브 라이브러리로 통합 용이
- 선언적 API로 커스터마이징 간편
- 반응형 지원 내장
- 성능 최적화 (SVG 기반)

**Alternatives Considered**:
- Chart.js: 강력하나 React 래퍼 필요
- D3.js: 유연하나 학습 곡선 높음
- ECharts: 기능 풍부하나 번들 크기 큼

### Database

**Decision**: PostgreSQL 15

**Rationale**:
- 금융 데이터에 적합한 ACID 보장
- 시계열 쿼리 성능 우수
- 복잡한 집계 쿼리 지원
- 파티셔닝으로 대용량 데이터 처리

**Alternatives Considered**:
- MySQL: PostgreSQL 대비 분석 기능 제한
- TimescaleDB: 순수 시계열에 적합하나 범용성 부족
- MongoDB: 트랜잭션 일관성 보장 어려움

## 2. 실시간 업데이트 전략

### 통신 방식

**Decision**: WebSocket (Server-Sent Events 폴백)

**Rationale**:
- 양방향 통신으로 즉시 알림 전송 가능
- 5초 이내 데이터 반영 요구사항 충족
- 연결 상태 관리 용이

**Implementation**:
```
1. 클라이언트 WebSocket 연결
2. 서버에서 데이터 변경 감지 시 브로드캐스트
3. 연결 실패 시 SSE로 폴백
4. 재연결 로직 (지수 백오프)
```

**Alternatives Considered**:
- Polling: 서버 부하 증가, 지연 발생
- Long Polling: 복잡한 구현, 연결 관리 어려움
- GraphQL Subscriptions: 추가 인프라 필요

## 3. 예측 알고리즘

### 시계열 예측 모델

**Decision**: statsmodels ARIMA + 시나리오 분석

**Rationale**:
- 계절성과 추세를 모두 반영 가능
- 신뢰 구간 계산 내장
- Python 생태계에서 검증된 라이브러리
- 해석 가능한 결과 제공

**Implementation**:
```python
# 기본 예측
model = ARIMA(data, order=(p, d, q))
forecast = model.fit().forecast(steps=3)

# 시나리오 분석
optimistic = forecast * 0.9   # 낙관적
conservative = forecast * 1.1  # 보수적
realistic = forecast           # 현실적
```

**Confidence Intervals**:
- 80% 신뢰구간: 일반 계획용
- 95% 신뢰구간: 리스크 관리용

**Alternatives Considered**:
- Prophet (Facebook): 강력하나 의존성 무거움
- LightGBM: 머신러닝 기반으로 해석 어려움
- 단순 이동평균: 신뢰구간 계산 불가

## 4. 데이터 무결성 검증

### 검증 전략

**Decision**: 다층 검증 (DB + API + UI)

**Implementation**:
1. **DB 레벨**: 제약조건, 트리거
   - CHECK: 금액 >= 0
   - FK: 부서, 예산항목 참조 무결성
   - UNIQUE: 중복 집행 방지

2. **API 레벨**: Pydantic 검증
   - 필수 필드 검증
   - 타입 검증
   - 범위 검증

3. **비즈니스 로직**: 합계 검증
   - 집행 합계 == 개별 집행 합
   - 잔액 = 배정액 - 집행액

4. **UI 레벨**: 에러 표시
   - 검증 실패 시 즉시 토스트 알림
   - 데이터 불일치 시 경고 배너

## 5. 성능 최적화 전략

### 대시보드 로딩 최적화

**Target**: 초기 로딩 3초 이내

**Strategies**:
1. **데이터베이스**
   - 복합 인덱스: (fiscal_year, department_id)
   - 집계 테이블: 일별/월별 사전 집계
   - 쿼리 최적화: EXPLAIN ANALYZE 기반

2. **백엔드**
   - 응답 캐싱: Redis (TTL 10초)
   - 페이지네이션: 대량 데이터 분할
   - 지연 로딩: 차트 데이터 비동기 로드

3. **프론트엔드**
   - 코드 스플리팅: 차트 컴포넌트 지연 로드
   - 메모이제이션: useMemo, React.memo
   - 가상화: 대량 테이블 데이터 처리

## 6. 인증/연동

### 기존 시스템 연동

**Decision**: 기존 인증 시스템 SSO 연동 + 재무 시스템 API 연동

**Assumptions** (spec.md 기반):
- 조직 내부 네트워크 환경
- 기존 재무 시스템 API 제공
- SSO 인증 시스템 존재

**Implementation**:
- 인증: 기존 SSO 토큰 검증
- 재무 데이터: 배치 동기화 + 실시간 Webhook
