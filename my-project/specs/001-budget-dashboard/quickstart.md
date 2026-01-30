# Quickstart: 예산 집행 현황 대시보드

**Feature**: 001-budget-dashboard
**Date**: 2026-01-30

## Prerequisites

- Python 3.11+
- Node.js 18+
- PostgreSQL 15+
- Git

## Quick Setup

### 1. Clone and Setup

```bash
# 프로젝트 클론
git clone <repository-url>
cd my-project

# 브랜치 체크아웃
git checkout 001-budget-dashboard
```

### 2. Backend Setup

```bash
# 백엔드 디렉토리 이동
cd backend

# 가상환경 생성 및 활성화
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 의존성 설치
pip install -r requirements.txt

# 환경 변수 설정
cp .env.example .env
# .env 파일을 편집하여 DATABASE_URL 등 설정

# 데이터베이스 마이그레이션
alembic upgrade head

# 개발 서버 실행
uvicorn src.main:app --reload --port 8000
```

### 3. Frontend Setup

```bash
# 새 터미널에서 프론트엔드 디렉토리 이동
cd frontend

# 의존성 설치
npm install

# 환경 변수 설정
cp .env.example .env.local
# API_URL=http://localhost:8000 등 설정

# 개발 서버 실행
npm run dev
```

### 4. Access Application

- Frontend: http://localhost:5173
- Backend API: http://localhost:8000
- API Docs: http://localhost:8000/docs

## Environment Variables

### Backend (.env)

```env
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/budget_dashboard

# Security
SECRET_KEY=your-secret-key-here

# CORS
ALLOWED_ORIGINS=http://localhost:5173

# External System (Optional)
FINANCE_SYSTEM_API_URL=https://finance.example.com/api
SSO_PROVIDER_URL=https://sso.example.com
```

### Frontend (.env.local)

```env
VITE_API_URL=http://localhost:8000/api/v1
VITE_WS_URL=ws://localhost:8000/ws
```

## Database Setup

### Create Database

```sql
CREATE DATABASE budget_dashboard;
CREATE USER budget_user WITH PASSWORD 'your-password';
GRANT ALL PRIVILEGES ON DATABASE budget_dashboard TO budget_user;
```

### Sample Data (Development)

```bash
# 개발용 샘플 데이터 로드
cd backend
python scripts/seed_data.py
```

## Running Tests

### Backend Tests

```bash
cd backend

# 전체 테스트 실행
pytest

# 특정 테스트 실행
pytest tests/unit/
pytest tests/integration/
pytest tests/contract/

# 커버리지 리포트
pytest --cov=src --cov-report=html
```

### Frontend Tests

```bash
cd frontend

# 전체 테스트 실행
npm test

# 워치 모드
npm run test:watch

# 커버리지 리포트
npm run test:coverage
```

## API Documentation

개발 서버 실행 후 다음 URL에서 API 문서 확인:

- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc
- OpenAPI JSON: http://localhost:8000/openapi.json

## Project Structure

```
my-project/
├── backend/
│   ├── src/
│   │   ├── models/         # SQLAlchemy 모델
│   │   ├── services/       # 비즈니스 로직
│   │   ├── api/            # FastAPI 라우터
│   │   └── main.py
│   ├── tests/
│   ├── alembic/            # DB 마이그레이션
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── components/     # React 컴포넌트
│   │   ├── pages/
│   │   ├── services/       # API 클라이언트
│   │   └── hooks/
│   ├── tests/
│   └── package.json
└── specs/
    └── 001-budget-dashboard/
        ├── spec.md
        ├── plan.md
        ├── research.md
        ├── data-model.md
        ├── contracts/
        └── quickstart.md   # This file
```

## Common Issues

### Database Connection Error

```
Connection refused: localhost:5432
```

**Solution**: PostgreSQL 서비스 실행 확인
```bash
# Linux/Mac
sudo systemctl start postgresql

# Mac (Homebrew)
brew services start postgresql
```

### CORS Error

```
Access to XMLHttpRequest blocked by CORS policy
```

**Solution**: Backend .env에서 ALLOWED_ORIGINS 확인

### WebSocket Connection Failed

```
WebSocket connection to 'ws://...' failed
```

**Solution**:
1. Backend 서버 실행 확인
2. 방화벽 설정 확인
3. Frontend의 WS_URL 설정 확인

## Next Steps

1. 기본 설정 완료 후 `/speckit.tasks`로 구현 태스크 생성
2. User Story 1 (P1)부터 순차적으로 구현
3. 각 스토리 완료 시 테스트 실행
