# Technical Walkthrough: JD-Match AI Architecture

## Overview

JD-Match AI is a multi-agent orchestration system designed to solve the recruitment matching problem through specialized AI agents and structured data pipelines. 

This document covers the engineering decisions, architecture patterns, and technical trade-offs that shape the system.

## Core Architecture: Multi-Agent Orchestration

### Why Multi-Agent?

The system uses a **multi-agent architecture** rather than a monolithic pipeline for a critical reason: **separation of concerns with specialized expertise**.

Each agent handles a distinct problem:
- **Resume Extractor**: Normalizes diverse resume formats into structured candidate profiles
- **JD Extractor**: Standardizes job descriptions into queryable requirement sets
- **Matcher**: Analyzes compatibility using normalized data from both agents

This design choice enables:
1. **Modularity**: Each agent can be tuned independently without affecting others
2. **Reusability**: Extractors can be used standalone for indexing or analytics
3. **Parallel Processing**: Agents can run independently (though currently sequential)
4. **Error Isolation**: Failures in one agent don't cascade

### Why Google Agent Development Kit (ADK)?

**ADK provides:**
- Structured agent definitions with clear inputs/outputs
- Built-in tool calling mechanism for document extraction
- Integrations with Google's Gemini models
- Standard reasoning loop without custom orchestration code

**Trade-off made**: Vendor lock-in to Google Cloud. We chose this to avoid building custom agentic infrastructure. For complete portability, this could be refactored to use LangChain, but ADK provides better type safety and less boilerplate.

## Data Flow Architecture

```
Resume File (PDF/DOCX/TXT)
        ↓
[Document Extractor] → Raw Text
        ↓
[Resume Extractor Agent] → CandidateResume (Pydantic model)
        ↓
    [Matcher Agent] ← JobDescription (Pydantic model)
        ↓
[JD Extractor Agent] ← Job Description File
        ↓
[Document Extractor] → Raw Text
```

### Why This Sequence?

1. **Document extraction is separate**: Allows reuse for other tasks (indexing, validation)
2. **Agents work on normalized text**: LLMs are more reliable with clean input
3. **Parallel extraction possible**: Resume and JD extraction could run concurrently
4. **Matcher is stateless**: Can be horizontally scaled with batch processing

## Data Modeling: Pydantic v2 Strategy

### Why Pydantic?

```python
CandidateResume {
    profile: {...},
    workExperience: [{...}],
    skills: {technical_skills: [...], non_technical_skills: [...]}
}
```

**Engineering benefits:**
- **Type validation**: Catches malformed LLM output automatically
- **Schema documentation**: Models serve as single source of truth
- **Serialization**: Seamless JSON conversion for APIs/storage
- **IDE support**: Full autocomplete and type checking

**Alternative considered**: Raw JSON. Rejected because LLMs frequently produce invalid JSON or include extra fields that cause downstream failures. Pydantic acts as a validation gate.

### Field Design Philosophy

Fields are deliberately **flat and optional** where possible:
- Easier for LLMs to populate (less nesting = fewer errors)
- Optional fields allow partial extraction (resume missing some data)
- Evidence fields support explainability ("why did we rate this skill?")

```python
# Good: LLM-friendly
skills: [{name: "Python", level: "Advanced", years_experience: 5}]

# Bad: Too nested, LLMs struggle
skills: {categories: {languages: {Python: {level: ...}}}}
```

## LLM Model Selection: Gemini 2.0 Flash

### Why Not GPT-4?

**Gemini 2.0 Flash chosen for:**
1. **Cost**: 10x cheaper than GPT-4 for this task
2. **Speed**: Faster for structured extraction (not creative generation)
3. **Context window**: 1M tokens - handles large documents
4. **Structured output**: Native support for schema-constrained outputs

**Trade-off**: Slightly less capable for nuanced analysis, but extraction tasks don't need frontier reasoning.

**Fallback strategy**: Code uses model as parameter, allowing easy A/B testing with GPT-4/Claude if needed.

## Document Processing: Format Abstraction

```python
DocumentExtractorTool
├── PDF Parser (PyPDF2)
├── DOCX Parser (python-docx)
├── TXT Parser (native)
└── Markdown Parser (native)
```

### Why Abstraction Layer?

Each format has different parsing quirks:
- **PDFs**: Require page-by-page extraction, handle corrupted encodings
- **DOCX**: Preserve formatting metadata, handle embedded objects
- **Text**: Already structured, minimal processing

**Abstraction benefits:**
1. Agents don't need format-specific logic
2. Easy to add new formats (RTF, HTML)
3. Consistent error handling
4. Preprocessing applied uniformly (whitespace normalization, encoding fixes)

## Error Handling Strategy

### Graceful Degradation

The system follows a **best-effort model**:

```python
# If skill extraction fails, continue with what we have
resume = CandidateResume(
    profile=extracted_profile,  # Must succeed
    workExperience=extracted_experience,  # Best effort
    skills=extracted_skills or {technical_skills: [], ...}  # Fallback
)
```

**Why not strict validation?** Real-world resumes are messy. A missing phone number shouldn't fail the entire pipeline. Downstream systems flag confidence scores, not hard failures.

## Scalability Considerations

### Current Limitations

1. **Sequential processing**: Resume and JD extraction run one after another
2. **No caching**: Each request re-extracts the same resume
3. **No async I/O**: Blocks on LLM calls
4. **Single-threaded**: Handles one document pair at a time

### Future Scaling Path

```
v1 (Current):
Request → Extract Resume → Extract JD → Match → Response

v2 (Async):
Request → [Extract Resume] || [Extract JD] → Match → Response

v3 (Cached):
Request → Check Cache[Resume] → Use Cached Profile or Extract
         → Check Cache[JD] → Use Cached Requirements or Extract
         → Match → Response

v4 (Distributed):
Load Balancer → Agent Worker Pool → Match Service → Result Cache
```

## Prompt Engineering: The Hidden Architecture

### Why Centralized Prompts?

```python
# ResumeAgent/sub/prompts.py
resume_extractor_prompt = """
You are a resume parsing expert...
Extract structured information...
Return valid JSON matching this schema...
"""
```

**Design decision**: Single source of truth for agent instructions.

**Benefits:**
1. Easy A/B testing (prompt_v1 vs prompt_v2)
2. Version control for prompt changes
3. Reproducibility across deployments
4. No magic strings scattered in code

### Prompt Structure

Each prompt includes:
1. **Role**: "You are X expert"
2. **Task**: Specific extraction goal
3. **Format**: "Return JSON with schema..."
4. **Examples**: 2-3 real examples
5. **Edge cases**: "If X is missing, do Y"

This structure is crucial because LLMs are instruction-sensitive.


## Testing Strategy (Future)

### Recommended Approach

```python
# tests/test_resume_extraction.py

def test_resume_extraction_with_real_doc():
    """Integration test with real resume"""
    result = resume_agent.run(input=SAMPLE_RESUME)
    assert result.profile.name is not None
    assert len(result.skills.technical_skills) > 0

def test_job_description_schema():
    """Validate JD schema compliance"""
    jd = JobDescription(**SAMPLE_JD_DATA)
    assert jd.jobTitle is not None
```

### Why Integration > Unit Tests

For LLM-based systems, unit tests are less valuable because:
- Agent outputs are non-deterministic
- Mocking LLM responses defeats the purpose
- Integration tests catch real failures

## Known Trade-offs

| Design Choice | Benefit | Cost |
|---|---|---|
| Multi-agent | Modularity, reusability | Complexity, latency (sequential) |
| Pydantic models | Type safety, validation | Stricter than raw JSON |
| Gemini 2.0 Flash | Cost, speed | Less capable for edge cases |
| Document abstraction | Format flexibility | Maintenance burden |
| Graceful degradation | Resilience | Possible silent failures |
| Prompt engineering | Control, tuning | Not reproducible across models |

## Security Architecture

### API Key Management

```python
# ✓ Good
api_key = os.getenv("GOOGLE_API_KEY")

# ✗ Bad
api_key = "sk-..."  # Never
```

### Data Privacy

- Resumes/JDs sent to Google API
- No local storage of sensitive data
- Recommend TLS for production deployments

**Future enhancement**: Support for self-hosted LLMs (Ollama, vLLM) for on-prem deployments.

## Deployment Recommendations

### Development
```bash
# Single machine, .env file
python -m ResumeAgent.agent
```

### Production
```
Recommended:
- Docker container
- Environment variables via secrets manager
- Redis cache for repeated resumes
- Load balancer for multiple replicas
- Monitoring for API quota exhaustion
```

## Summary: Why This Architecture?

| Aspect | Decision | Why |
|---|---|---|
| Architecture | Multi-agent | Separation of concerns |
| LLM | Gemini 2.0 Flash | Cost-effective extraction |
| Data | Pydantic models | Validation + type safety |
| Extraction | Abstracted layer | Format flexibility |
| Configuration | Environment variables | Security |
| Scaling | Sequential (v1) | MVP-appropriate |

The architecture prioritizes **correctness and maintainability** over premature optimization. Each layer is designed to be replaceable as requirements evolve.

## Further Reading

- [Google ADK Documentation](https://ai.google.dev/agent-development)
- [Pydantic v2 Guide](https://docs.pydantic.dev/)
- [Prompt Engineering Best Practices](https://platform.openai.com/docs/guides/prompt-engineering)
