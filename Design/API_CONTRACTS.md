# API CONTRACTS

## SkillGap.ai — FastAPI Endpoint Specifications

---

## Base URL

```
Production:  https://<app-runner-url>/api
Development: http://localhost:8000/api
```

All endpoints require `Authorization: Bearer <jwt>` header except auth endpoints.

All responses follow the envelope pattern:
```json
{
  "success": true,
  "data": { ... },
  "error": null
}
```

---

## 1. Auth Endpoints

### POST /api/auth/google

Initiate Google OAuth flow via Supabase Auth.

**Request**
```json
{
  "id_token": "google_oauth_id_token_string"
}
```

**Response 200**
```json
{
  "success": true,
  "data": {
    "access_token": "supabase_jwt_token",
    "token_type": "bearer",
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "name": "John Doe",
      "avatar_url": "https://..."
    }
  }
}
```

---

### POST /api/auth/email/signup

Register with email and password.

**Request**
```json
{
  "email": "user@example.com",
  "password": "securepassword123"
}
```

**Response 201**
```json
{
  "success": true,
  "data": {
    "message": "Account created. Please verify your email.",
    "user_id": "uuid"
  }
}
```

---

### POST /api/auth/email/login

Login with email and password.

**Request**
```json
{
  "email": "user@example.com",
  "password": "securepassword123"
}
```

**Response 200**
```json
{
  "success": true,
  "data": {
    "access_token": "supabase_jwt_token",
    "token_type": "bearer",
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "name": "John Doe"
    }
  }
}
```

---

### POST /api/auth/logout

Invalidate session.

**Request** — No body required.

**Response 200**
```json
{
  "success": true,
  "data": {
    "message": "Logged out successfully."
  }
}
```

---

## 2. Resume Endpoints

### POST /api/resume/upload

Upload a resume file (PDF or DOCX). Returns a file key used in subsequent calls.

**Request** — `multipart/form-data`

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `file` | File | Yes | Resume PDF or DOCX |

**Constraints**
- Max file size: 5MB
- Accepted types: `application/pdf`, `application/vnd.openxmlformats-officedocument.wordprocessingml.document`

**Response 200**
```json
{
  "success": true,
  "data": {
    "file_key": "uploads/uuid/resume.pdf",
    "file_name": "resume.pdf",
    "file_type": "pdf",
    "file_size_kb": 142
  }
}
```

**Error 400**
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "INVALID_FILE_TYPE",
    "message": "Only PDF and DOCX files are accepted."
  }
}
```

---

## 3. Analysis Endpoints

### POST /api/analysis/start

Parse the uploaded resume and run full ATS + skill gap analysis against the job description.

**Request**
```json
{
  "file_key": "uploads/uuid/resume.pdf",
  "job_description": "We are looking for a DevOps Engineer with experience in AWS, Docker, Kubernetes, Terraform, and CI/CD pipelines..."
}
```

**Response 200**
```json
{
  "success": true,
  "data": {
    "session_id": "uuid",
    "parsed_resume": {
      "name": "John Doe",
      "email": "john@example.com",
      "phone": "+1234567890",
      "linkedin": "linkedin.com/in/johndoe",
      "summary": "Software engineer with 3 years experience...",
      "education": [
        {
          "degree": "B.Tech Computer Science",
          "institution": "XYZ University",
          "year": "2022"
        }
      ],
      "skills": ["Python", "Docker", "AWS", "Git"],
      "experience": [
        {
          "title": "Software Engineer",
          "company": "ABC Corp",
          "duration": "Jan 2022 - Present",
          "bullets": [
            "Worked on AWS project.",
            "Maintained Docker containers."
          ]
        }
      ],
      "projects": [
        {
          "name": "Resume Parser",
          "description": "Built a Python tool to parse resumes.",
          "tech_stack": ["Python", "spaCy"]
        }
      ],
      "certifications": ["AWS Cloud Practitioner"],
      "achievements": ["Winner, Hackathon 2023"]
    },
    "ats_scores": {
      "overall": 64,
      "keywords": 58,
      "skills": 62,
      "formatting": 78,
      "experience": 55,
      "projects": 59
    },
    "ats_issues": [
      {
        "category": "experience",
        "severity": "high",
        "issue": "Experience bullets lack measurable achievements and specific impact.",
        "section": "Software Engineer at ABC Corp"
      },
      {
        "category": "keywords",
        "severity": "medium",
        "issue": "Keywords 'Kubernetes' and 'Terraform' from the job description are not present.",
        "section": "skills"
      }
    ],
    "skill_gaps": [
      {
        "skill": "AWS",
        "status": "present",
        "evidence": "Mentioned in skills and experience sections."
      },
      {
        "skill": "Docker",
        "status": "present",
        "evidence": "Mentioned in skills section."
      },
      {
        "skill": "Kubernetes",
        "status": "not_found",
        "evidence": null
      },
      {
        "skill": "Terraform",
        "status": "not_found",
        "evidence": null
      },
      {
        "skill": "CI/CD",
        "status": "weakly_evidenced",
        "evidence": "Mentioned once but not demonstrated in any project or role."
      }
    ],
    "raw_resume_text": "John Doe\njohn@example.com\n..."
  }
}
```

---

## 4. Reform Endpoints

### POST /api/reform/start

Rewrite weak resume sections using RAG-grounded Gemini Pro. Validates all output.

**Request**
```json
{
  "session_id": "uuid",
  "raw_resume_text": "John Doe\njohn@example.com\n...",
  "ats_issues": [ ... ],
  "skill_gaps": [ ... ],
  "job_description": "We are looking for a DevOps Engineer..."
}
```

**Response 200**
```json
{
  "success": true,
  "data": {
    "reformed_sections": {
      "summary": {
        "original": "Software engineer with 3 years experience.",
        "reformed": "Results-driven software engineer with 3 years of experience building and deploying cloud-native applications using AWS and Docker.",
        "validated": true,
        "changes": ["Strengthened action language", "Added specific technologies mentioned in original resume"]
      },
      "experience": [
        {
          "title": "Software Engineer",
          "company": "ABC Corp",
          "duration": "Jan 2022 - Present",
          "original_bullets": [
            "Worked on AWS project.",
            "Maintained Docker containers."
          ],
          "reformed_bullets": [
            "Designed and deployed cloud infrastructure using AWS EC2 and S3, reducing deployment time by streamlining the release process.",
            "Managed and maintained Docker containerized environments, ensuring consistent application performance across development and staging."
          ],
          "validated": true
        }
      ],
      "projects": [
        {
          "name": "Resume Parser",
          "original": "Built a Python tool to parse resumes.",
          "reformed": "Developed a Python-based resume parsing tool using spaCy NLP, enabling automated extraction of candidate information from unstructured text documents.",
          "validated": true
        }
      ]
    },
    "validation_summary": {
      "total_statements": 8,
      "approved": 8,
      "rejected": 0,
      "rejected_reasons": []
    }
  }
}
```

---

### POST /api/reform/rescore

Re-run ATS analysis on the reformed resume to produce the before vs after comparison.

**Request**
```json
{
  "session_id": "uuid",
  "reformed_resume_text": "John Doe\njohn@example.com\n...(reformed content)...",
  "job_description": "We are looking for a DevOps Engineer..."
}
```

**Response 200**
```json
{
  "success": true,
  "data": {
    "before": {
      "overall": 64,
      "keywords": 58,
      "skills": 62,
      "formatting": 78,
      "experience": 55,
      "projects": 59
    },
    "after": {
      "overall": 87,
      "keywords": 84,
      "skills": 86,
      "formatting": 92,
      "experience": 83,
      "projects": 86
    },
    "delta": {
      "overall": 23,
      "keywords": 26,
      "skills": 24,
      "formatting": 14,
      "experience": 28,
      "projects": 27
    }
  }
}
```

---

## 5. Roadmap Endpoints

### POST /api/roadmap/generate

Generate a personalized learning roadmap for skill gaps.

**Request**
```json
{
  "session_id": "uuid",
  "skill_gaps": [
    {
      "skill": "Kubernetes",
      "status": "not_found"
    },
    {
      "skill": "Terraform",
      "status": "not_found"
    },
    {
      "skill": "CI/CD",
      "status": "weakly_evidenced"
    }
  ],
  "job_description": "We are looking for a DevOps Engineer..."
}
```

**Response 200**
```json
{
  "success": true,
  "data": {
    "roadmap": [
      {
        "skill": "Kubernetes",
        "priority": "high",
        "status": "not_found",
        "estimated_weeks": 4,
        "stages": [
          {
            "stage": 1,
            "title": "Foundations",
            "level": "beginner",
            "topics": [
              "Container orchestration concepts",
              "Kubernetes architecture overview",
              "Pods, Nodes, and Clusters"
            ],
            "practice": "Set up a local Kubernetes cluster using Minikube."
          },
          {
            "stage": 2,
            "title": "Core Resources",
            "level": "intermediate",
            "topics": [
              "Deployments and ReplicaSets",
              "Services and Networking",
              "ConfigMaps and Secrets"
            ],
            "practice": "Deploy a multi-container application to a local cluster."
          },
          {
            "stage": 3,
            "title": "Production Readiness",
            "level": "advanced",
            "topics": [
              "Helm Charts",
              "Persistent Volumes",
              "Horizontal Pod Autoscaling"
            ],
            "practice": "Deploy an application to a managed Kubernetes service (EKS or GKE free tier)."
          }
        ]
      },
      {
        "skill": "Terraform",
        "priority": "high",
        "status": "not_found",
        "estimated_weeks": 3,
        "stages": [
          {
            "stage": 1,
            "title": "Foundations",
            "level": "beginner",
            "topics": [
              "Infrastructure as Code concepts",
              "HCL syntax",
              "Providers and Resources"
            ],
            "practice": "Write a Terraform configuration to provision an S3 bucket."
          },
          {
            "stage": 2,
            "title": "State and Modules",
            "level": "intermediate",
            "topics": [
              "Terraform state management",
              "Variables and outputs",
              "Reusable modules"
            ],
            "practice": "Build a reusable VPC module and deploy it."
          }
        ]
      }
    ]
  }
}
```

---

## 6. Download Endpoints

### POST /api/download/generate

Compose the final reformed resume and generate PDF and DOCX files.

**Request**
```json
{
  "session_id": "uuid",
  "reformed_sections": { ... },
  "parsed_resume": { ... }
}
```

**Response 200**
```json
{
  "success": true,
  "data": {
    "pdf": {
      "download_url": "https://s3.amazonaws.com/skillgap-ai-uploads/outputs/uuid/resume.pdf?X-Amz-Signature=...",
      "expires_in_seconds": 3600
    },
    "docx": {
      "download_url": "https://s3.amazonaws.com/skillgap-ai-uploads/outputs/uuid/resume.docx?X-Amz-Signature=...",
      "expires_in_seconds": 3600
    }
  }
}
```

---

## 7. Health Check

### GET /api/health

Used by AWS App Runner for health checks.

**Response 200**
```json
{
  "status": "ok",
  "version": "1.0.0"
}
```

---

## Error Codes Reference

| Code | HTTP Status | Description |
| :--- | :--- | :--- |
| `INVALID_FILE_TYPE` | 400 | File is not PDF or DOCX |
| `FILE_TOO_LARGE` | 400 | File exceeds 5MB limit |
| `PARSE_FAILED` | 422 | Resume could not be parsed |
| `JD_TOO_SHORT` | 400 | Job description is too short to extract skills |
| `VALIDATION_FAILED` | 422 | Evidence validation rejected all reformed content |
| `LLM_UNAVAILABLE` | 503 | Google AI API is unavailable |
| `UNAUTHORIZED` | 401 | Missing or invalid JWT token |
| `FORBIDDEN` | 403 | User does not have access to this resource |
| `NOT_FOUND` | 404 | Resource not found |
| `INTERNAL_ERROR` | 500 | Unexpected server error |

---

## Rate Limits

| Endpoint | Limit |
| :--- | :--- |
| `/api/resume/upload` | 10 requests / user / hour |
| `/api/analysis/start` | 10 requests / user / hour |
| `/api/reform/start` | 10 requests / user / hour |
| `/api/roadmap/generate` | 20 requests / user / hour |
| `/api/download/generate` | 20 requests / user / hour |

Limits are enforced to stay within Google AI API free tier rate limits.
