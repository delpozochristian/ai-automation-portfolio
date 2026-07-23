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

```
CVs / JD  →  n8n workflows  →  Gemini (LLM + embeddings)
                    ↓
              Qdrant (RAG)
                    ↓
     Match scores · Ranking · Interview coach
```

| Layer | Tech |
|---|---|
| Orchestration | n8n |
| LLM / embeddings | Google Gemini |
| Vector DB | Qdrant |
| Agents / RAG | LangChain nodes in n8n |

**Product modules**

1. Index CV + JD (RAG)
2. Chat over candidate knowledge
3. Single-candidate match analysis
4. **Batch Screening Engine** → ranked JSON
5. Interview Coach

---

## Demo script (60–90 seconds)

1. **Setup:** Job = Senior Backend Java Engineer (Java, Spring Boot, Kafka, AWS, SQL, leadership).
2. **Input:** 3 fictional CVs (strong / partial / leadership-heavy).
3. **Run:** Workflow `07 - Recruiter Screening Engine`.
4. **Output:** Ranked list with `overall_score`, strengths, gaps, `interview_priority`.
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
