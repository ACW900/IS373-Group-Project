# IS373 Group Record: Common Exam Note Coverage Checker

**Team:** Alex Wiemer ([ACW900](https://github.com/ACW900)) · Dylan H · **Course:** IS373, NJIT, Fall 2026

**Stage:** Review 1 (Plan), Nov 17, 2026. Nothing is built yet, so every check below is labeled `planned`.
**Status labels:** `planned` · `observed locally` · `observed in CI` · `observed hosted`. A plan or an expected answer is not an executed test.

**Origin:** this project builds on our IS247 Group 6 concept, a note-sharing platform for classmates (IS247 Group 6: Christian M, Matthew P, Dylan H, Alex W). IS247 covers the user-experience design; this IS373 project builds a working content system from it.

<details>
<summary><strong>3. Group content system + problem research</strong> (click to open)</summary>

## 3. Group content system + problem research

### Index

| Item | Where | Status |
|---|---|---|
| Project ideas (one each) + choice | [A](#a-project-ideas-one-each) | chosen |
| Reader, problem and question | [B](#b-reader-problem-and-question) | decided |
| Source manifest | [C](#c-source-manifest) | written (synthetic fixtures, labeled) |
| Workflow | [D](#d-workflow) | decided |
| Two approaches compared + choice | [E](#e-two-approaches-compared) | chosen (provisional) |
| Vector, graph or both | [F](#f-vector-graph-or-both) | vector + keyword baseline |
| User stories | [G](#g-user-stories) | written, all planned |
| Evaluation set | [H](#h-evaluation-set) | written before any run |
| Owner / reviewer split | [I](#i-owner--reviewer-split) | proposed split |
| Hosting, cost and limits | [J](#j-hosting-cost-and-limits) | planned |
| Peer check + changed / checked / next | [K](#k-peer-check-and-changed--checked--next) | peer check pending |

---

### A. Project ideas (one each)

| | Idea | From |
|---|---|---|
| **Idea 1** | **Common exam note coverage checker:** compare notes from different sections against a common exam's topic list, and show which topics are covered, by which notes, and which are missing. | Alex |
| **Idea 2** | `[Dylan: your idea in one or two sentences]` | Dylan |

**Chosen: Idea 1.** It builds on the note-sharing problem we already researched in IS247, so it helps both classes, and it fits the IS373 "information isolation" problem (source coverage, traceable comparisons, honest gaps).

---

### B. Reader, problem and question

| | |
|---|---|
| **IS373 problem type** | Information isolation |
| **Reader** | A student in a multi-section course that has a **common exam** |
| **Problem** | Every section takes the same exam, but each professor covers the material a little differently. A student's notes can miss topics another section covered, and those topics still show up on the exam. |
| **Question the system answers** | "Which topics on the exam topic list are missing or thin in my notes, and which notes cover them?" |
| **What the reader gets** | For each exam topic: the note passages that cover it (with source IDs), side by side, plus a list of topics **no** notes cover |
| **Demo course** | **Calculus I (Math 111)** as the example, since it's a large multi-section course with a common exam. The topic list and notes are **synthetic, written by us**, so the demo doesn't depend on anyone else's material. |

**How this differs from our individual RAG apps:** it isn't a chatbot. It compares several sources against a fixed topic list and reports coverage and gaps.

---

### C. Source manifest

**Rule:** we don't import classmates' notes, professors' slides or anyone's private work. Every source below is a **synthetic fixture written by us for the demo, and labeled as synthetic**. Real course material would need permission first.

| ID | What | Author | Version | Permission | What it's for |
|---|---|---|---|---|---|
| T01 | Calculus I common-exam topic list (10 topics) | us, synthetic | v1, Nov 2026 | ours | The fixed list every note set is checked against |
| N01 | "Section A" notes: limits, continuity, derivative definition, power/product/quotient rules, chain rule, implicit differentiation | us, synthetic | v1 | ours | Normal coverage |
| N02 | "Section B" notes: "differentiation shortcuts" (rules in different words), related rates, optimization, L'Hôpital's rule, linear approximation | us, synthetic | v1 | ours | Paraphrased coverage |
| N03 | "Section C" notes: Riemann sums, antiderivatives, Fundamental Theorem of Calculus, u-substitution, optimization, plus one passage about exam logistics | us, synthetic | v1 | ours | Overlap + an off-topic passage |
| N04 | One student's incomplete notes: limits and chain rule only | us, synthetic | v1 | ours | Gap detection |

**T01 topics:** limits · continuity · derivative definition · differentiation rules (power, product, quotient) · chain rule · implicit differentiation · related rates · optimization · Mean Value Theorem · Fundamental Theorem of Calculus.
*The Mean Value Theorem is deliberately left out of every note set, so the system has one true gap to find.*

---

### D. Workflow

```text
permission check (all fixtures are ours and labeled synthetic)
  → import T01 + N01–N04, each with a source ID and version
  → clean: strip formatting, split notes by heading/topic, remove duplicate passages
  → represent: T01 topics as fixed items; note passages as chunks
               (source ID, section, passage ID, text)
  → match: for each topic, find the note passages that cover it
           (keyword baseline + vector similarity)
  → no AI generation at first: show the source passages themselves
  → review: a person can mark a match "doesn't really cover it", which moves the topic toward the gap list
  → deliver: coverage report page (topic → covering passages side by side → gaps)
```

---

### E. Two approaches compared

| | Approach 1: keyword match | Approach 2: meaning match (vector) |
|---|---|---|
| How | Search each topic's words in the notes (SQLite FTS5) | Embed topics and note passages; match by similarity (sqlite-vec) |
| Strength | Simple, fast, easy to explain why something matched | Catches notes that cover a topic in different words (N02's "differentiation shortcuts") |
| Weakness | Misses paraphrases, so gaps look bigger than they are | Can match something related but wrong; a similarity score isn't proof |
| How we'll check it | Run H1–H5 | Run H1–H5 and compare with Approach 1 |

**Choice (provisional): vector matching, with keyword matching kept as the baseline.** H2 (paraphrase) is the case we expect the keyword baseline to miss. **Fallback:** if vector matching isn't working by Build, ship the keyword version with the gap report and label the limit.

---

### F. Vector, graph or both

**Vector, with a keyword baseline.** Our question is "does any note talk about this topic," which is a meaning match. A **graph** (topic → subtopic → passage) could later show how topics connect, for example that the chain rule leads into implicit differentiation, but we don't need typed relationships to answer the main question. We'll revisit it at Improve.

---

### G. User stories

| # | Story | Check | Status |
|---|---|---|---|
| 1 | As a student, I can see which exam topics my notes don't cover. | Checking N04 against T01 lists the 8 topics N04 leaves out. Checking all notes lists only the Mean Value Theorem. | planned |
| 2 | As a student, I can open the exact passage that covers a topic. | Every match shows a source ID + passage ID and opens that passage. | planned |
| 3 | As a student, I can compare notes from different sections side by side for one topic. | "Optimization" shows passages from N02 and N03 next to each other. | planned |
| 4 | As a reviewer, I can mark a match as wrong so it stops counting as coverage. | Mark a match "doesn't really cover it": it no longer counts, and a topic with no other match moves to the gap list. | planned |
| 5 | As the admin, I can re-import notes without creating duplicates or saving bad data. | Import N01 twice: the passage count stays the same. Import an empty file: a clear error, nothing saved. | planned |

---

### H. Evaluation set

Written **before** anything runs. Expected results come from how we wrote the fixtures.

| # | Type | Topic / input | Expected result |
|---|---|---|---|
| H1 | Exact wording | Chain rule | N01, N04 |
| H2 | Paraphrase (different words) | Differentiation rules (power, product, quotient) | N01 and N02 (N02 says "differentiation shortcuts"). The keyword baseline may miss N02. |
| H3 | Covered by one section only | Related rates | N02 only |
| H4 | True gap | Mean Value Theorem | No match, so it's listed as a gap |
| H5 | Off-topic passage | N03's exam-logistics passage | Matches no topic |

**What counts as a pass:** the right source IDs come back, nothing extra counts as coverage, and H4 is reported as a gap instead of a forced match.

---

### I. Owner / reviewer split

Every part has an owner and a **different** reviewer, and we'll link issues and PRs as evidence. *Proposed split, both of us to confirm:*

| Part | Owner | Reviewer | Evidence (issue / PR) |
|---|---|---|---|
| Synthetic fixtures (T01, N01–N04) + import + source IDs | Dylan | Alex | |
| Cleaning, duplicate check, bad-input test | Dylan | Alex | |
| Matching (keyword baseline + vector) | Alex | Dylan | |
| Review step + coverage report page | Alex | Dylan | |
| Hosting + CI (Docker, GitHub Actions, Caddy) | Alex | Dylan | |
| This record + evaluation set + running H1–H5 | both | both | |

---

### J. Hosting, cost and limits

- **Hosting (planned):** Docker container behind Caddy over HTTPS, deployed through a GitHub Actions pipeline like the one Alex already runs (QA → production, health check, rollback).
- **Cost:** no AI generation at first, so the only cost is embeddings for about 50 short passages, which is small and can run on a free tier or a small local model. We'll log real usage instead of guessing.
- **Limits:** import and review controls are admin-only; the public page is read-only. Fixtures are small on purpose. Any instructions written inside a note are treated as text, not commands.

---

### K. Peer check and changed / checked / next

**Peer check** (from a GitHub issue on this repo):
- Peer: `[ ]` · Date: `[ ]` · Issue: `[ ]`
- What they said: `[ ]`
- Our decision / what we changed: `[ ]`

**Changed:** chose the project (Idea 1) and the reader; wrote the source rules and synthetic fixture plan; decided the workflow; compared two approaches and chose vector with a keyword baseline; wrote 5 user stories and a 5-case evaluation set; proposed the owner/reviewer split.
**Checked (actually done):** we reviewed this plan together on Nov 17, 2026. Nothing has been run yet.
**Next:** write the fixtures, confirm the owner/reviewer split as GitHub issues, and build the first import → match → gap report path for **Build, Nov 24**.

**Help received:** Claude (an AI assistant) helped Alex prepare the meeting agenda and drafted this record from our decisions, including the synthetic fixture design and the evaluation cases.

</details>
