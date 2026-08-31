# COMPONENTS

## SkillGap.ai — Frontend and Backend Component Breakdown

---

## 1. Frontend Components

The frontend is a single HTML page with a JavaScript-driven state machine that progresses through distinct views.

### Page State Machine

```
┌─────────┐    ┌────────┐    ┌──────────┐    ┌─────────┐
│  LOGIN  │───►│ UPLOAD │───►│ANALYZING │───►│ RESULTS │
└─────────┘    └────────┘    └──────────┘    └─────────┘
                                                   │
                                                   ▼
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ DOWNLOAD │◄───│ ROADMAP  │◄───│COMPARISON│◄───│REFORMING │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
```

State transitions are managed in `js/app.js`. Only one view is visible at a time. All others are hidden with CSS (`display: none`).

---

### View: LOGIN

**File:** `index.html` — `#view-login`

**Elements:**
- SkillGap.ai logo and tagline
- "Sign in with Google" button
- Email + password form
- "Sign up" toggle
- Error message display

**JS:** `js/auth.js`

```
auth.js responsibilities:
  - initGoogleOAuth()         — load Google Identity Services script
  - handleGoogleSignIn(token) — POST /api/auth/google → store JWT
  - handleEmailLogin(form)    — POST /api/auth/email/login → store JWT
  - handleEmailSignup(form)   — POST /api/auth/email/signup → confirm message
  - handleLogout()            — POST /api/auth/logout → clear JWT → show login
  - getStoredToken()          — retrieve JWT from localStorage
  - isAuthenticated()         — check token validity
```

---

### View: UPLOAD

**File:** `index.html` — `#view-upload`

**Elements:**
- Drag-and-drop file zone
- "Browse file" button
- File type indicator (PDF / DOCX)
- File size display
- Job description textarea (min 100 characters)
- Character count indicator
- "Analyze Resume" submit button
- Loading state with progress indicator

**JS:** `js/upload.js`

```
upload.js responsibilities:
  - initDragDrop()             — attach drag/drop events to drop zone
  - handleFileSelect(file)     — validate file type and size
  - uploadFile(file)           — POST /api/resume/upload (multipart)
  - submitAnalysis(fileKey, jd) — POST /api/analysis/start
  - showUploadProgress()       — visual feedback during upload
  - validateJobDescription(jd) — check minimum length
```

---

### View: ANALYZING

**File:** `index.html` — `#view-analyzing`

**Elements:**
- Animated progress steps:
  1. Parsing resume...
  2. Analyzing ATS compatibility...
  3. Detecting skill gaps...
- Step completion indicators

This view is shown while `POST /api/analysis/start` is in flight (can take 5-15 seconds due to LLM calls).

---

### View: RESULTS

**File:** `index.html` — `#view-results`

**Elements:**

**ATS Score Panel**
- Overall score as a large circular progress ring
- Sub-score bars for: Keywords, Skills, Formatting, Experience, Projects
- Each bar shows score percentage + colored severity indicator

**Issues Panel**
- List of identified ATS issues
- Grouped by severity (High / Medium / Low)
- Each issue shows: category badge, description, affected section

**Skill Gap Panel**
- Skills listed in three groups:
  - Present (green checkmark)
  - Weakly Evidenced (yellow warning)
  - Not Found (red X)
- Tooltip on hover explaining each status

**Action Button**
- "Improve My Resume" → triggers reform flow

**JS:** `js/analysis.js`

```
analysis.js responsibilities:
  - renderATSScores(scores)      — draw score rings and bars
  - renderATSIssues(issues)      — render issue cards by severity
  - renderSkillGaps(gaps)        — render three-group skill list
  - getScoreColor(score)         — red/yellow/green based on value
  - triggerReform()              — POST /api/reform/start
```

---

### View: REFORMING

**File:** `index.html` — `#view-reforming`

**Elements:**
- Animated steps:
  1. Retrieving best practices...
  2. Rewriting experience sections...
  3. Rewriting project descriptions...
  4. Validating changes...
- Validation result summary

---

### View: COMPARISON

**File:** `index.html` — `#view-comparison`

**Elements:**

**Before vs After Score Panel**
- Side-by-side or animated score comparison
- Each sub-score shown with delta arrow and percentage change
- Overall improvement displayed prominently (e.g. "+23%")

**Resume Diff Panel**
- Side-by-side original vs reformed content
- Changed sections highlighted
- Validation badge ("Verified — based on your original content")

**Action Buttons**
- "View Learning Roadmap"
- "Download Resume"

**JS:** `js/reform.js`

```
reform.js responsibilities:
  - renderReformedSections(sections)   — show original vs reformed
  - renderScoreComparison(before, after, delta) — before/after panel
  - highlightChanges(original, reformed)       — diff highlighting
  - renderValidationBadge(summary)             — show validation status
```

---

### View: ROADMAP

**File:** `index.html` — `#view-roadmap`

**Elements:**
- One card per skill gap (only not_found and weakly_evidenced)
- Each card contains:
  - Skill name + priority badge (High / Medium / Low)
  - Estimated weeks
  - Stage accordion (expand/collapse per stage)
  - Stage contents: level badge, topic list, practice task

**JS:** `js/roadmap.js`

```
roadmap.js responsibilities:
  - renderRoadmap(roadmap)         — build roadmap cards
  - renderStageAccordion(stages)   — expandable stage sections
  - getPriorityColor(priority)     — color coding
  - getLevelBadge(level)           — beginner/intermediate/advanced labels
```

---

### View: DOWNLOAD

**File:** `index.html` — `#view-download`

**Elements:**
- "Your improved resume is ready" message
- Download PDF button (triggers presigned S3 URL)
- Download DOCX button (triggers presigned S3 URL)
- "Start Over" button → returns to UPLOAD view, clears state
- Optional: share/print buttons

**JS:** `js/download.js`

```
download.js responsibilities:
  - generateDownloads(sessionId, reformedSections, parsedResume)
      → POST /api/download/generate
  - triggerDownload(url, filename)   — initiate browser file download
  - handleStartOver()                — clear all state, return to upload
```

---

### Shared JS: api.js

Centralises all HTTP calls. Every other JS module calls functions from `api.js` rather than calling `fetch()` directly.

```javascript
// api.js structure
const API_BASE = "https://<app-runner-url>/api";

async function request(method, path, body, isMultipart = false) {
    // attach JWT, handle errors, return parsed JSON
}

export const api = {
    auth: {
        googleLogin: (idToken) => request("POST", "/auth/google", { id_token: idToken }),
        emailLogin: (email, password) => request("POST", "/auth/email/login", { email, password }),
        emailSignup: (email, password) => request("POST", "/auth/email/signup", { email, password }),
        logout: () => request("POST", "/auth/logout"),
    },
    resume: {
        upload: (formData) => request("POST", "/resume/upload", formData, true),
    },
    analysis: {
        start: (fileKey, jd) => request("POST", "/analysis/start", { file_key: fileKey, job_description: jd }),
    },
    reform: {
        start: (payload) => request("POST", "/reform/start", payload),
        rescore: (payload) => request("POST", "/reform/rescore", payload),
    },
    roadmap: {
        generate: (payload) => request("POST", "/roadmap/generate", payload),
    },
    download: {
        generate: (payload) => request("POST", "/download/generate", payload),
    },
};
```

---

## 2. Backend Service Components

### ResumeParser — `services/resume_parser.py`

**Responsibility:** Extract raw text and structured sections from uploaded PDF/DOCX files.

**Inputs:** S3 file key
**Outputs:** `ParsedResume` Pydantic model

```
PDF  → pdfplumber  → raw text
DOCX → python-docx → raw text
          │
          ▼
     spaCy pipeline
          │
     Section detection (regex + NLP)
          │
     Structured extraction:
     name, email, phone, linkedin, github,
     summary, education, skills, experience,
     projects, certifications, achievements
```

---

### ATSAnalyzer — `services/ats_analyzer.py`

**Responsibility:** Calculate ATS sub-scores using RAG-retrieved rules + Gemini Pro.

**Inputs:** `ParsedResume`, job description text, retrieved KB chunks
**Outputs:** `ATSScores`, `List[ATSIssue]`

```
Retrieved KB chunks (scoring rubrics)
          +
ParsedResume + JD
          │
          ▼
Gemini Pro prompt:
  [RULES] (from KB)
  [RESUME] (parsed sections)
  [JD] (job description)
  → Score each category 0-100
  → List specific issues with severity
          │
          ▼
ATSScores + List[ATSIssue]
```

---

### SkillGapPredictor — `services/skill_gap_predictor.py`

**Responsibility:** Extract required skills from JD, compare against resume, classify each.

**Inputs:** `ParsedResume`, job description text
**Outputs:** `List[SkillGap]`

```
Job Description
    │
    ▼ Gemini Pro: extract required skills list
Required Skills
    │
    ▼ Compare against ParsedResume.skills + experience + projects
For each skill:
  - Search in skills list           → present
  - Search in experience/projects   → present or weakly_evidenced
  - Not found anywhere              → not_found
```

---

### ResumeReformer — `services/resume_reformer.py`

**Responsibility:** Rewrite weak resume sections using RAG-grounded Gemini Pro.

**Inputs:** `ParsedResume`, `List[ATSIssue]`, `List[SkillGap]`, JD, KB chunks
**Outputs:** `ReformedSections`

```
Retrieved KB chunks (best practices, examples)
          +
Original resume content + issues + JD
          │
          ▼
Gemini Pro prompt:
  [RULES] (best practices from KB)
  [ORIGINAL] (section content)
  [ISSUES] (what needs improvement)
  [JD CONTEXT] (target role)
  → Generate reformed version
  → Must not introduce unsupported claims
```

---

### EvidenceValidator — `services/evidence_validator.py`

**Responsibility:** Validate that each reformed statement is grounded in the original resume.

**Inputs:** original resume text, list of generated statements
**Outputs:** approved statements, rejected statements with reasons

```
For each reformed statement:
          │
          ▼
Gemini Pro:
  [ORIGINAL RESUME TEXT]
  [GENERATED STATEMENT]
  → Is this statement fully supported by the original? (yes/no + reason)
          │
    ┌─────┴──────┐
  yes             no
    │               │
  Approved        Rejected
  (kept)          (removed, reason logged)
```

---

### RoadmapGenerator — `services/roadmap_generator.py`

**Responsibility:** Generate personalized learning roadmap for skill gaps.

**Inputs:** `List[SkillGap]` (not_found + weakly_evidenced only), JD
**Outputs:** `RoadmapResult`

```
Skill gaps + JD context
          │
          ▼
Gemini Pro:
  For each missing/weak skill:
  → Determine priority based on JD emphasis
  → Generate learning stages (beginner → advanced)
  → Suggest practice tasks per stage
  → Estimate weeks to competency
```

---

### RAGService — `services/rag_service.py`

**Responsibility:** Manage KB embedding at startup and semantic retrieval at query time.

```
STARTUP:
  check if knowledge_base_chunks table is populated
  if empty:
    read all .md files from /knowledge_base/
    chunk each file (512 tokens, 50 token overlap)
    embed each chunk → EmbeddingService
    store in Supabase pgvector

QUERY:
  receive query_text
  embed query → EmbeddingService
  pgvector cosine similarity search → top 5 chunks
  return List[KBChunk]
```

---

### EmbeddingService — `services/embedding_service.py`

**Responsibility:** Thin wrapper around Google `gemini-embedding-001` API.

```python
async def embed_text(text: str) -> List[float]:
    # call Google AI API
    # return 768-dimensional vector

async def embed_batch(texts: List[str]) -> List[List[float]]:
    # batch embed for KB seeding
```

---

### DocumentGenerator — `services/document_generator.py`

**Responsibility:** Compose final resume from reformed sections and generate PDF + DOCX.

```
ReformedSections + ParsedResume
          │
          ▼
Compose full resume content
          │
     ┌────┴────┐
     │         │
   PDF        DOCX
     │         │
WeasyPrint  python-docx
     │         │
   bytes     bytes
     │         │
     └────┬────┘
          │
     Upload to S3
     (outputs/session_id/resume.pdf)
     (outputs/session_id/resume.docx)
          │
     Generate presigned URLs (1hr expiry)
```

---

### StorageService — `services/storage_service.py`

**Responsibility:** All AWS S3 operations.

```python
async def upload_file(file_bytes, key, content_type) -> str
async def download_file(key) -> bytes
async def generate_presigned_url(key, expiry_seconds=3600) -> str
async def delete_file(key) -> None
```
