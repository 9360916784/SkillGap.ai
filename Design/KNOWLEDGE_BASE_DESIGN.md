# KNOWLEDGE BASE DESIGN

## SkillGap.ai — RAG Knowledge Base Structure

---

## 1. Purpose

The knowledge base is the foundation that prevents LLM hallucination and ensures consistent, explainable output across all analysis and reformation operations.

Instead of asking Gemini Pro to "know" what makes a good resume, the system retrieves relevant rules from this knowledge base and injects them into every prompt. This means:

- ATS scores are based on defined rubrics, not LLM guesses
- Resume improvements follow documented best practices
- Scoring is consistent across all users and runs
- Every recommendation can be traced back to a specific rule

---

## 2. Knowledge Base Files

Located at: `backend/knowledge_base/`

| File | Purpose | Used By |
| :--- | :--- | :--- |
| `ats_formatting_rules.md` | ATS formatting standards, what breaks ATS parsing | ATSAnalyzer |
| `resume_section_guide.md` | How each section should be written and structured | ATSAnalyzer, ResumeReformer |
| `scoring_rubrics.md` | Scoring criteria per sub-category (0-100 scale) | ATSAnalyzer |
| `strong_weak_examples.md` | Before/after language examples for common patterns | ResumeReformer |
| `action_verbs.md` | Strong action verbs organised by category | ResumeReformer |
| `skill_evidence_guide.md` | How skills should be evidenced in different sections | SkillGapPredictor, ATSAnalyzer |

---

## 3. File Contents Design

### ats_formatting_rules.md

Covers what ATS systems look for in formatting and what breaks them.

```markdown
# ATS Formatting Rules

## File Format
- PDF and DOCX are accepted by most ATS systems.
- Avoid scanned PDFs — they cannot be parsed.
- Avoid image-based resumes — text must be selectable.

## Layout and Structure
- Use a single-column layout for maximum ATS compatibility.
- Avoid tables, text boxes, headers/footers, and columns — ATS parsers often skip them.
- Use standard section headings: Education, Experience, Skills, Projects, Certifications.
- Avoid creative headings like "My Journey" or "What I've Built".

## Fonts and Styling
- Use standard fonts: Arial, Calibri, Times New Roman, Helvetica.
- Avoid decorative fonts — they may render incorrectly.
- Font size: 10-12pt for body, 14-16pt for name.

## Bullet Points
- Use standard bullet characters (•, -, *).
- Do not use custom symbols or icons as bullets.

## Contact Information
- Place contact info at the top of the document.
- Include: name, email, phone, LinkedIn URL.
- Avoid using images for contact info.

## File Naming
- Name the file clearly: FirstName-LastName-Resume.pdf
- Avoid special characters in file names.
```

---

### resume_section_guide.md

How each resume section should be written.

```markdown
# Resume Section Writing Guide

## Professional Summary
- 2-4 sentences maximum.
- Lead with your strongest value proposition.
- Mention years of experience and key domain.
- Include 2-3 core technical skills relevant to target role.
- Avoid: "I am a hardworking person who..."
- Prefer: "Software engineer with 3 years of experience building..."

## Experience Section
- Use reverse chronological order (most recent first).
- Each bullet must start with a strong past-tense action verb.
- Each bullet should follow: Action + Task + Result or Impact.
- Include numbers and metrics where possible.
- Weak: "Worked on backend systems."
- Strong: "Developed and optimised REST API endpoints using FastAPI, reducing average response time by 40%."
- 3-5 bullets per role is ideal.

## Projects Section
- List project name, one-line description, and tech stack used.
- Describe what problem it solved or what it demonstrated.
- Mention scale, users, or outcome if available.
- Weak: "Built a web app."
- Strong: "Developed a full-stack task management application using React and FastAPI, supporting user authentication and real-time updates."

## Skills Section
- Organise skills into categories: Languages, Frameworks, Tools, Cloud, Databases.
- List skills you can actually discuss in an interview.
- Do not list soft skills here (e.g. "teamwork", "communication").
- Only include skills evidenced elsewhere in the resume.

## Education Section
- List degree, institution, graduation year.
- Include relevant coursework if recently graduated.
- Include GPA only if above 3.5 / 8.0 CGPA.

## Certifications
- List: Certification name, issuing organisation, year.
- Only include current, unexpired certifications.
```

---

### scoring_rubrics.md

Defines exactly how each sub-score is calculated.

```markdown
# ATS Scoring Rubrics

## Keywords Score (0-100)
Measures how well resume keywords match the target job description.

| Score Range | Criteria |
|-------------|----------|
| 85-100      | All major JD keywords present, naturally integrated |
| 70-84       | Most JD keywords present, minor gaps |
| 55-69       | Several important keywords missing |
| 40-54       | Many keywords missing, poor JD alignment |
| 0-39        | Very few keywords from JD present |

Rules:
- Extract all technical keywords from JD.
- Check presence in: skills, experience bullets, project descriptions.
- Penalise keyword stuffing (same keyword repeated 5+ times).
- Synonyms count (e.g. "ML" and "Machine Learning" are equivalent).

## Skills Score (0-100)
Measures completeness and evidence quality of skills.

| Score Range | Criteria |
|-------------|----------|
| 85-100      | All required skills present and evidenced in context |
| 70-84       | Most skills present, some only listed without evidence |
| 55-69       | Core skills present but several required skills missing |
| 40-54       | Skills section incomplete, missing many required skills |
| 0-39        | Skills section absent or severely incomplete |

Rules:
- Skills listed in a skills section score lower than skills evidenced in experience.
- Skills mentioned in a project score higher than skills only in the skills list.
- Penalise if skills section only contains soft skills.

## Formatting Score (0-100)
Measures ATS readability and structural quality.

| Score Range | Criteria |
|-------------|----------|
| 85-100      | Clean structure, standard sections, no ATS-breaking elements |
| 70-84       | Minor formatting issues, sections present |
| 55-69       | Some non-standard elements detected |
| 40-54       | Multiple formatting problems |
| 0-39        | Severe formatting issues, likely to fail ATS parsing |

Rules:
- Penalise: missing standard sections, non-standard headings.
- Penalise: no contact information at top.
- Penalise: no clear section separators.

## Experience Score (0-100)
Measures the quality and specificity of experience descriptions.

| Score Range | Criteria |
|-------------|----------|
| 85-100      | Strong action verbs, specific tasks, measurable results |
| 70-84       | Good descriptions, some missing metrics |
| 55-69       | Vague descriptions, weak verbs |
| 40-54       | Generic statements, no specifics |
| 0-39        | Experience section missing or near-empty |

Rules:
- Each bullet must start with an action verb.
- Penalise: "Responsible for...", "Helped with...", "Worked on..."
- Reward: quantifiable results (%, time, scale, users).

## Projects Score (0-100)
Measures the quality of project descriptions.

| Score Range | Criteria |
|-------------|----------|
| 85-100      | Clear descriptions, tech stack, problem solved, outcome |
| 70-84       | Good descriptions, minor details missing |
| 55-69       | Vague descriptions, tech stack listed but no context |
| 40-54       | Projects listed but poorly described |
| 0-39        | No projects section or completely empty |

Rules:
- Each project must mention: what it does, technologies used.
- Reward: outcome, scale, or problem statement.
- Penalise: single-line project entries with no description.
```

---

### strong_weak_examples.md

Before/after pairs used to guide the reformer.

```markdown
# Strong vs Weak Resume Language Examples

## Experience Bullets

### Weak → Strong

| Weak | Strong |
|------|--------|
| Worked on backend systems. | Designed and developed RESTful API endpoints using FastAPI, handling 500+ daily requests. |
| Helped with AWS project. | Assisted in deploying cloud infrastructure using AWS EC2 and S3, supporting a team of 5 engineers. |
| Maintained Docker containers. | Managed Docker containerized microservices in a staging environment, ensuring consistent uptime during testing cycles. |
| Did data analysis. | Performed exploratory data analysis on a 50,000-row dataset using Python and Pandas, identifying key trends for the product team. |
| Worked with team on features. | Collaborated with a cross-functional team of 4 to deliver 3 product features across a 2-month sprint cycle. |

## Summary Statements

### Weak → Strong

| Weak | Strong |
|------|--------|
| I am a hardworking developer looking for opportunities. | Backend developer with 2 years of experience building scalable APIs using Python and FastAPI. |
| Passionate about technology and learning new things. | Results-driven engineer with hands-on experience in cloud deployment, containerization, and CI/CD pipelines. |

## Project Descriptions

### Weak → Strong

| Weak | Strong |
|------|--------|
| Built a website. | Developed a full-stack web application using React and Node.js, enabling users to track personal finance goals. |
| Made a Python script. | Built an automated web scraping tool using Python and BeautifulSoup, collecting and processing 10,000+ product listings daily. |
| Created a ML model. | Trained a sentiment analysis model using scikit-learn on a 20,000-sample dataset, achieving 87% accuracy on test data. |
```

---

### action_verbs.md

Strong action verbs categorised by type, used to improve bullet point language.

```markdown
# Action Verbs by Category

## Development and Engineering
Architected, Built, Coded, Debugged, Deployed, Designed, Developed, Engineered, Implemented, Integrated, Migrated, Optimised, Programmed, Refactored, Shipped, Tested

## Analysis and Research
Analysed, Assessed, Benchmarked, Evaluated, Examined, Identified, Investigated, Measured, Monitored, Profiled, Researched, Reviewed, Tested

## Collaboration and Leadership
Collaborated, Coordinated, Facilitated, Led, Managed, Mentored, Organised, Partnered, Presented, Supervised

## Improvement and Optimisation
Automated, Enhanced, Improved, Optimised, Reduced, Resolved, Standardised, Streamlined, Upgraded

## Creation and Delivery
Built, Created, Delivered, Designed, Developed, Drafted, Generated, Produced, Published, Released

## Data and Reporting
Collected, Compiled, Extracted, Processed, Queried, Reported, Transformed, Visualised
```

---

### skill_evidence_guide.md

How skills should be evidenced across different resume sections.

```markdown
# Skill Evidence Guide

## Evidence Quality Levels

### Level 3 — Strong Evidence (scores as Present)
A skill is strongly evidenced when:
- It appears in an experience bullet that describes specific work done with the skill.
- It appears in a project description with context of how it was applied.
- Example: "Deployed containerised services using Docker on AWS EC2" → Docker and AWS are strongly evidenced.

### Level 2 — Weak Evidence (scores as Weakly Evidenced)
A skill is weakly evidenced when:
- It appears only in the skills list with no supporting context elsewhere.
- It is mentioned once in passing without any task or outcome.
- Example: "Familiar with Kubernetes" in a bullet but no project or experience uses it.

### Level 1 — Not Found
A skill is not found when:
- It does not appear anywhere in the resume.
- Note: Not Found does NOT mean the candidate lacks the skill. It means the resume does not demonstrate it.

## Guidance for Common Skills

### Cloud Skills (AWS, GCP, Azure)
- Weak: Listed in skills section only.
- Strong: Used in a project or experience with specific services mentioned (e.g. EC2, S3, Lambda).

### Programming Languages
- Weak: Listed in skills section only.
- Strong: Used in a project description or experience bullet.

### Frameworks (React, FastAPI, Django)
- Weak: Listed in skills section only.
- Strong: Named in a project with a description of what was built.

### DevOps Tools (Docker, Kubernetes, Terraform)
- Weak: Mentioned without a deployment or infrastructure context.
- Strong: Described in a role or project with a specific use case.
```

---

## 4. Chunking Strategy

| Parameter | Value | Reason |
| :--- | :--- | :--- |
| Chunk size | 512 tokens | Fits within embedding model context, preserves coherent rules |
| Overlap | 50 tokens | Prevents losing context at chunk boundaries |
| Split on | Paragraph / heading boundary | Keeps related rules together |
| Metadata per chunk | source_file, category tag | Enables source filtering if needed |

---

## 5. Embedding and Storage

```python
# Pseudocode — seed_knowledge_base.py

for each .md file in /knowledge_base/:
    text = read_file(file)
    chunks = chunk_text(text, size=512, overlap=50)

    for i, chunk in enumerate(chunks):
        embedding = gemini_embed(chunk)   # 768-dim vector
        store_in_supabase(
            source_file = filename,
            chunk_index = i,
            content     = chunk,
            embedding   = embedding,
            metadata    = { "category": infer_category(filename) }
        )
```

This script runs once at deployment. If `knowledge_base_chunks` table is already populated, it skips. Can be force-rerun to update KB content.

---

## 6. Retrieval at Query Time

```python
# Pseudocode — rag_service.py

def retrieve_relevant_rules(query_text: str, top_k: int = 5) -> List[str]:
    query_embedding = gemini_embed(query_text)

    results = supabase.rpc("match_kb_chunks", {
        "query_embedding": query_embedding,
        "match_count": top_k
    })

    return [row["content"] for row in results]
```

### Prompt Injection Pattern

Retrieved chunks are injected into every LLM prompt as a `[RULES]` block:

```
You are an expert ATS resume analyst.

[RULES]
{retrieved_kb_chunks_joined_with_newlines}

[RESUME]
{resume_text}

[JOB DESCRIPTION]
{job_description}

Task: Analyse the resume against the job description using the rules provided above.
Score each category from 0-100 and list specific issues found.
```

---

## 7. Maintenance

The knowledge base is the only component that requires content curation by the team.

**When to update KB files:**
- When ATS scoring criteria needs adjustment
- When new resume best practices are identified
- When new skill evidence patterns are discovered
- When before/after examples need to be added

**After updating any KB file:**
1. Run `scripts/seed_knowledge_base.py --force` to re-embed
2. Test with a sample resume to verify new rules are being applied
3. Commit updated `.md` files to the repository
