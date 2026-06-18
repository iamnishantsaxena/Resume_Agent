# JD-Match AI: Quick Reference Summary

## One-Minute Pitch

JD-Match AI is a **Python multi-agent system** that automates resume screening. It extracts structured data from resumes and job descriptions using Google Gemini, performs intelligent matching across 5 dimensions (skills, experience, qualifications, location, work rights), and outputs quantified compatibility scores. Built for **HR teams**, **job platforms**, and **recruiting tools**.

**Tech**: Python + Google Agent Development Kit (ADK) + Gemini models + Pydantic v2

---

## Architecture at a Glance

```
User Input (Resume + JD)
        ↓
Root Agent (Orchestrator)
        ↓
    ┌───┴────┬────────┐
    ↓        ↓        ↓
Resume   JD      Matcher
Agent    Agent    Agent
    ↓        ↓        ↓
Match Result + Scores
```

**4 Agents Total**:
1. **Root Agent** (Gemini 1.5 Pro) - Orchestration & routing
2. **Resume Extractor** (Gemini 2.0 Flash) - Parse resumes → CandidateResume
3. **JD Extractor** (Gemini 2.0 Flash) - Parse JDs → JobDescription
4. **Matcher** (Gemini 2.0 Flash) - Compare & score → MatchResult

---

## Key Design Decisions

| Decision | Choice | Why |
|---|---|---|
| **Model** | Gemini 2.0 Flash (extraction) | 10x cheaper than GPT-4, fast enough for extraction |
| **Data Structure** | Pydantic v2 models | Type safety + validation + schema compliance |
| **Document Parsing** | Abstracted layer (PDF/DOCX/TXT) | Format independence, easy to extend |
| **Error Handling** | Graceful degradation | Real resumes are messy, don't fail on missing fields |
| **Architecture** | Multi-agent | Modularity, reusability, error isolation |

---

## 5-Dimensional Matching Formula

```
Overall Score = (Skills × 0.40) + (Experience × 0.25) + 
                (Qualifications × 0.20) + (Location × 0.10) + 
                (Work Rights × 0.05)
```

**Output**: "Excellent/Good/Fair/Poor Match" + strengths/weaknesses

---

## Data Models (Pydantic)

**CandidateResume**:
- Profile (name, email, phone, LinkedIn, location)
- Work Experience (title, company, dates, responsibilities)
- Skills (technical + non-technical, with validation & evidence)
- Education, Projects, Certifications, Languages

**JobDescription**:
- Title, responsibilities
- Skills (technical + non-technical, categorized as "Must Have" / "Nice to Have")
- Requirements (qualifications, years exp, work rights)
- Flexible arrangements, perks

**MatchResult**:
- Overall rating
- Dimension-by-dimension scores
- Key strengths & weaknesses

---

## Performance

| Metric | Current |
|---|---|
| **Latency (Single Match)** | 6-11 seconds |
| **Throughput** | 350-600 matches/hour |
| **Cost Per Match** | ~$0.001-0.002 |
| **Accuracy** | 85-90% (extraction), 80% (matching) |

---

## Common Interview Questions Quick Answers

| Q | A |
|---|---|
| **Why multi-agent?** | Separation of concerns, reusability, error isolation, independent optimization |
| **Why Gemini not GPT?** | 10x cheaper, fast enough, structured output support; trade-off is slightly lower accuracy on edge cases |
| **Why Pydantic?** | Type safety, validation, serialization, schema as docs; prevents LLM JSON errors |
| **How validate skills?** | Check work history/projects for evidence; flag unvalidated skills |
| **How scale to 1M/day?** | Phase 1: API + caching; Phase 2: Worker pool; Phase 3: Batch API; Phase 4: ML routing |
| **How handle scanned PDFs?** | Current: Fails; Future: Integrate OCR (Google Vision API) |
| **How handle title equivalency?** | Current: Exact match; Future: Build skill taxonomy or embedding-based similarity |
| **How ensure data quality?** | Schema validation + evidence requirements + sanity checks + prompt engineering |
| **What are limitations?** | No OCR, sequential processing, no caching, single-language, no skill synonyms |
| **How test LLM system?** | Integration tests + golden datasets + edge case coverage + A/B prompt testing + metrics tracking |

---

## Scalability Roadmap

| Version | Architecture | Latency | Throughput | Cost/Month |
|---|---|---|---|---|
| **v1** | Single threaded, no cache | 8s | 600/hr | $100 |
| **v2** | Parallel extraction + cache | 6s | 1200/hr | $2K |
| **v3** | Worker pool + Redis | 10s avg | 5000/hr | $5K |
| **v4** | Batch API + DB | 30s-24h | 50K+/hr | $10K |

---

## Security Best Practices

✓ API keys in environment variables  
✓ `.env` file in `.gitignore`  
✓ Use secrets manager in production  
✓ Input validation & sanitization  
✓ Rate limiting on endpoints  
✓ Audit logging  
✓ Regular dependency updates  

---

## What's Missing (Known Limitations)

1. **OCR support** - Can't parse scanned/image PDFs
2. **Skill synonyms** - Can't recognize "Cloud Engineer" = "DevOps Engineer"
3. **Batch processing** - Currently sequential, not parallel
4. **Caching** - Every request re-extracts identical data
5. **Confidence scores** - No field-level confidence tracking
6. **Multi-language** - English only
7. **Learning loop** - Doesn't improve from hiring outcomes
8. **Streaming** - No real-time updates

---

## Production Deployment Checklist

- [ ] API key in environment variable
- [ ] `.env` file created and `.gitignore`'d
- [ ] Run on HTTPS (TLS encryption)
- [ ] Input validation (file size, encoding)
- [ ] Rate limiting implemented
- [ ] Logging/monitoring configured
- [ ] Error handling tested
- [ ] Load testing completed
- [ ] Cache layer added
- [ ] Secrets manager integration
- [ ] Rollback plan documented
- [ ] Monitored for API quota exhaustion

---

## Interview Storyline

**Opening**: "JD-Match AI is a multi-agent system I built to automate resume screening..."

**Architecture**: [Draw diagram]

**Design Decisions**: "We chose Gemini over GPT-4 because cost is critical at scale. We use Pydantic for type safety and validation..."

**Implementation**: "Each agent is responsible for one task. Resume Extractor parses resumes into structured objects. Matcher compares them across 5 dimensions..."

**Scalability**: "Version 1 handles ~600 matches/hour. To scale to 1M/day, we'd add caching, parallel processing, worker pools, and eventually batch API..."

**Limitations**: "Current system can't handle scanned PDFs or skill synonyms, but we have a roadmap..."

**Learning**: "If recruiter disagrees with a match, we decompose the score, identify which agent was wrong, and iterate on the prompt..."

---

## Files to Reference During Interview

- **ARCHITECTURE.md** - Deep technical decisions and trade-offs
- **README.md** - Project overview and usage
- **INSTALLATION.md** - Setup process
- **dtypes_common.py** - Data models (what data we extract)
- **agent.py** - How agents are orchestrated
- **document_extractor.py** - How we handle multiple file formats
- **prompts.py** - How we instruct the LLMs

---

## Talking Points by Interview Type

**For System Design Round**:
- Scalability architecture (v1 → v4)
- Trade-offs (cost vs. accuracy, latency vs. throughput)
- Multi-agent design philosophy
- Caching strategy
- Database schema for scale

**For Coding Round**:
- Implement `DocumentExtractor.classify_document()`
- Implement `MatchScore` calculation
- Implement data models with Pydantic
- Write robust error handling

**For Product/Behavioral Round**:
- HR team problem (manual screening takes 6+ hours/role)
- Your solution (AI automation)
- Impact (40 hours saved/month)
- Trade-offs made (cost vs. accuracy)
- Lessons learned (LLMs are non-deterministic, need validation layer)

---

**Created**: June 18, 2026  
**Updated**: June 18, 2026  
**Status**: Ready for interview prep
