# JD-Match AI: Code Walkthrough for Interviews

## How Data Flows Through the System

### Example: Processing a Resume + Job Description

**Input**: 
- `resume.pdf` 
- `job_description.txt`

**Expected Output**:
```json
{
  "overallRating": "Good Match",
  "skillsMatchRating": "Excellent",
  "experienceMatchRating": "Good",
  "keyStrengths": ["5+ years Python", "Led team of 8"],
  "keyWeaknesses": ["No AWS experience"]
}
```

---

## Step 1: Document Extraction

**File**: `ResumeAgent/sub/document_extractor.py`

```python
from ResumeAgent.sub.document_extractor import DocumentExtractorTool

extractor = DocumentExtractorTool()

# Extract text from resume PDF
resume_text = extractor.extract_text("resume.pdf")
# Output: Raw text from PDF

# Extract text from job description
jd_text = extractor.extract_text("job_description.txt")
# Output: Raw text from TXT file
```

**How it Works**:
1. Detects file extension (`.pdf`, `.docx`, `.txt`)
2. Routes to appropriate parser:
   - **PDF**: Uses `PyPDF2` to extract text page-by-page
   - **DOCX**: Uses `python-docx` to iterate paragraphs
   - **TXT/MD**: Native Python file reading
3. Returns raw text as string

**Code Details**:
```python
@staticmethod
def _extract_from_pdf(file_path: str) -> str:
    """Extract text from PDF file."""
    text = ""
    try:
        with open(file_path, 'rb') as file:
            pdf_reader = PyPDF2.PdfReader(file)
            for page_num in range(len(pdf_reader.pages)):
                page = pdf_reader.pages[page_num]
                text += page.extract_text() + "\n"
    except Exception as e:
        raise ValueError(f"Failed to extract PDF: {str(e)}")
    return text
```

**Key Points**:
- Handles errors gracefully (raises with clear message)
- Page-by-page extraction (concatenates all pages)
- No format-specific logic outside this class

---

## Step 2: Resume Extraction Agent

**File**: `ResumeAgent/sub/agent.py` + `ResumeAgent/sub/prompts.py`

```python
from ResumeAgent.sub.agent import resume_extractor_agent

# Pass the raw resume text to the agent
result = resume_extractor_agent.run(input=resume_text)
```

**What Happens Inside**:
1. Agent receives raw resume text
2. Applies `resume_extractor_prompt` (from `prompts.py`)
3. Sends to Gemini 2.0 Flash with instruction:
   ```
   "You are an Entity Extraction Engine. Extract resume data 
   and return JSON matching this schema: {...}"
   ```
4. LLM returns JSON string
5. JSON is validated against `CandidateResume` Pydantic model
6. Returns Python object with full type safety

**Agent Creation** (Factory Pattern):
```python
def create_extraction_agent(name, description, instruction, 
                          model="gemini-2.0-flash-exp", tool=None):
    tools = [tool] if tool else []
    return Agent(
        name=name,
        description=description,
        model=model,
        instruction=instruction,
        tools=tools
    )

# Create resume extractor agent
resume_extractor_agent = create_extraction_agent(
    name="resume_extractor_agent",
    description="The agent that extracts and structures resume information from text",
    instruction=resume_extractor_prompt,
    tool=resume_file_extractor_tool  # DocumentExtractorTool
)
```

**Output**: `CandidateResume` Object
```python
CandidateResume(
    candidate_profile=CandidateProfileData(
        name="John Doe",
        email="john@example.com",
        phone="+1234567890",
        linkedin_url="linkedin.com/in/johndoe",
        location="San Francisco, CA"
    ),
    work_experience=[
        WorkExperienceEntry(
            job_title="Senior Software Engineer",
            company="Tech Corp",
            start_date="2020-01-01",
            end_date="Present",
            responsibilities=["Led team of 8", "Architected microservices"]
        ),
        # ... more jobs
    ],
    skills=CandidateSkillsData(
        technical_skills=[
            CandidateSkillDetail(
                name="Python",
                level="Advanced",
                validated=True,
                years_experience=5,
                evidence=["Job: Senior Software Engineer @ Tech Corp"]
            ),
            # ... more skills
        ],
        non_technical_skills=[...]
    ),
    # ... education, projects, certifications, etc
)
```

---

## Step 3: Job Description Extraction Agent

**File**: `ResumeAgent/sub/agent.py` + `ResumeAgent/sub/prompts.py`

```python
from ResumeAgent.sub.agent import jd_extractor_agent

# Pass the raw JD text to the agent
result = jd_extractor_agent.run(input=jd_text)
```

**What Happens**:
1. Agent receives raw job description text
2. Applies `jd_entity_extraction_prompt`
3. Sends to Gemini 2.0 Flash
4. Returns JSON validated against `JobDescription` Pydantic model

**Output**: `JobDescription` Object
```python
JobDescription(
    job_title="Senior Python Engineer",
    responsibilities=[
        "Design and implement scalable APIs",
        "Mentor junior developers",
        "Lead code reviews"
    ],
    skills=SkillsData(
        technical_skills=[
            TechnicalSkillDetail(
                name="Python",
                description="Core language for backend services",
                requirement="Must Have",
                experience_level_required="5+ years",
                is_key_skill=True
            ),
            TechnicalSkillDetail(
                name="AWS",
                description="Cloud infrastructure",
                requirement="Nice to Have",
                experience_level_required="2+ years",
                is_key_skill=False
            )
        ],
        non_technical_skills=[
            NonTechnicalSkillDetail(
                name="Team Leadership",
                description="Lead and mentor engineers",
                requirement="Must Have",
                experience_level_required="3+ years"
            )
        ]
    ),
    requirements=RequirementsData(
        qualification="Bachelor's Degree in CS or related field",
        work_rights="Eligible to work in US",
        min_years_experience_required=5
    ),
    flexible_arrangement=FlexibleArrangementData(
        work_from_home=WorkFromHomeData(
            available="YES",
            days=3
        )
    ),
    perks=["Health insurance", "401k matching", "Unlimited PTO"]
)
```

---

## Step 4: Matching Agent

**File**: `ResumeAgent/sub/agent.py` + `ResumeAgent/sub/prompts.py`

```python
from ResumeAgent.sub.agent import resume_jd_matcher_agent

# Pass both structured objects to matcher
result = resume_jd_matcher_agent.run(
    input={
        "resume": candidate_resume,  # From step 2
        "job_description": job_description  # From step 3
    }
)
```

**Matching Logic** (Handled by Gemini):
1. Agent receives both `CandidateResume` and `JobDescription` as JSON
2. Applies `resume_jd_matcher_prompt` with instruction:
   ```
   "Compare these two objects. Rate match quality across:
   - Skills (40% weight)
   - Experience (25%)
   - Qualifications (20%)
   - Location (10%)
   - Work Rights (5%)"
   ```
3. Returns JSON validated against `MatchResult` model

**Output**: `MatchResult` Object
```python
class MatchResult(BaseModel):
    overall_rating: Literal["Excellent Match", "Good Match", "Fair Match", "Poor Match"]
    skills_match_rating: Literal["Excellent", "Good", "Fair", "Poor"]
    experience_match_rating: Literal["Excellent", "Good", "Fair", "Poor"]
    qualifications_match_rating: Literal["Meets", "Exceeds", "Partially Meets", "Does Not Meet"]
    location_preference_match_rating: Literal["Exact Match", "Compatible", "Mismatch"]
    work_rights_match_rating: Literal["Confirmed Match", "Likely Match", "Mismatch"]
    key_strengths: List[str]
    key_weaknesses: List[str]
```

**Example Output**:
```python
MatchResult(
    overall_rating="Good Match",
    skills_match_rating="Excellent",
    experience_match_rating="Good",
    qualifications_match_rating="Meets",
    location_preference_match_rating="Compatible",
    work_rights_match_rating="Confirmed Match",
    key_strengths=[
        "5+ years Python (exceeds requirement of 5 years)",
        "Led team of 8 (demonstrates leadership for mentoring role)",
        "AWS experience aligns with nice-to-have",
        "Remote preference matches 3 days/week WFH"
    ],
    key_weaknesses=[
        "No Kubernetes experience mentioned (minor, not required)"
    ]
)
```

---

## Step 5: Root Agent Orchestration

**File**: `ResumeAgent/agent.py`

```python
from ResumeAgent.agent import agent

# User provides resume + JD in any format
result = agent.run(
    input="Here is my resume: [PDF content] and here is the job: [TXT content]"
)
```

**What Root Agent Does**:
1. Receives user input (can be resume, JD, or both)
2. Classifies what type of document(s) provided
3. If only resume: Extracts just the resume
4. If only JD: Extracts just the JD
5. If both: Extracts both, then runs matcher
6. Aggregates results and presents to user

**Root Agent Definition**:
```python
root_agent = Agent(
    name="root_agent",
    description=(
        "Root agent that coordinates multiple sub-agents. "
        "It extracts job descriptions, extracts resumes, and performs matching."
    ),
    model="gemini-1.5-pro-latest",  # Better reasoning model
    instruction=root_agent_prompt,
    tools=[
        AgentTool(agent=jd_extractor_agent),
        AgentTool(agent=resume_extractor_agent),
        AgentTool(agent=resume_jd_matcher_agent)
    ],
    output_key="match_result",
)
```

---

## Data Validation with Pydantic

**File**: `ResumeAgent/dtypes_common.py`

### How Validation Works

```python
from ResumeAgent.dtypes_common import CandidateResume

# LLM returns JSON
llm_output = {
    "candidate_profile": {
        "name": "John Doe",
        "email": "john@example.com",
        "phone": "+1234567890",
        "linkedin_url": "linkedin.com/in/johndoe",
        "location": "San Francisco"
    },
    "skills": {
        "technical_skills": [
            {
                "name": "Python",
                "level": "Advanced",  # ← Must be one of 4 options
                "validated": True,
                "years_experience": 5  # ← Must be int
                "evidence": ["Job: Senior SE @ Tech Corp"]
            }
        ]
    },
    # ... other fields
}

# Pydantic validates on initialization
try:
    resume = CandidateResume(**llm_output)
    # ✓ Success - all fields valid
except ValidationError as e:
    # ✗ Failed - malformed data
    print(e.errors())
    # [
    #   {
    #     'loc': ('skills', 'technical_skills', 0, 'level'),
    #     'msg': "Input should be 'Beginner', 'Intermediate', 'Advanced', or 'Expert'",
    #     'type': 'enum'
    #   }
    # ]
```

### Field-Level Validation

```python
class CandidateSkillDetail(BaseModel):
    name: str  # ← Required
    level: Literal["Beginner", "Intermediate", "Advanced", "Expert"]  # ← Constrained enum
    years_experience: int = Field(..., ge=0)  # ← Non-negative
    validated: bool  # ← True or False only
    evidence: List[str] = Field(default_factory=list)  # ← Optional

# Examples of what fails:
CandidateSkillDetail(name="Python", level="Advanced", years_experience="five")  
# ✗ "five" is not int

CandidateSkillDetail(name="Python", level="Expert", years_experience=-5)
# ✗ Negative years not allowed

CandidateSkillDetail(name="Python")
# ✗ Missing required fields: level, validated
```

---

## Prompt Engineering Deep Dive

**File**: `ResumeAgent/sub/prompts.py`

### Resume Extractor Prompt Structure

```python
resume_extractor_prompt = """
## SYSTEM PROMPT: Advanced Resume Entity Extraction Engine (v1)

**Your Persona:** 
You are a highly sophisticated, exceptionally thorough, and specialized 
Entity Extraction Engine. Your specific expertise lies in parsing and analyzing 
unstructured text from candidate resumes.

**Core Directive:** 
Perform an exhaustive, multi-pass analysis of the provided resume text. 
Identify, extract, validate, and categorize specific information including:
- Candidate contact details
- Professional summary
- Detailed work history (with validation and experience estimation)
- Educational background
- Project experience
- Technical and non-technical skills
- Certifications
- Location information
- Work preferences
- Language proficiency

**Output:** 
You MUST produce ONLY a single, valid JSON object as your output.

[Example JSON structure provided...]

**Edge Cases:**
- If dates are missing, use approximate timeframes
- If years of experience unclear, estimate from job progression
- Only include skills WITH evidence from work experience or projects
- Mark skills as validated=true only if found in descriptions
"""
```

**Key Elements**:
1. **Persona**: Sets expectations for expertise and precision
2. **Directive**: Clear primary task
3. **Format**: Specifies JSON output
4. **Examples**: Shows exact desired format
5. **Edge Cases**: Handles ambiguity

### Why This Matters

```python
# ✗ Bad prompt
"Extract resume data and return JSON"

# ✓ Good prompt
"You are a resume parsing expert. Extract data into JSON. 
If date is missing, use X. If skill unvalidated, mark as Y. 
Return only valid JSON."
```

LLMs respond better to:
- Clear role definition
- Specific examples
- Edge case handling
- Format specification

---

## Error Handling Pattern

**Throughout the codebase**:

```python
# Document extraction with error handling
try:
    text = extractor.extract_text("resume.pdf")
except FileNotFoundError:
    return {"error": "File not found", "status": "failed"}
except ValueError as e:
    return {"error": str(e), "status": "extraction_failed"}

# Agent execution with fallback
try:
    result = resume_extractor_agent.run(input=text)
except Exception as e:
    # Graceful degradation - return what we can
    result = CandidateResume(
        candidate_profile=None,
        skills={"technical_skills": [], "non_technical_skills": []},
        # ... minimal default data
    )
    log_error(e)
```

**Philosophy**: Don't crash entire pipeline on one field missing. Extract what's available, flag low-confidence data.

---

## Configuration Management

**File**: `.env` (not in repo, created locally)

```bash
# .env file
GOOGLE_API_KEY=your_key_here
GOOGLE_GENAI_USE_VERTEXAI=FALSE
```

**Usage in Code**:
```python
import os
from dotenv import load_dotenv

load_dotenv()  # Load from .env

api_key = os.getenv("GOOGLE_API_KEY")
if not api_key:
    raise ValueError("GOOGLE_API_KEY not set")
```

**Security Best Practices**:
- Never commit `.env` to git
- Add `.env` to `.gitignore`
- Use environment variables in production
- Use secrets manager (Google Secret Manager, AWS Secrets Manager)

---

## Testing Patterns (Not Yet Implemented, But Recommended)

### Integration Test Example

```python
def test_resume_extraction_end_to_end():
    """Test full pipeline with real resume"""
    
    # Load sample resume
    sample_resume_text = load_file("tests/samples/resume_001.txt")
    
    # Run extraction
    result = resume_extractor_agent.run(input=sample_resume_text)
    
    # Assertions on structure
    assert isinstance(result, CandidateResume)
    assert result.candidate_profile.name is not None
    assert len(result.skills.technical_skills) > 0
    
    # Assertions on data quality
    for skill in result.skills.technical_skills:
        assert skill.level in ["Beginner", "Intermediate", "Advanced", "Expert"]
        assert skill.years_experience >= 0

def test_matching_logic():
    """Test matching between known resume and JD"""
    
    # Load sample data
    resume = load_sample_resume()
    jd = load_sample_jd()
    
    # Run match
    result = resume_jd_matcher_agent.run(
        input={"resume": resume, "job_description": jd}
    )
    
    # Verify result structure
    assert result.overall_rating in ["Excellent Match", "Good Match", "Fair Match", "Poor Match"]
    assert len(result.key_strengths) > 0 or len(result.key_weaknesses) > 0
```

### Prompt Version Testing

```python
def test_prompt_versions():
    """Compare old vs new prompt accuracy"""
    
    test_set = load_golden_dataset()
    
    # Test v1 prompt
    v1_results = [
        test_prompt_accuracy(RESUME_EXTRACTOR_V1, case)
        for case in test_set
    ]
    v1_accuracy = sum(v1_results) / len(v1_results)
    
    # Test v2 prompt
    v2_results = [
        test_prompt_accuracy(RESUME_EXTRACTOR_V2, case)
        for case in test_set
    ]
    v2_accuracy = sum(v2_results) / len(v2_results)
    
    print(f"v1: {v1_accuracy:.2%}")  # 85%
    print(f"v2: {v2_accuracy:.2%}")  # 88%
    
    if v2_accuracy > v1_accuracy + 0.02:  # >2% improvement
        deploy_prompt(RESUME_EXTRACTOR_V2)
```

---

## Interview Code Questions & Answers

### Q: "Implement the `classify_document()` method"

**Your Answer**:
```python
def classify_document(self, text: str) -> str:
    """Classify as 'resume', 'job_description', or 'unknown'"""
    text_lower = text.lower()
    
    # Count keyword occurrences
    resume_score = sum(1 for keyword in self.RESUME_KEYWORDS 
                      if keyword in text_lower)
    jd_score = sum(1 for keyword in self.JD_KEYWORDS 
                  if keyword in text_lower)
    
    # Classification logic
    if resume_score > jd_score and resume_score >= 2:
        return "resume"
    elif jd_score > resume_score and jd_score >= 2:
        return "job_description"
    else:
        return "unknown"
```

### Q: "How would you implement skill matching?"

**Your Answer**:
```python
def calculate_skill_match(candidate_skills: List[str], 
                         jd_requirements: List[Tuple[str, str]]) -> str:
    """
    Match candidate skills against JD requirements.
    jd_requirements: [(skill, requirement)] where requirement is "Must Have" or "Nice to Have"
    """
    
    must_have_matches = 0
    must_have_total = 0
    nice_to_have_matches = 0
    nice_to_have_total = 0
    
    for skill, requirement in jd_requirements:
        if requirement == "Must Have":
            must_have_total += 1
            if skill in candidate_skills:
                must_have_matches += 1
        else:
            nice_to_have_total += 1
            if skill in candidate_skills:
                nice_to_have_matches += 1
    
    # Score: Must-haves are critical
    must_have_ratio = must_have_matches / must_have_total if must_have_total > 0 else 1.0
    nice_to_have_ratio = nice_to_have_matches / nice_to_have_total if nice_to_have_total > 0 else 0
    
    if must_have_ratio == 1.0 and nice_to_have_ratio > 0.5:
        return "Excellent"
    elif must_have_ratio >= 0.8:
        return "Good"
    elif must_have_ratio >= 0.5:
        return "Fair"
    else:
        return "Poor"
```

---

## Troubleshooting Common Issues

### Issue: "LLM returns invalid JSON"

**Solution**: Use Pydantic validation
```python
try:
    result = CandidateResume(**llm_json_response)
except ValidationError:
    # Retry with stricter prompt
    # Or fallback to partial extraction
    result = extract_fallback(llm_json_response)
```

### Issue: "Resume extraction fails for some fields"

**Solution**: Graceful degradation
```python
resume = CandidateResume(
    candidate_profile=extracted_profile,  # Must succeed
    work_experience=extracted_experience or [],  # Use empty list if fails
    skills=extracted_skills or SkillsData(...)  # Fallback default
)
```

### Issue: "Scanned PDF extraction fails"

**Solution**: Not yet implemented
- Current: Document extraction catches error, returns error message
- Future: Integrate OCR (Google Vision API)
- Workaround: User converts PDF to text externally

---

**End of Code Walkthrough**

Use this document to:
- Explain how data flows through the system
- Show specific code examples during interview
- Point to line numbers when discussing implementation
- Walk through a complete end-to-end example
