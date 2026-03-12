# 📋 JD-Match AI – Resume & Job Description Matching Engine

An intelligent **multi-agent system** that intelligently extracts, analyzes, and matches resumes with job descriptions using Google's Agent Development Kit and advanced AI models. Perfect for recruitment automation, candidate screening, and HR workflows.

## 🎯 Project Overview

**JD-Match AI** is a sophisticated Python application designed to automate tedious recruitment tasks. It uses AI-powered agents to process resumes and job descriptions, extract structured information, and provide intelligent matching analysis to identify the best-fit candidates.

Instead of manually reviewing resumes against job descriptions, this system does it automatically with high accuracy, saving hours of HR time.

### Key Use Cases
- 🔍 **Candidate Screening**: Quickly identify qualified candidates from resume databases
- 📊 **Skill Matching**: Automatically map candidate skills to job requirements
- 📈 **Recruitment Analytics**: Get detailed compatibility reports between candidates and positions
- 🤖 **Workflow Automation**: Integrate into existing HR systems for automated applicant tracking

## ✨ Features

### 1. **Multi-Agent Architecture**
Modular design with specialized agents for different tasks:
- **Resume Extraction Agent**: Intelligently parses and structures resume data including work experience, education, skills, and projects
- **Job Description Extraction Agent**: Extracts key job requirements, technical/non-technical skills, qualifications, and perks
- **Resume-JD Matcher Agent**: Analyzes compatibility between candidate profiles and job requirements, providing detailed matching reports

### 2. **Multi-Format Document Support**
- ✅ **PDF** documents
- ✅ **DOCX** (Word) documents  
- ✅ **TXT** text files
- ✅ **Markdown** files

### 3. **Advanced NLP & AI**
- Powered by **Google Gemini models** (1.5-pro, 2.0-flash)
- Intelligent text understanding and extraction
- Context-aware analysis and matching

### 4. **Structured Data Output**
- **Pydantic models** for type-safe data structures
- Consistent, validated output formats
- Easy integration with downstream systems

### 5. **Extracted Data Includes**
**From Resumes:**
- Candidate profile (name, email, phone, location)
- Work experience with responsibilities
- Technical and non-technical skills with proficiency levels
- Education and certifications
- Projects and portfolio links

**From Job Descriptions:**
- Job title and responsibilities
- Required technical and non-technical skills
- Qualifications and experience requirements
- Work-from-home and flexibility options
- Perks and benefits

**From Matching:**
- Skill gap analysis
- Experience alignment
- Compatibility percentage
- Detailed match report

### 6. **Customizable & Extensible**
- Flexible instruction sets for each agent
- Easy to modify prompts and extraction rules
- Add custom agents for specialized tasks

### 7. **Secure Configuration**
- Environment variable management for API keys
- `.env` file support for local development
- Safe credential handling

## 🛠️ Technology Stack

| Component | Technology |
|-----------|-----------|
| **Language** | Python 3.8+ |
| **Multi-Agent Framework** | Google Agent Development Kit (ADK) |
| **LLM Model** | Google Gemini (1.5-pro, 2.0-flash-exp) |
| **Data Validation** | Pydantic v2 |
| **PDF Processing** | PyPDF2 |
| **Word Documents** | python-docx |
| **Async HTTP** | aiohttp |
| **Authentication** | google-auth |
| **Google Cloud** | google-cloud-* libraries |

## 📁 Project Structure

```
Resume_Agent/
├── ResumeAgent/                    # Main package
│   ├── __init__.py
│   ├── agent.py                    # Root agent coordinator
│   ├── dtypes_common.py            # Pydantic models for structured data
│   ├── .env                        # Environment variables (local only)
│   └── sub/                        # Sub-agents module
│       ├── __init__.py
│       ├── agent.py                # Sub-agent definitions
│       ├── prompts.py              # Agent instructions & prompts
│       ├── document_extractor.py   # File parsing utility
│       └── __init__.py
├── requirements.txt                # Python dependencies
├── instructions.md                 # Quick reference guide
├── README.md                       # This file
└── .gitignore                      # Git ignore file
```

### Component Breakdown

| File | Purpose |
|------|---------|
| `agent.py` (root) | Coordinates all sub-agents; entry point for API calls |
| `agent.py` (sub) | Defines individual extraction and matching agents |
| `dtypes_common.py` | Pydantic models defining data structures for resumes, JDs, and match results |
| `prompts.py` | Centralized agent instructions for consistent behavior |
| `document_extractor.py` | Handles multi-format document parsing (PDF, DOCX, TXT) |

## 🚀 Getting Started

### Prerequisites

Before you begin, ensure you have:

- ✅ **Python 3.8 or higher** - [Download here](https://www.python.org/downloads/)
- ✅ **Google Cloud AI API Access** - [Get API Key](https://aistudio.google.com/app/apikey)
- ✅ **pip** (comes with Python)
- ✅ **Git** (optional, for cloning)
- ✅ **A code editor** (VS Code recommended)

### Installation Guide

#### Step 1: Clone or Download the Repository

**Option A: Clone with Git**
```bash
git clone https://github.com/iamnishantsaxena/JD-Match-AI.git
cd Resume_Agent
```

**Option B: Download ZIP**
1. Click "Code" → "Download ZIP"
2. Extract the folder
3. Open terminal in the extracted folder

#### Step 2: Create and Activate Virtual Environment

A virtual environment isolates project dependencies from your system Python.

**On macOS:**
```bash
# Create virtual environment
python3 -m venv env

# Activate virtual environment
source env/bin/activate

# You should see (env) in your terminal prompt
```

**On Windows (Command Prompt):**
```cmd
# Create virtual environment
python -m venv env

# Activate virtual environment
env\Scripts\activate

# You should see (env) in your terminal prompt
```

**On Windows (PowerShell):**
```powershell
# Create virtual environment
python -m venv env

# Set execution policy (if needed)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Activate virtual environment
env\Scripts\Activate.ps1

# You should see (env) in your terminal prompt
```

**Troubleshooting Virtual Environment:**
- If `python` command doesn't work, try `python3`
- On Windows, ensure you're in the project directory
- Make sure .venv isn't activated already: run `deactivate` first

#### Step 3: Upgrade pip and Install Dependencies

```bash
# Upgrade pip to latest version
pip install --upgrade pip

# Install all required packages
pip install -r requirements.txt
```

**Expected Output:** You should see packages installing (aiohttp, google-genai, pydantic, etc.)

**Troubleshooting Installation:**
- If you get permission errors on macOS, try `pip install --user -r requirements.txt`
- If specific packages fail, try installing them individually:
  ```bash
  pip install google-genai pydantic PyPDF2 python-docx google-adk
  ```

#### Step 4: Configure Google API Key

**Step 4a: Get Your API Key**
1. Go to [Google AI Studio](https://aistudio.google.com/app/apikey)
2. Click "Create API Key"
3. Copy the key (keep it secret!)

**Step 4b: Create .env File**

**On macOS/Linux:**
```bash
# Create .env file in the project root
touch .env

# Open it in your editor and add the following lines:
```

Then add to the `.env` file:
```
GOOGLE_GENAI_USE_VERTEXAI=FALSE
GOOGLE_API_KEY=your_api_key_here
```

**On Windows:**
1. Open the project folder in Explorer
2. Right-click → New → Text Document
3. Name it `.env` (include the dot)
4. Open it and add:
```
GOOGLE_GENAI_USE_VERTEXAI=FALSE
GOOGLE_API_KEY=your_api_key_here
```
5. Replace `your_api_key_here` with your actual API key

**On Windows (PowerShell/Command Prompt):**
```powershell
# Create .env file
New-Item -Path ".env" -ItemType File

# Edit with default editor
notepad .env
```

**Step 4c: Secure Your Credentials**

Add `.env` to your `.gitignore` file to prevent accidental uploads:

1. Open `.gitignore` in a text editor
2. Add a new line: `.env`
3. Save the file

**Important:** ⚠️ Never commit `.env` to version control or share your API key!

#### Step 5: Verify Installation

Run this test to confirm everything works:

```bash
# Test imports
python -c "from google.adk.agents import Agent; from pydantic import BaseModel; print('✅ All imports successful!')"

# Or run Python interactive mode
python
```

Then in Python:
```python
from ResumeAgent.agent import agent
print('✅ Agent imported successfully!')
print('Agent name:', agent.name)
print('Agent model:', agent.model)
exit()
```

You should see success messages. If not, check the troubleshooting section.

## 💡 Usage & Quick Start

### Basic Usage Example

```python
from ResumeAgent.agent import agent

# Run the agent to extract and match documents
result = agent.run(
    input="Please extract and match the provided resume with the job description"
)

# Access results
print(result)
```

### Complete Workflow Example

```python
from ResumeAgent.agent import agent
import json

# 1. Extract resume
resume_result = agent.run(
    input="""
    Extract the candidate's information from this resume:
    [Your resume text here]
    """
)
print("Resume Extracted:")
print(json.dumps(resume_result, indent=2))

# 2. Extract job description
jd_result = agent.run(
    input="""
    Extract the job requirements from this job description:
    [Your JD text here]
    """
)
print("Job Description Extracted:")
print(json.dumps(jd_result, indent=2))

# 3. Match resume with job description
match_result = agent.run(
    input="""
    Compare this candidate with the job and provide a match analysis:
    Resume: [extracted resume data]
    Job Description: [extracted JD data]
    """
)
print("Match Result:")
print(json.dumps(match_result, indent=2))
```

### Using with Document Files

```python
from ResumeAgent.sub.document_extractor import DocumentExtractorTool
from ResumeAgent.agent import agent

# Initialize the document extractor
extractor = DocumentExtractorTool()

# Extract text from various file formats
resume_text = extractor.extract_text("path/to/resume.pdf")
jd_text = extractor.extract_text("path/to/job_description.docx")

# Use extracted text with agent
result = agent.run(
    input=f"""
    Extract and match:
    Resume: {resume_text}
    Job Description: {jd_text}
    """
)

print(result)
```

### Accessing Extracted Data

The agents return structured data based on Pydantic models:

```python
from ResumeAgent.dtypes_common import JobDescription, CandidateResume

# The extracted data follows these models:
# - JobDescription: Contains job title, skills, requirements, perks, etc.
# - CandidateResume: Contains profile, experience, education, skills, etc.

# Example accessing structured data:
job_desc = JobDescription(...)  # From agent output
print(f"Job Title: {job_desc.jobTitle}")
print(f"Required Experience: {job_desc.Requirements.minYearsExperienceRequired} years")
print(f"Technical Skills: {[skill.name for skill in job_desc.Skills.Technical_Skills]}")
```

## 📊 Data Structures

### Resume Data Model

```python
CandidateResume {
    profile: {
        name, email, phone, linkedin_url, portfolio_url, location
    },
    workExperience: [{
        job_title, company, location, start_date, end_date, responsibilities
    }],
    education: [{
        institution, degree, field_of_study, start_date, graduation_date
    }],
    skills: {
        technical_skills: [{name, level, years_experience, evidence}],
        non_technical_skills: [{name, level, years_experience, evidence}]
    },
    projects: [{
        name, description, technologies, link
    }],
    certifications: [...]
}
```

### Job Description Data Model

```python
JobDescription {
    jobTitle,
    responsibilities: [...],
    skills: {
        technical_skills: [{name, requirement, experience_level}],
        non_technical_skills: [{name, requirement, experience_level}]
    },
    requirements: {
        qualification, work_rights, min_years_experience
    },
    flexible_arrangement: {
        work_from_home: {available: YES/NO/Not Mentioned, days: 0-5}
    },
    perks: [...],
    extra_details: {...}
}
```

## 🧠 How It Works

### Agent Flow

```
┌─────────────────────────────────────┐
│      Root Coordinator Agent         │
│  Orchestrates sub-agents workflow   │
└──────────────┬──────────────────────┘
               │
        ┌──────┼──────┐
        │      │      │
        ▼      ▼      ▼
┌──────────┐ ┌──────────┐ ┌──────────────┐
│ Resume   │ │   JD     │ │   Matcher    │
│Extractor │ │Extractor │ │   Agent      │
└──────────┘ └──────────┘ └──────────────┘
        │      │              │
        └──────┼──────────────┘
               │
               ▼
        ┌─────────────────┐
        │  Structured     │
        │  Output Data    │
        └─────────────────┘
```

**Workflow Steps:**
1. **Input Documents**: Resume and Job Description (PDF, DOCX, TXT)
2. **Document Extraction**: Convert files to text using DocumentExtractorTool
3. **Resume Agent**: Extracts candidate information into structured Pydantic format
4. **JD Agent**: Extracts job requirements into structured Pydantic format
5. **Matcher Agent**: Analyzes compatibility and generates detailed report
6. **Output**: Structured data + match analysis with compatibility scores

## 🔧 Configuration Options

### Environment Variables

Create a `.env` file in the project root:

```bash
# Required - Get your key from https://aistudio.google.com/app/apikey
GOOGLE_API_KEY=your_actual_api_key_here

# Optional (defaults shown)
GOOGLE_GENAI_USE_VERTEXAI=FALSE
```

### Customizing Agent Models

Edit `ResumeAgent/sub/agent.py` to change LLM models:

```python
# Current models (gemini-2.0-flash is faster and cheaper)
resume_extractor_agent = create_extraction_agent(
    model="gemini-2.0-flash-exp"  # Change here
)

jd_extractor_agent = create_extraction_agent(
    model="gemini-2.0-flash-exp"  # Change here
)

# Available models:
# - "gemini-2.0-flash-exp"      # Fast, cheaper, recommended
# - "gemini-1.5-pro-latest"     # More capable, slower, more expensive
# - "gemini-1.5-flash"          # Balanced option
```

### Customizing Agent Prompts

Edit `ResumeAgent/sub/prompts.py` to modify extraction instructions:

```python
resume_extractor_prompt = """
Your custom instructions here...
"""

jd_entity_extraction_prompt = """
Your custom instructions here...
"""

resume_jd_matcher_prompt = """
Your custom instructions here...
"""
```

## 🐛 Troubleshooting

### Common Issues & Solutions

**Issue: `ModuleNotFoundError: No module named 'google'`**
- ✅ Solution: Activate virtual environment and reinstall requirements
  ```bash
  source env/bin/activate  # macOS/Linux
  env\Scripts\activate     # Windows
  pip install -r requirements.txt
  ```

**Issue: `API Key not found` or `GOOGLE_API_KEY` error**
- ✅ Solution: Ensure `.env` file exists in project root with correct key
  - Check file is named `.env` (not `.env.txt`)
  - Verify key format: `GOOGLE_API_KEY=sk-...`
  - Restart Python/IDE after creating `.env`
  - Ensure `.env` is in the same directory as `requirements.txt`

**Issue: `FileNotFoundError: File not found: ...`**
- ✅ Solution: Use absolute paths or ensure files exist
  ```python
  import os
  file_path = os.path.abspath("path/to/file.pdf")
  text = extractor.extract_text(file_path)
  ```

**Issue: PDF extraction returns empty/garbled text**
- ✅ Solution: Some PDFs are image-based (scanned)
  - Try converting to text first
  - Use `.txt` or `.docx` format instead
  - Consider using OCR for scanned documents

**Issue: `Permission denied` on macOS/Linux**
- ✅ Solution: Update permissions
  ```bash
  chmod +x env/bin/activate
  ```

**Issue: Windows PowerShell won't activate virtual environment**
- ✅ Solution: Set execution policy
  ```powershell
  Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
  env\Scripts\Activate.ps1
  ```

**Issue: Slow API responses or timeout errors**
- ✅ Solution: 
  - Check internet connection
  - Use `gemini-2.0-flash-exp` (faster model)
  - Try again (temporary API issues)

**Issue: `ImportError: cannot import name 'CandidateResume'`**
- ✅ Solution: Check import path - module is in `dtypes_common`, not sub-package
  ```python
  # Correct
  from ResumeAgent.dtypes_common import CandidateResume, JobDescription
  
  # Incorrect
  from ResumeAgent.sub.dtypes_common import CandidateResume
  ```

### Getting Help

- 📖 Check [Google AI Studio Docs](https://aistudio.google.com/app)
- 🐍 Review [Pydantic Documentation](https://docs.pydantic.dev/)
- 💬 Check existing issues or open a new one on GitHub

## 📝 Example Use Cases

### 1. Candidate Screening Pipeline
```python
# Batch process multiple candidates
from ResumeAgent.sub.document_extractor import DocumentExtractorTool
from ResumeAgent.agent import agent

extractor = DocumentExtractorTool()
candidates = ["alice.pdf", "bob.docx", "charlie.txt"]

for resume_file in candidates:
    try:
        resume_text = extractor.extract_text(resume_file)
        match = agent.run(input=f"Score candidate match: {resume_text}")
        print(f"{resume_file}: {match}")
    except Exception as e:
        print(f"Error processing {resume_file}: {e}")
```

### 2. Skill Gap Analysis
```python
# Identify missing skills
jd_skills = [s.name for s in job_desc['skills']['technical_skills']]
resume_skills = [s.name for s in resume_data['skills']['technical_skills']]
missing_skills = set(jd_skills) - set(resume_skills)
print(f"Candidate needs to learn: {missing_skills}")
```

### 3. Integration with ATS
```python
# Store results in database
import json
from datetime import datetime

result = agent.run(input="Extract and match...")
output = {
    "timestamp": datetime.now().isoformat(),
    "candidate": result.get('match_result', {}),
    "match_score": result.get('score', 0)
}

with open('match_results.json', 'w') as f:
    json.dump(output, f, indent=2)
```

### 4. Batch Resume Analysis
```python
# Process all resumes in a folder
import os
from pathlib import Path

resume_folder = Path("resumes/")
matches = []

for resume_file in resume_folder.glob("*.pdf"):
    text = extractor.extract_text(str(resume_file))
    result = agent.run(input=f"Extract resume: {text}")
    matches.append({"file": resume_file.name, "data": result})

# Save to CSV or database
import csv
with open("candidates.csv", "w") as f:
    writer = csv.DictWriter(f, fieldnames=["file", "name", "skills", "experience"])
    for m in matches:
        writer.writerow(m)
```

## 📜 Dependencies

Key packages (see `requirements.txt` for full list):

| Package | Version | Purpose |
|---------|---------|---------|
| `google-genai` | ^1.20.0 | Google Gemini API access |
| `google-adk` | ^1.2.1 | Agent Development Kit |
| `pydantic` | ^2.0 | Data validation & models |
| `PyPDF2` | - | PDF text extraction |
| `python-docx` | - | DOCX document handling |
| `google-cloud-*` | - | Google Cloud services |
| `aiohttp` | - | Async HTTP client |
| `google-auth` | - | Google authentication |

## 📄 License & Attribution

This project is built with:
- 🤖 [Google Generative AI](https://ai.google.dev/)
- 📦 [Pydantic](https://docs.pydantic.dev/)
- 🔧 [Python](https://www.python.org/)
- Open source community libraries

## 🤝 Contributing

Contributions welcome! Areas for improvement:
- OCR support for scanned documents
- Support for more document formats (RTF, HTML)
- Enhanced matching algorithms
- Performance optimizations
- Unit test coverage
- API endpoint wrapper

## ❓ FAQ

**Q: Can I use this commercially?**
A: Check Google AI's terms of service regarding API usage in production. Free tier has rate limits.

**Q: Is my data secure?**
A: Data is sent to Google's API. For sensitive data, review Google's privacy policy and consider running with open-source LLMs.

**Q: Can I use other LLMs besides Gemini?**
A: Current implementation uses Google Gemini. OpenAI/other models would require architecture changes to the agent framework.

**Q: What's the cost?**
A: Google Generative AI has a free tier (see [pricing](https://ai.google.dev/pricing)). Production usage incurs costs.

**Q: Can I run this offline?**
A: Current version requires internet for Google API. Local LLM support could be added in future versions.

**Q: How long does matching take?**
A: Typically 5-15 seconds per resume-JD pair depending on document size and API load.

**Q: What file size limits exist?**
A: No hard limits, but very large documents (100+ pages) may timeout. Test with your document sizes.

## 📞 Support

- 📧 Email: nishantsaxena@example.com
- 🐙 GitHub: [iamnishantsaxena/JD-Match-AI](https://github.com/iamnishantsaxena/JD-Match-AI)
- 🐛 Issues: Report bugs on GitHub Issues
- 💬 Discussions: GitHub Discussions for questions

## 🚀 Quick Reference Commands

```bash
# Setup
python3 -m venv env
source env/bin/activate  # macOS/Linux
# OR
env\Scripts\activate  # Windows

# Install
pip install --upgrade pip
pip install -r requirements.txt

# Create .env
touch .env  # macOS/Linux
# OR
type nul > .env  # Windows

# Test
python -c "from ResumeAgent.agent import agent; print('OK')"

# Run
python
# >>> from ResumeAgent.agent import agent
# >>> result = agent.run(input="...")

# Deactivate environment
deactivate
```

## 📅 Roadmap

- [ ] REST API wrapper
- [ ] Web dashboard UI
- [ ] Batch processing API
- [ ] Database integration
- [ ] Authentication & multi-user support
- [ ] Custom extraction rules
- [ ] Export to multiple formats (CSV, JSON, PDF)
- [ ] Performance benchmarking
- [ ] Docker containerization
- [ ] Unit tests & CI/CD

---

**Last Updated:** March 12, 2026
**Version:** 1.0.0
**Python:** 3.8+
**Status:** Active Development ✅
