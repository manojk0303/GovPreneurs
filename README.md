# GovPreneurs — Auto-Proposal Engine 
**Role:** AI PM Intern Case Study | **By:** Manojkumar K 

**Demo video :** https://drive.google.com/file/d/1yrWjHOsFXkjfSuBul4_U--3mbYd36LK3/view?usp=sharing

**UI design :** https://v0-govpreneurs.vercel.app/

**RAG Workflow :** https://drive.google.com/file/d/1is8SkNIKZY_WkKoJo1cGgijmVO8i3xkT/view?usp=sharing

---

## Part 1 — Data Integration 

**Source:** SAM.gov Assistance Listings API (`api.sam.gov/assistance-listings/v1/search`)
No webhooks exist — we poll on a schedule.

**Schema — fields we actually need:**

| Field | Why |
|---|---|
| `assistanceListingId` | Unique ID — deduplication + change detection |
| `applicant.types[].code` | Eligibility gate — checked before AI runs |
| `deadlines.list[].end` | Response deadline — drives urgency alerts |
| `overview.objective` | Scope of work — fed directly into RAG |
| `applicationProcedure.URL` | RFP PDF link — chunked for AI |
| `publishedDate` | Change detection — if newer than stored → update |
| `financialInformation.rangeAndAverageAssistance` | Contract value — user decides if worth pursuing |

**Keeping data fresh:**
- Poll every 6hrs for new opportunities
- Every 30min if deadline is within 72hrs
- Every 15min if user is actively drafting

**When something changes:**
1. Compare `publishedDate` — if newer, pull the diff
2. Alert the user ("⚠️ Deadline moved to Aug 20")
3. If PDF changed — re-chunk and re-embed so AI isn't working off stale data
4. If cancelled — remove from listings immediately

---

## Part 2 — RAG Workflow + Prompt (The Brain)

**Why RAG?** So the AI writes from real documents, not from memory. No hallucination.


**System Prompt:**

```
You are a government proposal writer for GovPreneurs.

Write each proposal section using ONLY the context provided below.
Do not invent requirements, statistics, or capabilities.

Rules:
- Cite every paragraph with [Source: {section name}]
- If info is missing, write: [PLACEHOLDER: insert {specific detail}]
- If user has no matching experience for a requirement, write:
  [GAP: No direct experience found for "{requirement}"]
- Tone: professional, confident, government-compliant
- Mirror the agency's own language where possible

Goal: A contracting officer should be able to verify every
claim against the cited source.
```

---

## Part 3 — UI Design (Proposal Review Screen)

**3 things the design must solve:**
1. **Trust** — user can see exactly where every sentence came from
2. **Control** — user can refine any section without regenerating everything
3. **Encouragement** — feels like a co-pilot, not a government form

**Screen layout:**
- **Left sidebar** — section list with completion status (✓ done / ⚠ needs review)
- **Center canvas** — AI draft with blue `[RFP §3.2]` citation badges; gaps in amber
- **Right panel** — trust score, compliance checklist, Submit button
- **Top bar** — opportunity name, deadline countdown, match score %


---
