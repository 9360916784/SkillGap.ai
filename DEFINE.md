# DEFINE PHASE

## Project Name

**SkillGap.ai — AI Resume ATS Analyzer + Intelligent Resume Reformer + Skill Gap Predictor**

---

## 1. Business Problem

Finding a job is not only about having the required skills. Candidates must also present their skills, projects, experience, and achievements effectively in their resumes.

Many companies use **Applicant Tracking Systems (ATS)** to automatically parse and filter resumes before they are reviewed by recruiters. A resume that is not ATS-friendly gets rejected before a human ever sees it.

Candidates commonly face the following problems:

- Poor ATS-friendly formatting
- Missing important keywords from the target job description
- Weak, vague project and experience descriptions
- Generic statements with no measurable achievements
- Skills present in their knowledge but not evidenced clearly in the resume
- Difficulty understanding why their resume receives a low ATS score
- No clear understanding of which skills are required for a target role
- No guidance on how to develop missing skills

### Core Business Problem

> Candidates need an intelligent system that can understand their resume, identify ATS and content weaknesses, identify skills that are missing or insufficiently evidenced for a target role, improve the resume without fabricating information, verify the improvements, and provide a learning roadmap for identified skill gaps.

The project focuses on **resume analysis and improvement**.

It does **not** focus on job matching or job recommendations.

---

## 2. Proposed Solution

The proposed system is an **AI-powered Resume Intelligence Platform** that helps candidates analyze and improve their resumes for a specific target job description.

The user provides:

1. **Resume** — PDF or DOCX
2. **Target Job Description** — pasted text

The system performs the following pipeline:

```
Resume
   +
Target Job Description
        │
        ▼
   Resume Parsing
        │
        ▼
   Resume Analysis
        │
        ├── ATS Analysis        ← grounded via RAG knowledge base
        ├── Skill Analysis
        └── Content Analysis
                │
                ▼
        Skill Gap Detection
                │
                ▼
        AI Resume Reformer      ← grounded via RAG knowledge base
                │
                ▼
        Evidence Validation     ← anti-fabrication check
                │
                ▼
        Improved Resume
                │
                ▼
        Before vs After
                │
                ▼
       Learning Roadmap
```

The goal is to transform the system from a simple resume checker into a complete **resume improvement and skill development assistant**.

---

## 3. Project Objectives

### Objective 1 — Resume Understanding

Extract important information from the candidate's resume:

- Name and contact information
- Education
- Skills and technologies
- Projects
- Experience and internships
- Certifications
- Achievements
- Professional summary

### Objective 2 — ATS Analysis

Evaluate the resume based on:

- Keyword match with the target job description
- Skills coverage
- Formatting and ATS readability
- Section structure
- Content quality
- Strength of project and experience descriptions

Scores are calculated against a structured ATS rules knowledge base using RAG, not generated arbitrarily by the LLM.

### Objective 3 — Skill Gap Detection

Analyze the target job description and determine:

- Required skills for the role
- Skills clearly present and evidenced in the resume
- Skills mentioned but weakly evidenced
- Skills not found in the resume at all

### Objective 4 — Intelligent Resume Reformation

Improve the wording, structure, and impact of the resume while preserving the candidate's actual experience. The reformer references a knowledge base of best practices before generating any content.

### Objective 5 — Evidence Validation (Anti-Fabrication)

Prevent the AI from generating:

- Fake experience
- Fake projects
- Fake skills or tools
- Fake certifications
- Fake achievements

Every AI-generated statement must be supported by information provided by the candidate in their original resume.

### Objective 6 — Before vs After Measurement

Re-analyze the resume after reformation and show a measurable, explainable improvement in ATS score across all sub-categories.

### Objective 7 — Learning Roadmap

Generate a personalized learning roadmap based on skills that are missing or insufficiently evidenced, with priority levels, learning sequence, and beginner-to-advanced progression.

---

## 4. Stakeholders

### 4.1 Primary Stakeholder — Candidate / Job Seeker

The main user of the system. They upload their resume and target job description to receive analysis, an improved resume, and a learning roadmap.

**Target Users:**

- College students preparing for placements
- Fresh graduates entering the job market
- Entry-level and mid-level job seekers
- Career switchers targeting a new domain
- Internship applicants

### 4.2 Secondary Stakeholders

**Educational Institutions**
Colleges can use the system to help students prepare resumes for placement activities.

**Placement and Career Development Teams**
Career development teams can use the platform to provide structured, consistent resume feedback at scale.

**Training Institutes**
Training organizations can use the skill gap information to identify where learners need additional training and align their course offerings accordingly.

---

## 5. User Journey

### Step 1 — Upload Resume

The user uploads their resume in PDF or DOCX format.

### Step 2 — Provide Target Job Description

The user pastes the job description for their target role.

Example:

```
DevOps Engineer

Requirements:
AWS, Linux, Git, Docker, Kubernetes, Terraform, CI/CD
```

### Step 3 — Resume Analysis

The system parses the resume, extracts structured information, and performs ATS, skill, and content analysis — all grounded in the RAG knowledge base.

### Step 4 — View ATS Score

```
Overall ATS Score: 78%

Keywords       82%
Skills         76%
Formatting     91%
Projects       84%
Experience     68%
```

### Step 5 — Identify Skill Gaps

```
✓  AWS           — Present
✓  Linux         — Present
✓  Git           — Present
✓  Docker        — Present
✓  CI/CD         — Present
⚠  Kubernetes    — Weakly Evidenced
⚠  Terraform     — Not Found
```

### Step 6 — Reform Resume

The AI rewrites weak sections using best practices retrieved from the knowledge base, without fabricating any information.

### Step 7 — Validate Changes

The system checks that every AI-generated statement is supported by the original resume content. Unsupported statements are rejected.

### Step 8 — Compare Results

```
Before: 64%  ───────────────────►  After: 87%
                  +23% Improvement
```

### Step 9 — Download Improved Resume

The user downloads the improved resume as a clean document.

### Step 10 — View Learning Roadmap

```
Kubernetes
     ↓
  Pods and Deployments
  Services and Networking
  Helm Charts

Terraform
     ↓
  HCL Syntax
  Providers and Resources
  State Management
```

---

## 6. Core Components

### 6.1 Resume Parser

Extracts text and structured information from PDF and DOCX files.

Identifies:
- Personal information
- Education
- Skills and tools
- Projects
- Work experience and internships
- Certifications
- Achievements
- Professional summary

### 6.2 RAG Knowledge Base

A set of structured **markdown documents** covering:

- ATS formatting rules and standards
- Resume section best practices
- Scoring rubrics per category (keywords, skills, formatting, experience, projects)
- Strong vs weak language examples
- Industry-standard resume structure guidelines

These documents are chunked, embedded using Google `gemini-embedding-001`, and stored in Supabase pgvector. At query time, relevant chunks are retrieved semantically and injected into the LLM prompt as grounding context.

```
Markdown Knowledge Base (.md files)
            ↓
   Chunked + Embedded at startup
            ↓
   Stored in Supabase (pgvector)
            ↓
   Retrieved at query time (semantic search)
            ↓
   Injected into LLM prompt as context
            ↓
   Grounded, consistent, explainable output
```

### 6.3 Resume Analysis Engine

Analyzes the extracted resume content against the knowledge base and job description.

Identifies:
- Weak or vague descriptions
- Missing measurable achievements
- Repeated keywords
- Poor section structure
- Generic statements
- Formatting issues

### 6.4 ATS Analyzer

Calculates an ATS-oriented score with the following sub-categories:

| Category   | Description                                      | Weight |
| :--------- | :----------------------------------------------- | :----- |
| Keywords   | Match between resume and job description         | 25%    |
| Skills     | Technical and domain skills coverage             | 25%    |
| Formatting | ATS-readable structure and layout                | 20%    |
| Experience | Strength and specificity of experience sections  | 15%    |
| Projects   | Quality and technical clarity of project entries | 15%    |
| **Overall**| Weighted composite                               | 100%   |

Scoring logic is based on identifiable rules retrieved from the knowledge base, not on LLM-generated numbers alone.

### 6.5 Skill Gap Predictor

Extracts required skills from the target job description and compares them against skills evidenced in the resume.

```
Target Job Description
        │
        ▼
  Skill Extraction (spaCy + LLM)
        │
        ▼
  Compare with Resume Evidence
        │
        ▼
┌──────────────────────────────┐
│ Present                      │
│ Weakly Evidenced             │
│ Not Found                    │
└──────────────────────────────┘
```

> "Not Found" means the resume does not sufficiently evidence the skill — not that the candidate lacks it.

### 6.6 AI Resume Reformer

Rewrites weak resume content to be clearer, more impactful, and ATS-optimized.

Before generating any content, the reformer retrieves relevant best practices from the knowledge base via RAG.

Improvements applied to:
- Action verbs
- Sentence structure and conciseness
- Technical clarity and specificity
- Project descriptions
- Experience descriptions
- Achievement statements

Example:

| | |
|---|---|
| **Before** | Worked on AWS project. |
| **After** | Developed and deployed a cloud-based application using AWS services including EC2 and S3. |

### 6.7 Evidence Validation

Validates every AI-generated statement against the original resume content before accepting it.

```
Original Resume Evidence
         ↓
   Retrieved Best Practices (RAG)
         ↓
      AI Reformation
         ↓
   Evidence Validation Check
         ↓
   Approved Content Only
```

Statements that introduce information not present in the original resume are rejected.

### 6.8 Before vs After Analyzer

Re-runs the full ATS analysis on the improved resume and produces a comparison report:

```
Category     Before    After    Change
─────────────────────────────────────
Keywords       58%      84%     +26%
Skills         62%      86%     +24%
Formatting     78%      92%     +14%
Experience     55%      83%     +28%
Projects       59%      86%     +27%
─────────────────────────────────────
Overall        64%      87%     +23%
```

### 6.9 Learning Roadmap Generator

Generates a personalized, structured learning path for each skill that is missing or weakly evidenced.

Each roadmap entry includes:
- Skill name and current status
- Priority level (High / Medium / Low)
- Learning sequence (topic order)
- Subtopics per skill
- Beginner to advanced progression

---

## 7. Anti-Fabrication Requirement

This is a critical and non-negotiable business requirement.

The AI must **never fabricate candidate information**.

### Prohibited Example

Candidate's original resume says:
> Learned Docker.

The AI must **not** generate:
> Implemented Docker-based containerization in a production microservices environment.

There is no evidence that the candidate did this.

### Acceptable Example

The AI can generate:
> Developed foundational knowledge of Docker containerization concepts.

This stays within the bounds of what the candidate stated.

### Validation Principle

```
Every generated statement
        ↓
Must map to evidence
in the original resume
        ↓
Unsupported statements
are rejected before output
```

---

## 8. Existing Solutions and Identified Gap

### Existing Solutions

| Platform       | Relevant Features                                                                 |
| :------------- | :-------------------------------------------------------------------------------- |
| Jobscan        | ATS analysis, keyword matching, missing skills, formatting analysis               |
| Rezi           | ATS scoring, resume checking, AI-assisted resume improvement                      |
| Resume Worded  | Resume feedback, bullet-point analysis, content improvement, resume scoring       |

### Gap in Existing Solutions

Existing platforms identify problems. SkillGap.ai completes the full improvement cycle:

```
UNDERSTAND
     ↓
ANALYZE
     ↓
IDENTIFY GAPS
     ↓
REFORM         ← grounded in knowledge base, not arbitrary LLM output
     ↓
VALIDATE       ← anti-fabrication check
     ↓
MEASURE        ← before vs after comparison
     ↓
LEARN          ← personalized roadmap
```

### Key Differentiating Features

1. RAG-powered analysis grounded in a structured ATS knowledge base
2. Evidence-grounded AI resume reforming
3. Anti-fabrication validation at every step
4. Measurable before vs after ATS improvement
5. Personalized learning roadmap for skill gaps
6. Fully cloud-deployed — no local dependencies

---

## 9. Technology Stack

| Domain               | Technology                        | Reason                                                                |
| :------------------- | :-------------------------------- | :-------------------------------------------------------------------- |
| **Frontend**         | HTML + CSS + JavaScript           | No build step, sufficient for the UI, easy to host statically         |
| **Backend**          | Python + FastAPI                  | Async support for LLM calls, Pydantic for structured data validation  |
| **Document Parsing** | pdfplumber + python-docx          | Best-in-class extraction for PDF and DOCX                             |
| **NLP**              | spaCy                             | Section detection, entity recognition, keyword extraction             |
| **LLM**              | Google Gemini Pro (Gemini API)    | Resume reformation, skill extraction, roadmap generation              |
| **Embeddings**       | Google gemini-embedding-001       | Knowledge base embedding — same API key, no separate provider         |
| **Vector Store + DB**| Supabase (PostgreSQL + pgvector)  | Stores resume data, analysis results, and knowledge base vectors      |
| **Knowledge Base**   | Markdown files + pgvector         | ATS rules, best practices, scoring rubrics — embedded at startup      |
| **File Storage**     | AWS S3                            | Resume file uploads (PDF and DOCX)                                    |
| **Frontend Hosting** | AWS S3 Static Website             | Hosts the HTML/CSS/JS frontend                                        |
| **Backend Hosting**  | AWS EC2 (t2.micro) + Docker       | Runs the containerized FastAPI application                            |
| **CI/CD**            | GitHub Actions                    | Automated build and deployment pipeline                               |

---

## 10. Cloud Architecture

Everything is cloud-hosted. There are no local-only components. The system is fully functional after deployment.

```
User Browser
     │
     ▼
AWS S3 — Static Frontend (HTML + CSS + JS)
     │
     ▼ HTTPS API calls
AWS EC2 t2.micro — Docker container (FastAPI)
     │
     ├──► AWS S3                    — Resume file storage (PDF / DOCX)
     │
     ├──► Supabase (PostgreSQL + pgvector)
     │         ├── Resume data and analysis results
     │         └── Knowledge base vectors (ATS rules, best practices)
     │
     ├──► Google Gemini API (LLM — Gemini Pro + Embeddings — gemini-embedding-001)
```

### Free Tier Summary

| Service        | Free Tier Allowance                           |
| :------------- | :-------------------------------------------- |
| AWS EC2        | t2.micro, 750 hours/month (first 12 months)   |
| AWS S3         | 5 GB storage, 20K GET / 2K PUT requests/month |
| Supabase       | 500 MB database, pgvector included            |
| Google AI API  | Free tier — Gemini Pro + gemini-embedding-001 |
| GitHub Actions | 2,000 minutes/month free                      |

---

## 11. Functional Requirements

The system must allow users to:

- Upload a resume in PDF or DOCX format
- Paste a target job description
- Parse and extract structured information from the resume
- Analyze the resume for ATS compatibility using a knowledge base
- View a detailed ATS score with sub-category breakdown
- View identified resume weaknesses with explanations
- View a skill gap analysis against the target job description
- Reform resume content using AI (grounded in best practices)
- View validation of every AI-generated change
- Download the improved resume
- View a before vs after ATS score comparison
- View a personalized learning roadmap for missing or weak skills

---

## 12. Non-Functional Requirements

| Requirement     | Description                                                                           |
| :-------------- | :------------------------------------------------------------------------------------ |
| **Accuracy**    | Resume extraction and analysis must be accurate and consistent                        |
| **Reliability** | AI must not introduce unsupported information into the resume                         |
| **Explainability** | Every score and recommendation must include a reason                               |
| **Performance** | Resume analysis and reformation should complete within a reasonable response time     |
| **Security**    | Resume data must be securely handled and not exposed to other users                   |
| **Privacy**     | User resume content must remain private                                               |
| **Scalability** | The system should support multiple concurrent users                                   |
| **Deployability** | All components must run on cloud infrastructure — no local dependencies             |
| **Usability**   | The interface must be simple enough for students and non-technical job seekers        |

---

## 13. Project Scope

### In Scope

- Resume parsing from PDF and DOCX
- RAG-powered ATS scoring with sub-score breakdown
- Keyword and skills analysis
- Skill gap detection against a target job description
- AI resume content reformation grounded in a knowledge base
- Anti-fabrication evidence validation
- Before and after ATS score comparison
- Improved resume generation and download
- Personalized learning roadmap for skill gaps
- Full cloud deployment (AWS + Supabase)

### Out of Scope

- Job searching or job matching
- Job recommendations
- Automatic job applications
- Recruiter-side candidate ranking
- Fabrication of experience, projects, skills, certifications, or achievements
- Local-only deployments

---

## 14. Success Criteria

The project is considered successful when it can:

1. Parse resumes accurately from PDF and DOCX
2. Identify and extract all standard resume sections correctly
3. Calculate meaningful, explainable ATS sub-scores grounded in a knowledge base
4. Extract required skills from any target job description
5. Correctly classify skills as present, weakly evidenced, or not found
6. Reform resume content with measurable quality improvement
7. Validate every AI-generated statement against original resume content
8. Demonstrate a measurable before vs after ATS score improvement
9. Generate a structured, actionable, and personalized learning roadmap
10. Explain every recommendation and score clearly to the user
11. Run fully on cloud infrastructure with no local dependencies

---

## 15. One-Line Definition

> **An AI-powered resume intelligence platform that analyzes a candidate's resume against a target job description, identifies ATS and skill gaps using a RAG-grounded knowledge base, intelligently reforms the resume without fabricating information, validates every improvement, measures the before vs after impact, and generates a personalized learning roadmap.**

---

## 16. Development Lifecycle

```
DEFINE      — Problem definition, objectives, scope, stack      ✅ Complete
DESIGN      — Architecture, data models, API contracts          ✅ Complete
DEVELOPMENT — Implementation by component                       ⬜ Pending
DEPLOYMENT  — Docker, AWS App Runner, S3, CI/CD pipeline        ⬜ Pending
```
