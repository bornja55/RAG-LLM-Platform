# Backend - FastAPI Application

This directory will contain the FastAPI backend application.

## Structure (Planned)

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI app entry point
│   ├── config.py            # Configuration
│   ├── models/              # SQLAlchemy models
│   ├── schemas/             # Pydantic schemas
│   ├── api/                 # API routes
│   │   ├── v1/
│   │   │   ├── auth.py
│   │   │   ├── workspaces.py
│   │   │   ├── documents.py
│   │   │   └── query.py
│   ├── services/            # Business logic
│   ├── db/                  # Database
│   │   ├── base.py
│   │   └── session.py
│   └── utils/               # Utilities
├── tests/                   # Tests
├── alembic/                 # Database migrations
├── requirements.txt         # Dependencies
├── requirements-dev.txt     # Dev dependencies
└── README.md
```

## Setup (Coming Soon)

```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run development server
uvicorn app.main:app --reload
```

## Technology Stack

- **Framework**: FastAPI 0.109+
- **ORM**: SQLAlchemy 2.0
- **Database**: PostgreSQL 16
- **Vector DB**: Qdrant
- **Cache**: Redis
- **AI/ML**: LangChain, Gemini API
- **Authentication**: JWT

## Documentation

Once running, API docs will be available at:
- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`
