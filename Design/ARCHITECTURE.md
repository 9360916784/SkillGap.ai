# ARCHITECTURE

## SkillGap.ai — System Architecture

---

## 1. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER BROWSER                            │
│                  (HTML + CSS + JavaScript)                       │
│                    Hosted on AWS S3                             │
└───────────────────────────┬─────────────────────────────────────┘
                            │ HTTPS API calls
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                      AWS APP RUNNER                             │
│              Docker Container — FastAPI (Python)                │
│                                                                 │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐ │
│   │ Auth Router  │  │Resume Router │  │  Analysis Router     │ │
│   └──────────────┘  └──────────────┘  └──────────────────────┘ │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐ │
│   │ Reform Router│  │Roadmap Router│  │  Download Router     │ │
│   └──────────────┘  └──────────────┘  └──────────────────────┘ │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                  CORE SERVICES LAYER                    │   │
│   │  ResumeParser │ ATSAnalyzer │ SkillGapPredictor        │   │
│   │  ResumeReformer │ EvidenceValidator │ RoadmapGenerator  │   │
│   │  RAGService │ EmbeddingService │ DocumentGenerator     │   │
│   └─────────────────────────────────────────────────────────┘   │
└──────┬──────────────┬────────────────┬──────────────────────────┘
       │              │                │
       ▼              ▼                ▼
┌────────────┐ ┌─────────────┐ ┌──────────────────────────────┐
│  AWS S3    │ │  Supabase   │ │     Google AI API            │
│            │ │             │ │                              │
│ - Resume   │ │ - Users     │ │ - gemini-1.5-pro (LLM)      │
│   uploads  │ │ - Sessions  │ │ - gemini-embedding-001       │
│ - Static   │ │ - pgvector  │ │   (Embeddings)               │
│   frontend │ │   (KB vecs) │ │                              │
└────────────┘ └─────────────┘ └──────────────────────────────┘
```

---

## 2. Request Flow — Resume Analysis

```
User uploads Resume + Job Description
              │
              ▼
POST /api/resume/upload
  → Store PDF/DOCX in AWS S3
  → Return file_key
              │
              ▼
POST /api/analysis/start
  → Parse resume (pdfplumber / python-docx)
  → Extract structured sections (spaCy)
  → Embed query against KB (gemini-embedding-001 + pgvector)
  → ATS Analysis (Gemini Pro + retrieved KB rules)
  → Skill Gap Detection (Gemini Pro + JD)
  → Return analysis_id + full analysis result
              │
              ▼
POST /api/reform/start
  → Receive analysis_id + original resume text
  → Retrieve best practices from KB (RAG)
  → Generate reformed content (Gemini Pro)
  → Evidence validation check
  → Return reformed_sections
              │
              ▼
POST /api/reform/rescore
  → Re-run ATS analysis on reformed resume
  → Return before_score + after_score + delta
              │
              ▼
POST /api/roadmap/generate
  → Receive skill gaps from analysis
  → Generate personalized roadmap (Gemini Pro)
  → Return structured roadmap
              │
              ▼
POST /api/download/generate
  → Compose final resume from reformed sections
  → Generate PDF (WeasyPrint)
  → Generate DOCX (python-docx)
  → Upload to S3 with signed URLs
  → Return download_url_pdf + download_url_docx
```

---

## 3. RAG Pipeline

```
STARTUP (once at deployment):
  ┌─────────────────────────────┐
  │  Markdown KB files          │
  │  /knowledge_base/*.md       │
  └────────────┬────────────────┘
               │ chunk (512 tokens, 50 overlap)
               ▼
  ┌─────────────────────────────┐
  │  gemini-embedding-001       │
  │  → embed each chunk         │
  └────────────┬────────────────┘
               │ store vectors
               ▼
  ┌─────────────────────────────┐
  │  Supabase pgvector table    │
  │  knowledge_base_chunks      │
  └─────────────────────────────┘

QUERY TIME (per analysis request):
  User resume + JD context
               │
               ▼
  Embed query → gemini-embedding-001
               │
               ▼
  pgvector cosine similarity search
  → top 5 relevant KB chunks
               │
               ▼
  Inject chunks into Gemini Pro prompt
  as [RULES] context block
               │
               ▼
  Grounded, consistent LLM output
```

---

## 4. Authentication Flow

```
User visits app
       │
       ├──► Google OAuth → Supabase Auth → JWT token
       │
       └──► Email/Password → Supabase Auth → JWT token

JWT token stored in browser (httpOnly cookie or localStorage)
       │
       ▼
Every API request includes Authorization: Bearer <jwt>
       │
       ▼
FastAPI middleware validates JWT via Supabase Auth API
       │
       ├──► Valid → attach user_id to request → proceed
       └──► Invalid → 401 Unauthorized
```

---

## 5. Cloud Deployment Architecture

```
GitHub Repository
       │
       ▼ push to main
GitHub Actions CI/CD Pipeline
       │
       ├──► Build Docker image
       ├──► Run tests
       ├──► Push image to AWS ECR
       └──► Deploy to AWS App Runner
                    │
                    ▼
            AWS App Runner
            (auto-scaling, HTTPS, no server management)
                    │
                    ├──► reads env vars from AWS Secrets Manager
                    ├──► connects to Supabase (external)
                    └──► connects to Google AI API (external)

Static Frontend
       │
GitHub Actions → build → upload to AWS S3 → served via S3 static website
                                │
                                ▼
                         AWS CloudFront (optional CDN layer)
```

---

## 6. Service Responsibilities

| Service | Responsibility |
| :--- | :--- |
| **FastAPI** | API routing, request validation, orchestrating core services |
| **ResumeParser** | Extract text + sections from PDF/DOCX |
| **ATSAnalyzer** | Calculate ATS sub-scores using KB rules + Gemini Pro |
| **SkillGapPredictor** | Extract JD skills, compare against resume, classify gaps |
| **ResumeReformer** | Rewrite weak sections using RAG-grounded Gemini Pro |
| **EvidenceValidator** | Validate reformed content against original resume |
| **RoadmapGenerator** | Generate personalized learning roadmap via Gemini Pro |
| **RAGService** | Manage KB chunking, embedding, retrieval from pgvector |
| **EmbeddingService** | Interface to gemini-embedding-001 API |
| **DocumentGenerator** | Produce final PDF and DOCX from reformed resume content |
| **Supabase** | PostgreSQL DB + pgvector + Auth |
| **AWS S3** | Resume file storage + static frontend hosting |
| **AWS App Runner** | Managed container hosting for FastAPI |
| **GitHub Actions** | CI/CD pipeline |

---

## 7. Technology Decisions Summary

| Decision | Choice | Reason |
| :--- | :--- | :--- |
| LLM | Gemini 1.5 Pro | Already have API access, capable, free tier |
| Embeddings | gemini-embedding-001 | Same API key as LLM, no extra provider |
| Vector store | Supabase pgvector | Combines DB + vector store, free tier |
| Auth | Supabase Auth | Built-in Google OAuth + email, same instance |
| Backend | FastAPI | Async, Pydantic validation, known by team |
| Frontend | HTML + CSS + JS | No build step, static S3 hosting |
| Container hosting | AWS App Runner | Managed, auto-scale, free tier, no EC2 management |
| File storage | AWS S3 | Free tier 5GB, presigned URLs for secure access |
| CI/CD | GitHub Actions | Free tier 2000 min/month, native Docker support |
| PDF generation | WeasyPrint | Python-native, HTML-to-PDF, no external service |
| DOCX generation | python-docx | Python-native, no external service |
