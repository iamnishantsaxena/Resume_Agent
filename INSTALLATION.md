# Installation Guide

## Prerequisites

- Python 3.8 or higher - [Download here](https://www.python.org/downloads/)
- Google Cloud AI API Access - [Get API Key](https://aistudio.google.com/app/apikey)
- pip (comes with Python)
- Git (optional, for cloning)

## Step 1: Clone or Download Repository

**Option A: Clone with Git**
```bash
git clone https://github.com/iamnishantsaxena/JD-Match-AI.git
cd Resume_Agent
```

**Option B: Download ZIP**
1. Click "Code" → "Download ZIP"
2. Extract the folder
3. Open terminal in the extracted folder

## Step 2: Create and Activate Virtual Environment

**On macOS/Linux:**
```bash
python3 -m venv env
source env/bin/activate
# You should see (env) in your terminal prompt
```

**On Windows (Command Prompt):**
```cmd
python -m venv env
env\Scripts\activate
# You should see (env) in your terminal prompt
```

**On Windows (PowerShell):**
```powershell
python -m venv env
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
env\Scripts\Activate.ps1
# You should see (env) in your terminal prompt
```

**Troubleshooting:**
- If `python` doesn't work, try `python3`
- On Windows, ensure you're in the project directory
- Run `deactivate` first if another environment is active

## Step 3: Install Dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

**If specific packages fail:**
```bash
pip install google-genai pydantic PyPDF2 python-docx google-adk
```

## Step 4: Configure Google API Key

### Get Your API Key

1. Go to [Google AI Studio](https://aistudio.google.com/app/apikey)
2. Click "Create API Key"
3. Copy the key (keep it secret!)

### Create .env File

**On macOS/Linux:**
```bash
touch .env
```

**On Windows:**
```cmd
type nul > .env
```

Then open `.env` and add:
```
GOOGLE_GENAI_USE_VERTEXAI=FALSE
GOOGLE_API_KEY=your_api_key_here
```

Replace `your_api_key_here` with your actual API key.

**Secure your credentials:**
1. Add `.env` to `.gitignore` to prevent accidental uploads
2. Never commit `.env` to version control or share your API key

## Step 5: Verify Installation

```bash
# Test imports
python -c "from google.adk.agents import Agent; from pydantic import BaseModel; print('All imports successful!')"

# Or run Python interactive mode
python
```

Then in Python:
```python
from ResumeAgent.agent import agent
print('Agent imported successfully!')
print('Agent name:', agent.name)
print('Agent model:', agent.model)
exit()
```

You should see success messages. If not, check the troubleshooting section below.

## Troubleshooting

**ModuleNotFoundError: No module named 'google'**
- Activate virtual environment and reinstall requirements
  ```bash
  source env/bin/activate  # macOS/Linux
  env\Scripts\activate     # Windows
  pip install -r requirements.txt
  ```

**API Key not found or GOOGLE_API_KEY error**
- Ensure `.env` file exists in project root with correct key
- Check file is named `.env` (not `.env.txt`)
- Verify key format: `GOOGLE_API_KEY=sk-...`
- Restart Python/IDE after creating `.env`
- Ensure `.env` is in the same directory as `requirements.txt`

**FileNotFoundError: File not found**
- Use absolute paths or ensure files exist
  ```python
  import os
  file_path = os.path.abspath("path/to/file.pdf")
  text = extractor.extract_text(file_path)
  ```

**PDF extraction returns empty/garbled text**
- Some PDFs are image-based (scanned)
- Try converting to text first
- Use `.txt` or `.docx` format instead
- Consider using OCR for scanned documents

**Permission denied on macOS/Linux**
- Update permissions:
  ```bash
  chmod +x env/bin/activate
  ```

**Windows PowerShell won't activate virtual environment**
- Set execution policy:
  ```powershell
  Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
  env\Scripts\Activate.ps1
  ```

**Slow API responses or timeout errors**
- Check internet connection
- Use `gemini-2.0-flash-exp` (faster model)
- Try again (temporary API issues)

**ImportError: cannot import name 'CandidateResume'**
- Check import path - module is in `dtypes_common`, not sub-package
  ```python
  # Correct
  from ResumeAgent.dtypes_common import CandidateResume, JobDescription
  
  # Incorrect
  from ResumeAgent.sub.dtypes_common import CandidateResume
  ```

## Quick Commands

```bash
# Activate environment
source env/bin/activate  # macOS/Linux
env\Scripts\activate     # Windows

# Test setup
python -c "from ResumeAgent.agent import agent; print('OK')"

# Run Python interactive
python
# >>> from ResumeAgent.agent import agent
# >>> result = agent.run(input="...")

# Deactivate environment
deactivate
```

## Need Help?

- Check [Google AI Studio Docs](https://aistudio.google.com/app)
- Review [Pydantic Documentation](https://docs.pydantic.dev/)
- Check existing issues or open a new one on GitHub
