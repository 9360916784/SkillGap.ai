# SkillGap.ai

> **AI-Powered Resume Intelligence Platform**
> Analyze. Reform. Validate. Learn.

---

## What is SkillGap.ai?

SkillGap.ai is an AI-powered resume intelligence platform that helps candidates close the gap between their current resume and their target job description.

It doesn't just score your resume — it understands it, identifies what's missing, rewrites it without fabricating anything, validates every change, and generates a personalized learning roadmap for skills you need to develop.

The core philosophy:

```
Don't just score the resume.

Understand it.
     ↓
Analyze it.
     ↓
Improve it.
     ↓
Verify it.
     ↓
Measure it.
     ↓
Learn from it.
```

---

## The Problem

Candidates fail ATS screening not because they lack skills, but because their resume doesn't communicate those skills effectively.

Common issues:

- Poor ATS-friendly formatting
- Missing keywords from the job description
- Weak, vague project and experience descriptions
- Generic statements with no measurable achievements
- Skills present but not evidenced clearly
- No understanding of which skills are missing for a target role

Existing tools tell you what's wrong. SkillGap.ai fixes it.

---

## How it Works

```
User provides:
  1. Resume (PDF or DOCX)
  2. Target Job Description (pasted text)

System pipeline:
  Resume Parsing
       ↓
  Resume Analysis
  ├── ATS Analysis       ← grounded in ATS knowledge base (RAG)
  ├── Skill Analysis
  └── Content Analysis
       ↓
  Skill Gap Detection
       ↓
  AI Resume Reformer     ← grounded in resume best practices (RAG)
       ↓
  Evidence Validation (Anti-Fabrication)
       ↓
  Improved Resume
       ↓
  Before vs After Comparison
       ↓
  Personalized Learning Roadmap
```

---

## Key Features

### Resume Parser
Extracts structured information from PDF and DOCX files:
- Name, contact information
- Education, certifications
- Skills, tools, technologies
- Projects, internships, experience
- Achievements and professional summary

### ATS Analyzer
Calculates a detailed ATS score with sub-scores:

| Category    | Description                                      |
| :---------- | :----------------------------------------------- |
| Keywords    | Match between resume content and job description |
| Skills      | Technical and domain skills coverage             |
| Formatting  | ATS-readable structure and layout                |
| Experience  | Strength of experience descriptions              |
| Projects    | Quality and specificity of project descriptions  |
| Overall     | Weighted composite score                         |

Scores are based on a structured ATS rules knowledge base, not arbitrary LLM judgment. The LLM retrieves relevant rules before scoring, making results consistent and explainable.

### RAG-Powered Knowledge Base
To reduce LLM hallucination and ensure consistent, grounded output, the system maintains a structured knowledge base of:

- ATS formatting rules and standards
- Resume section best practices
- Scoring rubrics per category
- Strong vs weak language examples
- Industry-standard resume structure guidelines

These are stored as **markdown documents**, chunked, embedded, and stored in a vector database. Before any analysis or reformation, the LLM retrieves the most relevant rules from this knowledge base — so its output is always anchored to defined standards, not just general training knowledge.

```
Markdown Knowledge Base (.md files)
            ↓
   Chunked + Embedded at startup
            ↓
   Stored in Vector Store (pgvector)
            ↓
   Retrieved at query time (semantic search)
            ↓
   Injected into LLM prompt as context
            ↓
   Grounded, consistent, explainable output
```

### Skill Gap Predictor
Compares skills required by the job description against skills evidenced in the resume:

```
Status: Present          — Clearly evidenced in resume
Status: Weakly Evidenced — Mentioned but not demonstrated
Status: Not Found        — Not present in resume
```

> Note: "Not Found" does not mean the candidate doesn't have the skill. It means the resume doesn't sufficiently evidence it.

### AI Resume Reformer
Rewrites weak resume sections to be clearer, more impactful, and ATS-optimized. The reformer always references the knowledge base for best practices before generating any content.

Example:

|            | Version                                                                          |
| ---------- | -------------------------------------------------------------------------------- |
| **Before** | Worked on AWS project.                                                           |
| **After**  | Developed and deployed a cloud-based application using AWS services (EC2, S3).   |

The AI improves action verbs, sentence structure, technical clarity, and conciseness — without inventing experience that isn't there.

### Anti-Fabrication Validation
Every AI-generated statement is validated against the original resume content.

The system will never:
- Add fake projects or experience
- Claim tools or technologies not mentioned
- Inflate job titles or responsibilities
- Generate certifications not earned

```
Original Resume Evidence
         ↓
   Retrieved Best Practices (RAG)
         ↓
      AI Reformation
         ↓
   Evidence Validation
         ↓
   Approved Content only
```

### Before vs After Comparison
After reformation, the system re-analyzes the resume and shows measurable improvement:

```
Before:  ATS Score 64%
After:   ATS Score 87%
         ─────────────
         +23% Improvement
```

### Learning Roadmap
Generates a personalized, structured learning path for skills that are missing or weakly evidenced:

```
Skill Gap: Kubernetes
     ↓
  Pods and Deployments
  Services and Networking
  Helm Charts
     ↓
Skill Gap: Terraform
     ↓
  HCL Syntax
  Providers and Resources
  State Management
```

Each roadmap entry includes priority level, learning sequence, and beginner-to-advanced progression.

---

## Tech Stack

| Domain               | Technology                        | Purpose                                                              |
| :------------------- | :-------------------------------- | :------------------------------------------------------------------- |
| **Frontend**         | HTML + CSS + JavaScript           | Upload UI, results dashboard, before/after view, roadmap display     |
| **Backend**          | Python + FastAPI                  | API layer, resume processing pipeline, LLM orchestration             |
| **Document Parsing** | pdfplumber + python-docx          | Text and structure extraction from PDF and DOCX                      |
| **NLP**              | spaCy                             | Section detection, entity recognition, keyword extraction            |
| **LLM**              | Google Gemini Pro (Gemini API)    | Resume reformation, skill extraction, roadmap generation             |
| **Embeddings**       | Google gemini-embedding-001       | Embedding knowledge base docs and query vectors (same API key)       |
| **Vector Store + DB**| Supabase (PostgreSQL + pgvector)  | Stores resume data, analysis results, and knowledge base vectors     |
| **Knowledge Base**   | Markdown files + pgvector         | ATS rules, best practices, scoring rubrics — embedded at startup     |
| **File Storage**     | AWS S3                            | Resume file uploads (PDF/DOCX)                                       |
| **Frontend Hosting** | AWS S3 Static Website             | Hosts the HTML/CSS/JS frontend                                       |
| **Backend Hosting**  | AWS EC2 (t2.micro) + Docker       | Runs the FastAPI container                                           |
| **CI/CD**            | GitHub Actions                    | Automated build and deployment pipeline                              |

---

## Cloud Architecture (AWS Free Tier Compatible)

```
User Browser
     │
     ▼
AWS S3 (Static Frontend)
     │
     ▼ API calls
AWS EC2 t2.micro — Docker container (FastAPI)
     │
     ├──► AWS S3 (Resume file storage)
     │
     ├──► Supabase (PostgreSQL + pgvector)
     │         ├── Resume data and analysis results
     │         └── Knowledge base vectors (ATS rules, best practices)
     │
     ├──► Google Gemini API (LLM — Gemini Pro + Embeddings — gemini-embedding-001)
```

Everything runs on hosted/managed services. There are no local-only components. The system is fully functional after deployment.

### Free Tier Summary

| Service         | Free Tier                                    |
| :-------------- | :------------------------------------------- |
| AWS EC2         | t2.micro, 750 hours/month (12 months)        |
| AWS S3          | 5 GB storage, 20K GET, 2K PUT requests/month |
| Supabase        | 500 MB database, 50K embeddings              |
| Google AI API   | Free tier — Gemini Pro + gemini-embedding-001|
| GitHub Actions  | 2,000 minutes/month for free                 |

---

## Project Scope

### In Scope
- Resume parsing (PDF and DOCX)
- ATS scoring with sub-score breakdown
- RAG-powered grounded analysis using a markdown knowledge base
- Keyword and skills analysis
- Skill gap detection against a target job description
- AI-powered resume content reformation (grounded in best practices)
- Anti-fabrication evidence validation
- Before and after ATS comparison
- Improved resume generation and download
- Personalized learning roadmap

### Out of Scope
- Job searching or job matching
- Job recommendations
- Automatic job applications
- Recruiter-side candidate ranking
- Any fabrication of experience, projects, skills, or certifications
- Local-only deployments (everything must work on cloud)

---

## Target Users

- College students preparing for placements
- Fresh graduates entering the job market
- Entry-level and mid-level job seekers
- Career switchers targeting a new domain
- Internship applicants

---

## Success Criteria

The system is considered successful when it can:

1. Parse resumes accurately from PDF and DOCX
2. Identify and extract all standard resume sections
3. Calculate meaningful, explainable ATS sub-scores grounded in a knowledge base
4. Extract required skills from any job description
5. Correctly classify skills as present, weakly evidenced, or not found
6. Reform resume content with clear, measurable improvements
7. Validate every AI-generated statement against original content
8. Demonstrate a measurable before/after ATS score improvement
9. Generate a structured, actionable learning roadmap
10. Explain every recommendation and score to the user
11. Run fully on cloud infrastructure after deployment (no local dependencies)

---

## Development Phases

```
DEFINE      — Problem definition, objectives, scope           ✅ Complete
DESIGN      — Architecture, data models, API contracts        ✅ Complete
DEVELOPMENT — Implementation by component                     ⬜ Pending
DEPLOYMENT  — Docker, AWS App Runner, S3, CI/CD pipeline      ⬜ Pending
```

---

## Project Name

**SkillGap.ai — AI Resume ATS Analyzer + Intelligent Resume Reformer + Skill Gap Predictor**
