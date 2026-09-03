# ⚡ SkillGap.ai

> **The Intelligent Resume Matching & Upskilling Engine**  
> Bridge the gap between developer profiles and job requirements with vector embeddings, skill differential analysis, and automated learning

## 🎯 Overview

**SkillGap.ai** moves beyond traditional, keyword-stuffed ATS tools. It analyzes candidate profiles against target job descriptions using Natural Language Processing (NLP) to calculate semantic compatibility, highlight missing tech stack requirements, and generate an actionable learning path to close the gap.

---

## 🚀 Key Features

* 📄 **Semantic Parsing:** Deep extraction of skills, tools, and domain concepts using NLP entity recognition.
* 📊 **Precision Match Score:** Cosine similarity scoring powered by vector embeddings (Sentence-BERT).
* 🔬 **Gap Differential Engine:** Automatically pinpoints exact missing frameworks, tools, and cloud skills.
* 🗺️ **Automated Learning Roadmaps:** Generates step-by-step topic hierarchies for missing skills so candidates know exactly what to study next.
* 🔮 **Fit Prediction:** ML classification modeling to evaluate overall hiring fit based on job requirements.

---

## 💻 Tech Stack

| Domain | Tools & Frameworks |
| :--- | :--- |
| **NLP & ML** | Python, spaCy, Hugging Face Transformers, Scikit-learn, Sentence-BERT |
| **Backend & API** | FastAPI / Flask |
| **Frontend UI** | Streamlit / React |
| **Cloud & DevOps** | AWS (S3, Lambda/EC2), Docker, GitHub Actions |

---

## 📊 Example Output

```text
[INPUT]
Target Role : Cloud Engineer
JD Stack    : AWS, Docker, Kubernetes, Terraform
Resume      : AWS, Docker

[SKILLGAP.AI ANALYSIS]
Compatibility Score : 72%
Matched Skills      : [AWS, Docker]
Skill Gaps          : [Kubernetes, Terraform]

[ROADMAP GENERATED]
1. Kubernetes  ──► Architecture ──► Pods & Services ──► Helm Charts
2. Terraform   ──► HCL Syntax ──► State Management ──► Provider Setup
