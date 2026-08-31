# FOLDER STRUCTURE

## SkillGap.ai — Ready-to-Deploy Project Structure

---

## Complete Project Tree

```
SkillGap.ai/
│
├── README.md                          # Project overview
├── DEFINE.md                          # Define phase document
├── .gitignore                         # Git ignore rules
├── .env.example                       # Environment variable template (no secrets)
│
├── Design/                            # Design phase documents
│   ├── ARCHITECTURE.md
│   ├── FOLDER_STRUCTURE.md
│   ├── API_CONTRACTS.md
│   ├── DATA_MODELS.md
│   ├── COMPONENTS.md
│   └── KNOWLEDGE_BASE_DESIGN.md
│
├── backend/                           # FastAPI application
│   ├── Dockerfile                     # Container definition for App Runner
│   ├── requirements.txt               # Python dependencies (pinned versions)
│   ├── .env.example                   # Backend env var template
│   │
│   ├── main.py                        # FastAPI app entry point, router registration
│   ├── config.py                      # Settings loaded from environment variables
│   │
│   ├── routers/                       # API route handlers
│   │   ├── __init__.py
│   │   ├── auth.py                    # POST /auth/google, POST /auth/email, POST /auth/logout
│   │   ├── resume.py                  # POST /resume/upload
│   │   ├── analysis.py                # POST /analysis/start
│   │   ├── reform.py                  # POST /reform/start, POST /reform/rescore
│   │   ├── roadmap.py                 # POST /roadmap/generate
│   │   └── download.py                # POST /download/generate, GET /download/{file_id}
│   │
│   ├── services/                      # Core business logic
│   │   ├── __init__.py
│   │   ├── resume_parser.py           # PDF/DOCX text + section extraction
│   │   ├── ats_analyzer.py            # ATS scoring with RAG-grounded rules
│   │   ├── skill_gap_predictor.py     # JD skill extraction + gap classification
│   │   ├── resume_reformer.py         # LLM-powered resume content rewriting
│   │   ├── evidence_validator.py      # Anti-fabrication validation
│   │   ├── roadmap_generator.py       # Personalized learning roadmap generation
│   │   ├── rag_service.py             # KB chunking, embedding, pgvector retrieval
│   │   ├── embedding_service.py       # gemini-embedding-001 API wrapper
│   │   ├── document_generator.py      # PDF (WeasyPrint) + DOCX (python-docx) output
│   │   └── storage_service.py         # AWS S3 upload/download/presigned URLs
│   │
│   ├── models/                        # Pydantic request/response models
│   │   ├── __init__.py
│   │   ├── auth_models.py             # Login, token response models
│   │   ├── resume_models.py           # Resume upload, parsed resume models
│   │   ├── analysis_models.py         # ATS scores, skill gap, analysis result models
│   │   ├── reform_models.py           # Reform request/response, rescore models
│   │   ├── roadmap_models.py          # Roadmap structure models
│   │   └── download_models.py         # Download URL response models
│   │
│   ├── db/                            # Database layer
│   │   ├── __init__.py
│   │   ├── supabase_client.py         # Supabase client initialisation
│   │   ├── migrations/                # SQL migration scripts
│   │   │   ├── 001_create_users.sql
│   │   │   ├── 002_create_sessions.sql
│   │   │   └── 003_create_knowledge_base_chunks.sql
│   │   └── queries/                   # Reusable DB query functions
│   │       ├── __init__.py
│   │       ├── user_queries.py
│   │       ├── session_queries.py
│   │       └── kb_queries.py          # pgvector similarity search queries
│   │
│   ├── middleware/                    # FastAPI middleware
│   │   ├── __init__.py
│   │   ├── auth_middleware.py         # JWT validation via Supabase Auth
│   │   └── cors_middleware.py         # CORS configuration
│   │
│   ├── utils/                         # Shared utility functions
│   │   ├── __init__.py
│   │   ├── text_utils.py              # Text cleaning, chunking helpers
│   │   ├── file_utils.py              # File type detection, temp file handling
│   │   └── logger.py                  # Structured logging setup
│   │
│   └── knowledge_base/                # RAG knowledge base — markdown source files
│       ├── ats_formatting_rules.md    # ATS formatting standards and rules
│       ├── resume_section_guide.md    # How each resume section should be written
│       ├── scoring_rubrics.md         # Scoring criteria per ATS sub-category
│       ├── strong_weak_examples.md    # Before/after language examples
│       ├── action_verbs.md            # Strong action verbs by category
│       └── skill_evidence_guide.md    # How skills should be evidenced in resume
│
├── frontend/                          # Static HTML/CSS/JS frontend
│   ├── index.html                     # Single page application entry point
│   │
│   ├── css/
│   │   ├── main.css                   # Global styles, CSS variables, typography
│   │   ├── components.css             # Reusable UI component styles
│   │   ├── layout.css                 # Page layout, grid, responsive breakpoints
│   │   └── animations.css             # Loading states, transitions
│   │
│   ├── js/
│   │   ├── app.js                     # Main app entry, page state machine
│   │   ├── api.js                     # All fetch() calls to backend API
│   │   ├── auth.js                    # Google OAuth + email login flow
│   │   ├── upload.js                  # File upload handling, drag and drop
│   │   ├── analysis.js                # Render ATS scores, skill gap display
│   │   ├── reform.js                  # Trigger reform, show before/after
│   │   ├── roadmap.js                 # Render learning roadmap
│   │   └── download.js                # Trigger PDF/DOCX download
│   │
│   └── assets/
│       ├── logo.svg                   # SkillGap.ai logo
│       └── icons/                     # UI icons (SVG)
│
├── scripts/                           # Utility scripts
│   ├── seed_knowledge_base.py         # Chunk + embed KB markdown files into pgvector
│   ├── test_gemini.py                 # Quick smoke test for Gemini API connectivity
│   └── test_supabase.py               # Quick smoke test for Supabase connectivity
│
├── tests/                             # Backend tests
│   ├── __init__.py
│   ├── conftest.py                    # pytest fixtures
│   ├── test_resume_parser.py
│   ├── test_ats_analyzer.py
│   ├── test_skill_gap_predictor.py
│   ├── test_resume_reformer.py
│   ├── test_evidence_validator.py
│   └── test_roadmap_generator.py
│
└── .github/
    └── workflows/
        ├── deploy-backend.yml         # Build Docker → push ECR → deploy App Runner
        └── deploy-frontend.yml        # Upload frontend to S3 static site
```

---

## Key File Descriptions

### backend/main.py
FastAPI app entry point. Registers all routers, initialises middleware, triggers KB seeding check on startup.

### backend/config.py
Loads all environment variables using Pydantic `BaseSettings`. Single source of truth for config. Never hardcode secrets anywhere else.

### backend/services/rag_service.py
Central RAG orchestrator. On startup, checks if KB is already embedded in pgvector. If not, reads markdown files, chunks them, embeds via `embedding_service.py`, and stores in Supabase. At query time, performs cosine similarity search and returns top-k chunks.

### backend/services/evidence_validator.py
Takes the original resume text and a list of AI-generated statements. For each statement, checks whether it can be grounded in the original resume content. Rejects any statement that introduces information not present in the source.

### backend/db/migrations/
Raw SQL files run once against Supabase to create tables. Applied manually or via a migration script. Numbered sequentially.

### scripts/seed_knowledge_base.py
Standalone script run once at deployment (or on demand) to populate the pgvector knowledge base. Can also be triggered by `main.py` startup if the table is empty.

### frontend/js/app.js
Manages the single-page flow state machine:
```
STATE: login → upload → analyzing → results → reforming → comparison → roadmap → download
```

### .github/workflows/deploy-backend.yml
On push to `main`:
1. Build Docker image
2. Push to AWS ECR
3. Update AWS App Runner service to use new image

### .github/workflows/deploy-frontend.yml
On push to `main`:
1. Sync `frontend/` folder to S3 bucket
2. Optionally invalidate CloudFront cache

---

## Environment Variables

All secrets are stored in environment variables. Never committed to the repository.

### backend/.env.example

```env
# Google AI
GOOGLE_AI_API_KEY=your_google_ai_api_key_here

# Supabase
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your_supabase_anon_key_here
SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_role_key_here

# AWS S3
AWS_ACCESS_KEY_ID=your_aws_access_key_here
AWS_SECRET_ACCESS_KEY=your_aws_secret_key_here
AWS_S3_BUCKET_NAME=skillgap-ai-uploads
AWS_REGION=us-east-1

# App
APP_ENV=production
FRONTEND_URL=https://your-frontend-s3-url.com
JWT_SECRET=your_jwt_secret_here
```

### AWS App Runner
Environment variables are injected via AWS Secrets Manager — not stored in the container image.

---

## Docker Setup

### backend/Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies for pdfplumber and WeasyPrint
RUN apt-get update && apt-get install -y \
    libpango-1.0-0 \
    libpangocairo-1.0-0 \
    libgdk-pixbuf2.0-0 \
    libffi-dev \
    shared-mime-info \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Download spaCy model
RUN python -m spacy download en_core_web_sm

COPY . .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## Deployment Checklist

```
[ ] Supabase project created
[ ] Run SQL migrations (db/migrations/)
[ ] Seed knowledge base (scripts/seed_knowledge_base.py)
[ ] AWS S3 bucket created (static frontend + file uploads)
[ ] AWS ECR repository created
[ ] AWS App Runner service configured
[ ] GitHub Actions secrets configured:
    [ ] GOOGLE_AI_API_KEY
    [ ] SUPABASE_URL
    [ ] SUPABASE_ANON_KEY
    [ ] SUPABASE_SERVICE_ROLE_KEY
    [ ] AWS_ACCESS_KEY_ID
    [ ] AWS_SECRET_ACCESS_KEY
    [ ] AWS_S3_BUCKET_NAME
    [ ] AWS_REGION
[ ] CORS_ORIGINS updated with frontend S3 URL
[ ] Push to main branch → CI/CD deploys automatically
```
