# JD-Match AI – Resume & Job Description Matching System
A Python multi-agent system for extracting and matching resumes with job descriptions.

## Overview

JD-Match AI automates the resume screening and job matching process using AI agents. It extracts structured data from resumes against job descriptions and compares to see if candidate fit.

## Key Features

- Multi-agent extraction and matching
- Supports PDF, DOCX, TXT, Markdown
- Uses Google Gemini models
- Produces structured data via Pydantic
- Configurable prompts and models

## Tech Stack

- Python 3.8+
- Google Agent Development Kit (ADK)
- Google Gemini
- Pydantic v2
- PyPDF2
- python-docx

## Project Structure

```
Resume_Agent/
├── ResumeAgent/
│   ├── agent.py
│   ├── dtypes_common.py
│   └── sub/
│       ├── agent.py
│       ├── prompts.py
│       └── document_extractor.py
├── INSTALLATION.md
├── requirements.txt
└── README.md
```

## Usage

Basic example:

```python
from ResumeAgent.agent import agent
result = agent.run(input="Extract and match the provided resume with the job description")
print(result)
```

Extract text from a resume:

```python
from ResumeAgent.sub.document_extractor import DocumentExtractorTool
from ResumeAgent.agent import agent

extractor = DocumentExtractorTool()
text = extractor.extract_text("path/to/resume.pdf")
result = agent.run(input=f"Extract candidate information: {text}")
```

## Data Models

Resume data includes profile, work experience, education, skills, and projects.
Job description data includes title, responsibilities, skills, requirements, and perks.

## Configuration

Create `.env`:

```bash
GOOGLE_API_KEY=your_actual_api_key_here
GOOGLE_GENAI_USE_VERTEXAI=FALSE
```

Change model settings in `ResumeAgent/sub/agent.py`.

## Support

- Email: nishantsaxena@example.com
- GitHub: https://github.com/iamnishantsaxena/JD-Match-AI

Last Updated: March 12, 2026
Version: 1.0.0
Python: 3.8+
Status: Active Development
