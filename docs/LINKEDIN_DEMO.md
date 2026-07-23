# LinkedIn Demo — AI Talent Copilot

**Title:** AI Recruiting Assistant powered by n8n + Gemini + RAG

---

## Hook (post opening)

Hiring teams still drown in CVs.

I built **AI Talent Copilot** — an n8n + Gemini + Qdrant system that screens multiple candidates against a job description, returns a ranked shortlist with evidence-based scores, and prepares interview questions automatically.

Repo: https://github.com/delpozochristian/ai-automation-portfolio

---

## The problem

- Recruiters spend hours reading CVs with inconsistent criteria
- “Fit” decisions are hard to explain to hiring managers
- Interview prep is ad-hoc and misses role-specific gaps
- Scaling screening means hiring more people — not better process

---

## The architecture

**Index once / retrieve many** — CVs are embedded into `candidates_knowledge` once; screening only retrieves chunks (no PDF re-processing).

```
CVs  →  08 Indexer  →  candidates_knowledge ─┐
JD   →  04 Indexer  →  jobs_knowledge ───────┼→ 07 Screening → ATS JSON ranking
                                             └→ Gemini (explainable scores)
```

| Layer | Tech |
|---|---|
| Orchestration | n8n |
| LLM / embeddings | Google Gemini |
| Vector DB | Qdrant (`candidates_knowledge`, `jobs_knowledge`) |
| Agents / RAG | LangChain nodes in n8n |

**Product modules**

1. Index JD + multi-candidate pool (RAG)
2. Chat over candidate knowledge
3. Single-candidate match analysis
4. **Batch Screening Engine** → ATS JSON (`criteria_scores` + `evidence`)
5. Interview Coach

---

## Demo script (60–90 seconds)

1. **Index:** Run `08` (candidates → `candidates_knowledge`) + `04` (JD → `jobs_knowledge`).
2. **Setup:** Job = Senior Backend Java Engineer (Java, Spring Boot, Kafka, AWS, SQL, leadership).
3. **Screen:** Run `07 - Recruiter Screening Engine` (no PDFs — RAG only).
4. **Output:** ATS JSON with `candidate_id`, `criteria_scores`, `evidence`, `recommendation`, `interview_priority`.
5. **Follow-up:** Open Match AI or Interview Coach for the #1 candidate.

Expected pattern from demo data:

| Rank | Candidate | Priority | Why |
|---|---|---|---|
| 1 | Alex Rivera | HIGH | Deep Kafka + Spring + tech lead |
| 2 | Sam Okonkwo | HIGH | Strong leadership; validate Kafka IC depth |
| 3 | Jordan Lee | LOW | Solid mid-level; gaps on seniority/Kafka |

See `demo/sample_screening_result.json`.

---

## Expected results (what to claim)

- Minutes instead of hours to produce a first shortlist
- Structured, auditable scores (not only a chat paragraph)
- Clear gaps → better interview questions
- Same rubric applied to every candidate (consistency)

---

## Anti-bias note (credibility)

Prompts evaluate **skills, experience, and JD requirements only**.  
No demographic proxies. Missing evidence is labeled — not invented.

Details: `docs/PROMPTS.md`

---

## Suggested LinkedIn carousel / visuals

1. Problem slide — “100 CVs, 1 role, no ranking”
2. Architecture diagram (n8n → Gemini → Qdrant)
3. Screenshot of Screening Engine canvas
4. JSON ranking output
5. Business impact — before/after
6. CTA — link to GitHub repo

---

## Sample caption (copy/paste)

```text
AI Recruiting Assistant powered by n8n + Gemini + RAG

I turned a set of n8n workflows into AI Talent Copilot:
→ index CVs & job descriptions (RAG / Qdrant)
→ match candidates with explainable scores
→ batch-screen multiple CVs into a ranked shortlist
→ generate interview questions from real gaps

Stack: n8n · Google Gemini · Qdrant · LangChain

Built as a portfolio product to show AI Engineering + automation
applied to Talent Acquisition.

Repo in comments 👇
```

---

## Call to action

- Star / fork the repo
- Message for a live walkthrough
- Hiring managers / TA leaders: happy to discuss PoC scope
