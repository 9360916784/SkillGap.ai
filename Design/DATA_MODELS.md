# DATA MODELS

## SkillGap.ai — Database Schema and Pydantic Models

---

## 1. Supabase Database Schema

### Overview

```
users                    ← managed by Supabase Auth
sessions                 ← one per analysis run (stateless)
knowledge_base_chunks    ← RAG vector store
```

Sessions are stateless — each analysis run is independent. There is no persistent history stored between runs. The session row exists only for the duration of the current request cycle (linking file upload to analysis to reform to download).

---

### Table: users

Managed automatically by Supabase Auth. Extended with a `profiles` table for any custom fields.

```sql
-- Supabase Auth manages auth.users internally.
-- We extend it with a public profiles table.

CREATE TABLE public.profiles (
    id          UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
    email       TEXT NOT NULL,
    full_name   TEXT,
    avatar_url  TEXT,
    created_at  TIMESTAMPTZ DEFAULT NOW(),
    updated_at  TIMESTAMPTZ DEFAULT NOW()
);

-- Auto-create profile on user signup
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO public.profiles (id, email, full_name, avatar_url)
    VALUES (
        NEW.id,
        NEW.email,
        NEW.raw_user_meta_data->>'full_name',
        NEW.raw_user_meta_data->>'avatar_url'
    );
    RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
    AFTER INSERT ON auth.users
    FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();
```

---

### Table: sessions

One row per analysis session. Stateless — deleted or expired after download.

```sql
CREATE TABLE public.sessions (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id             UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    file_key            TEXT NOT NULL,              -- S3 key of uploaded resume
    file_name           TEXT NOT NULL,
    file_type           TEXT NOT NULL CHECK (file_type IN ('pdf', 'docx')),
    job_description     TEXT NOT NULL,
    status              TEXT NOT NULL DEFAULT 'uploaded'
                            CHECK (status IN (
                                'uploaded',
                                'parsing',
                                'analyzed',
                                'reforming',
                                'reformed',
                                'downloading',
                                'completed',
                                'failed'
                            )),
    error_message       TEXT,
    created_at          TIMESTAMPTZ DEFAULT NOW(),
    expires_at          TIMESTAMPTZ DEFAULT NOW() + INTERVAL '24 hours'
);

CREATE INDEX idx_sessions_user_id ON public.sessions(user_id);
CREATE INDEX idx_sessions_status ON public.sessions(status);
```

---

### Table: knowledge_base_chunks

Stores chunked and embedded markdown knowledge base documents for RAG retrieval.

```sql
-- Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE public.knowledge_base_chunks (
    id              BIGSERIAL PRIMARY KEY,
    source_file     TEXT NOT NULL,              -- e.g. 'ats_formatting_rules.md'
    chunk_index     INTEGER NOT NULL,           -- chunk number within the file
    content         TEXT NOT NULL,              -- raw chunk text
    embedding       VECTOR(768) NOT NULL,       -- gemini-embedding-001 output dimension
    metadata        JSONB DEFAULT '{}',         -- category, tags, etc.
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- IVFFlat index for fast cosine similarity search
CREATE INDEX idx_kb_chunks_embedding
    ON public.knowledge_base_chunks
    USING ivfflat (embedding vector_cosine_ops)
    WITH (lists = 50);

CREATE INDEX idx_kb_chunks_source ON public.knowledge_base_chunks(source_file);
```

---

### pgvector Similarity Search Query

```sql
-- Find top 5 most relevant KB chunks for a query embedding
SELECT
    id,
    source_file,
    content,
    metadata,
    1 - (embedding <=> $1::vector) AS similarity_score
FROM
    public.knowledge_base_chunks
ORDER BY
    embedding <=> $1::vector
LIMIT 5;
```

---

## 2. Pydantic Models

### auth_models.py

```python
from pydantic import BaseModel, EmailStr
from typing import Optional


class GoogleAuthRequest(BaseModel):
    id_token: str


class EmailSignupRequest(BaseModel):
    email: EmailStr
    password: str  # min 8 chars enforced in validator


class EmailLoginRequest(BaseModel):
    email: EmailStr
    password: str


class UserResponse(BaseModel):
    id: str
    email: str
    name: Optional[str] = None
    avatar_url: Optional[str] = None


class AuthResponse(BaseModel):
    access_token: str
    token_type: str = "bearer"
    user: UserResponse
```

---

### resume_models.py

```python
from pydantic import BaseModel
from typing import Optional, List


class UploadResponse(BaseModel):
    file_key: str
    file_name: str
    file_type: str  # 'pdf' or 'docx'
    file_size_kb: float


class EducationEntry(BaseModel):
    degree: str
    institution: str
    year: Optional[str] = None


class ExperienceEntry(BaseModel):
    title: str
    company: str
    duration: Optional[str] = None
    bullets: List[str] = []


class ProjectEntry(BaseModel):
    name: str
    description: str
    tech_stack: List[str] = []


class ParsedResume(BaseModel):
    name: Optional[str] = None
    email: Optional[str] = None
    phone: Optional[str] = None
    linkedin: Optional[str] = None
    github: Optional[str] = None
    summary: Optional[str] = None
    education: List[EducationEntry] = []
    skills: List[str] = []
    experience: List[ExperienceEntry] = []
    projects: List[ProjectEntry] = []
    certifications: List[str] = []
    achievements: List[str] = []
```

---

### analysis_models.py

```python
from pydantic import BaseModel
from typing import Optional, List
from enum import Enum


class SkillStatus(str, Enum):
    present = "present"
    weakly_evidenced = "weakly_evidenced"
    not_found = "not_found"


class IssueSeverity(str, Enum):
    high = "high"
    medium = "medium"
    low = "low"


class ATSScores(BaseModel):
    overall: int          # 0-100
    keywords: int
    skills: int
    formatting: int
    experience: int
    projects: int


class ATSIssue(BaseModel):
    category: str         # 'keywords' | 'skills' | 'formatting' | 'experience' | 'projects'
    severity: IssueSeverity
    issue: str            # human-readable description
    section: Optional[str] = None  # which part of the resume


class SkillGap(BaseModel):
    skill: str
    status: SkillStatus
    evidence: Optional[str] = None   # quote or note if present/weak


class AnalysisRequest(BaseModel):
    file_key: str
    job_description: str


class AnalysisResult(BaseModel):
    session_id: str
    parsed_resume: "ParsedResume"    # from resume_models
    ats_scores: ATSScores
    ats_issues: List[ATSIssue]
    skill_gaps: List[SkillGap]
    raw_resume_text: str
```

---

### reform_models.py

```python
from pydantic import BaseModel
from typing import Optional, List


class ReformedBullet(BaseModel):
    original: str
    reformed: str
    validated: bool
    rejection_reason: Optional[str] = None


class ReformedExperience(BaseModel):
    title: str
    company: str
    duration: Optional[str]
    original_bullets: List[str]
    reformed_bullets: List[str]
    validated: bool


class ReformedProject(BaseModel):
    name: str
    original: str
    reformed: str
    validated: bool


class ReformedSections(BaseModel):
    summary: Optional[dict] = None      # {original, reformed, validated}
    experience: List[ReformedExperience] = []
    projects: List[ReformedProject] = []


class ValidationSummary(BaseModel):
    total_statements: int
    approved: int
    rejected: int
    rejected_reasons: List[str] = []


class ReformRequest(BaseModel):
    session_id: str
    raw_resume_text: str
    ats_issues: List["ATSIssue"]
    skill_gaps: List["SkillGap"]
    job_description: str


class ReformResult(BaseModel):
    reformed_sections: ReformedSections
    validation_summary: ValidationSummary


class RescoreRequest(BaseModel):
    session_id: str
    reformed_resume_text: str
    job_description: str


class ScoreDelta(BaseModel):
    overall: int
    keywords: int
    skills: int
    formatting: int
    experience: int
    projects: int


class RescoreResult(BaseModel):
    before: "ATSScores"
    after: "ATSScores"
    delta: ScoreDelta
```

---

### roadmap_models.py

```python
from pydantic import BaseModel
from typing import List
from enum import Enum


class Priority(str, Enum):
    high = "high"
    medium = "medium"
    low = "low"


class Level(str, Enum):
    beginner = "beginner"
    intermediate = "intermediate"
    advanced = "advanced"


class RoadmapStage(BaseModel):
    stage: int
    title: str
    level: Level
    topics: List[str]
    practice: str


class SkillRoadmap(BaseModel):
    skill: str
    priority: Priority
    status: str           # from SkillStatus enum
    estimated_weeks: int
    stages: List[RoadmapStage]


class RoadmapRequest(BaseModel):
    session_id: str
    skill_gaps: List["SkillGap"]
    job_description: str


class RoadmapResult(BaseModel):
    roadmap: List[SkillRoadmap]
```

---

### download_models.py

```python
from pydantic import BaseModel


class DownloadFile(BaseModel):
    download_url: str       # S3 presigned URL
    expires_in_seconds: int = 3600


class DownloadRequest(BaseModel):
    session_id: str
    reformed_sections: dict
    parsed_resume: dict


class DownloadResult(BaseModel):
    pdf: DownloadFile
    docx: DownloadFile
```

---

## 3. Knowledge Base Chunk Schema

Each KB chunk stored in pgvector has the following structure:

```python
class KBChunk(BaseModel):
    id: int
    source_file: str        # e.g. 'ats_formatting_rules.md'
    chunk_index: int
    content: str            # raw text of the chunk
    embedding: List[float]  # 768-dim vector from gemini-embedding-001
    metadata: dict          # e.g. {"category": "formatting", "tags": ["ats", "structure"]}
```

---

## 4. Data Flow Summary

```
POST /resume/upload
  → S3 (file_key stored)
  → sessions row created (status: uploaded)

POST /analysis/start
  → S3 (read file by file_key)
  → resume parsed → ParsedResume
  → RAG retrieval → KB chunks
  → ATSScores + ATSIssues + SkillGaps computed
  → sessions row updated (status: analyzed)
  → AnalysisResult returned (in-memory, not persisted)

POST /reform/start
  → ReformResult computed (in-memory)
  → sessions row updated (status: reformed)

POST /reform/rescore
  → RescoreResult computed (in-memory)

POST /roadmap/generate
  → RoadmapResult computed (in-memory)

POST /download/generate
  → PDF + DOCX generated
  → uploaded to S3
  → presigned URLs returned
  → sessions row updated (status: completed)
```
