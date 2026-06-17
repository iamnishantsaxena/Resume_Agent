# JD-Match AI – Resume & Job Description Matching Engine

An intelligent multi-agent system that intelligently extracts, analyzes, and matches resumes with job descriptions using Google's Agent Development Kit and advanced AI models. Perfect for recruitment automation, candidate screening, and HR workflows.

## Project Overview

JD-Match AI is a sophisticated Python application designed to automate tedious recruitment tasks. It uses AI-powered agents to process resumes and job descriptions, extract structured information, and provide intelligent matching analysis to identify the best-fit candidates.

Instead of manually reviewing resumes against job descriptions, this system does it automatically with high accuracy, saving hours of HR time.

## Key Features

- Multi-Agent Architecture with specialized agents for different tasks
- Multi-Format Document Support (PDF, DOCX, TXT, Markdown)
- Advanced NLP & AI powered by Google Gemini models (1.5-pro, 2.0-flash)
- Structured Data Output using Pydantic models
- Comprehensive Extraction including candidate profile, work experience, skills, education, projects, and job requirements
- Customizable & Extensible with flexible instruction sets
- Secure Configuration with environment variable management for API keys

## Technology Stack

| Component | Technology |
|-----------|-----------|
| Language | Python 3.8+ |
| Framework | Google Agent Development Kit (ADK) |
| LLM | Google Gemini (1.5-pro, 2.0-flash-exp) |
| Data Validation | Pydantic v2 |
| PDF Processing | PyPDF2 |
| Word Documents | python-docx |

## Project Structure

```
Resume_Agent/
├── ResumeAgent/
│   ├── agent.py                 # Root agent coordinator
│   ├── dtypes_common.py         # Pydantic models
│   └── sub/
│       ├── agent.py             # Sub-agent definitions
│       ├── prompts.py           # Agent instructions
│       └── document_extractor.py # File parsing utility
├── INSTALLATION.md              # Detailed setup guide
├── requirements.txt             # Python dependencies
└── README.md                    # This file
```

## Quick Start

### Installation

See [INSTALLATION.md](INSTALLATION.md) for detailed setup instructions.

**Quick summary:**
```bash
# Clone and setup
git clone https://github.com/iamnishantsaxena/JD-Match-AI.git
cd Resume_Agent

# Create virtual environment
python3 -m venv env
source env/bin/activate  # macOS/Linux or env\Scripts\activate on Windows

# Install dependencies
pip install -r requirements.txt

# Configure API key
echo "GOOGLE_API_KEY=your_api_key_here" > .env

# Verify
python -c "from ResumeAgent.agent import agent; print('OK')"
```

## Usage

### Basic Example

```python
from ResumeAgent.agent import agent

result = agent.run(
    input="Extract and match the provided resume with the job description"
)
print(result)
```

### Extract Resume

```python
from ResumeAgent.sub.document_extractor import DocumentExtractorTool
from ResumeAgent.agent import agent

extractor = DocumentExtractorTool()
resume_text = extractor.extract_text("path/to/resume.pdf")

result = agent.run(
    input=f"Extract candidate information: {resume_text}"
)
```

### Batch Processing

```python
from ResumeAgent.sub.document_extractor import DocumentExtractorTool
from ResumeAgent.agent import agent
from pathlib import Path

extractor = DocumentExtractorTool()
results = []

for resume_file in Path("resumes/").glob("*.pdf"):
    text = extractor.extract_text(str(resume_file))
    result = agent.run(input=f"Extract resume: {text}")
    results.append({"file": resume_file.name, "data": result})
```

## Data Models

### Resume Data

```python
CandidateResume {
    profile: {name, email, phone, linkedin_url, location},
    workExperience: [{job_title, company, responsibilities}],
    education: [{institution, degree, field_of_study}],
    skills: {technical_skills, non_technical_skills},
    projects: [{name, description, technologies}]
}
```

### Job Description Data

```python
JobDescription {
    jobTitle,
    responsibilities: [...],
    skills: {technical_skills, non_technical_skills},
    requirements: {qualification, min_years_experience},
    perks: [...]
}
```

## Configuration

### Environment Variables

Create `.env` in project root:

```bash
GOOGLE_API_KEY=your_actual_api_key_here
GOOGLE_GENAI_USE_VERTEXAI=FALSE
```

### Customizing Models

Edit `ResumeAgent/sub/agent.py`:

```python
# Available models:
# - "gemini-2.0-flash-exp"      # Fast, cheaper, recommended
# - "gemini-1.5-pro-latest"     # More capable, slower
# - "gemini-1.5-flash"          # Balanced option
```

### Customizing Prompts

Edit `ResumeAgent/sub/prompts.py` to modify extraction instructions.

## FAQ

**Q: Can I use this commercially?**
A: Check Google AI's terms of service regarding API usage in production. Free tier has rate limits.

**Q: Is my data secure?**
A: Data is sent to Google's API. For sensitive data, review Google's privacy policy.

**Q: Can I use other LLMs besides Gemini?**
A: Current implementation uses Google Gemini. Other models would require architecture changes.

**Q: What's the cost?**
A: Google Generative AI has a free tier. Production usage incurs costs.

**Q: Can I run this offline?**
A: Current version requires internet for Google API.

**Q: How long does matching take?**
A: Typically 5-15 seconds per resume-JD pair depending on document size and API load.

## Example Use Cases

1. **Candidate Screening**: Quickly identify qualified candidates from resume databases
2. **Skill Matching**: Automatically map candidate skills to job requirements
3. **Recruitment Analytics**: Get detailed compatibility reports between candidates and positions
4. **Workflow Automation**: Integrate into existing HR systems for automated applicant tracking

## Support

- Email: nishantsaxena@example.com
- GitHub: [iamnishantsaxena/JD-Match-AI](https://github.com/iamnishantsaxena/JD-Match-AI)
- Issues: Report bugs on GitHub Issues
- Docs: Check [Google AI Studio](https://aistudio.google.com/app) and [Pydantic](https://docs.pydantic.dev/)

## Roadmap

- [ ] REST API wrapper
- [ ] Web dashboard UI
- [ ] Batch processing API
- [ ] Database integration
- [ ] Custom extraction rules
- [ ] Export to multiple formats (CSV, JSON, PDF)
- [ ] Docker containerization
- [ ] Unit tests & CI/CD

---

Last Updated: March 12, 2026
Version: 1.0.0
Python: 3.8+
Status: Active Development
