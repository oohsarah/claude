# 예산 통제프로세스 수립 Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-01-30

## Active Technologies

| Category | Technology | Version |
|----------|------------|---------|
| Backend Language | Python | 3.11 |
| Backend Framework | FastAPI | latest |
| Frontend Language | TypeScript | 5.x |
| Frontend Framework | React | 18 |
| Database | PostgreSQL | 15+ |
| ORM | SQLAlchemy | 2.x |
| Charts | Recharts | latest |
| Testing (Backend) | pytest | latest |
| Testing (Frontend) | Vitest | latest |

## Project Structure

```text
my-project/
├── backend/
│   ├── src/
│   │   ├── models/          # SQLAlchemy 모델
│   │   ├── services/        # 비즈니스 로직
│   │   ├── api/             # FastAPI 라우터
│   │   └── main.py
│   ├── tests/
│   │   ├── contract/
│   │   ├── integration/
│   │   └── unit/
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── components/      # React 컴포넌트
│   │   ├── pages/
│   │   ├── services/        # API 클라이언트
│   │   └── hooks/
│   ├── tests/
│   └── package.json
├── specs/                   # Feature specifications
│   └── 001-budget-dashboard/
└── memory/
    └── constitution.md      # Project principles
```

## Commands

### Backend (Python/FastAPI)

```bash
# 개발 서버 실행
cd backend && uvicorn src.main:app --reload --port 8000

# 테스트 실행
cd backend && pytest

# 마이그레이션
cd backend && alembic upgrade head
```

### Frontend (React/TypeScript)

```bash
# 개발 서버 실행
cd frontend && npm run dev

# 테스트 실행
cd frontend && npm test

# 빌드
cd frontend && npm run build
```

## Code Style

### Python
- Black formatter (line length: 88)
- isort for imports
- Type hints required for all functions
- Docstrings for public functions

### TypeScript
- ESLint + Prettier
- Strict mode enabled
- Prefer functional components with hooks
- Type definitions required (no `any`)

## Constitution Principles

이 프로젝트는 다음 핵심 원칙을 준수해야 합니다:

1. **신속 정확한 분석**: 데이터 수집부터 분석 결과까지 최소 지연, 정확도 검증 필수
2. **예측 기반 의사결정**: 과거 패턴 기반 예측, 신뢰 구간 포함, 시나리오 분석 지원

## Recent Features

### 001-budget-dashboard (2026-01-30)
- 예산 집행 현황 실시간 대시보드
- 시각화 차트 (원형, 막대, 추세선)
- 예산 소요 예측 분석

<!-- MANUAL ADDITIONS START -->
<!-- Add project-specific guidelines here -->
<!-- MANUAL ADDITIONS END -->
