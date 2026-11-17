# IS373 Group Record: Common Exam Note Coverage Checker

**Team:** Alex Wiemer ([ACW900](https://github.com/ACW900)) · Dylan H · **Course:** IS373, NJIT, Fall 2026

**Stage:** Review 1 (Plan). Nothing is built yet, so every check below is labeled `planned`.
**Status labels:** `planned` · `observed locally` · `observed in CI` · `observed hosted`. A plan or an expected answer is not an executed test.
**Legend:** **Decided** = agreed by both of us · **Proposed** = one person's suggestion, not yet agreed · `[ ]` = still to fill in.

**Origin:** this project builds on our IS247 Group 6 concept, a note-sharing platform for classmates (IS247 Group 6: Christian M, Matthew P, Dylan H, Alex W). IS247 covers the user-experience design; this IS373 project builds a working content system from it.

<details>
<summary><strong>3. Group content system + problem research</strong> (click to open)</summary>

## 3. Group content system + problem research

### Index

| Item | Where | Status |
|---|---|---|
| Reader, problem and question | [A](#a-reader-problem-and-question) | decided |
| Source manifest | [B](#b-source-manifest) | sources to decide |
| Workflow | [C](#c-workflow) | proposed |
| Two approaches compared + choice | [D](#d-two-approaches-compared) | proposed |
| Vector, graph or both | [E](#e-vector-graph-or-both) | proposed |
| User stories | [F](#f-user-stories) | proposed, all planned |
| Evaluation set | [G](#g-evaluation-set) | to write once the course is chosen |
| Owner / reviewer split | [H](#h-owner--reviewer-split) | to decide |
| Peer check + changed / checked / next | [I](#i-peer-check-and-changed--checked--next) | pending |

---

### A. Reader, problem and question

| | |
|---|---|
| **IS373 problem type** | Information isolation (help a reader find and compare related material, with source coverage and honest gaps). **Decided** |
| **Reader** | A student in a multi-section course that has a **common exam**. **Decided** |
| **Problem** | Every section takes the same exam, but each professor covers the material a little differently. A student's notes can miss topics another section covered, and those topics still show up on the exam. **Decided** |
| **Question the system answers** | "Which topics on the exam topic list are missing or thin in my notes, and which notes cover them?" **Decided** |
| **What the reader gets** | For each exam topic: which notes cover it (with a link to the source passage), shown side by side, plus a list of topics **no** notes cover. |
| **Demo course** | `[ ]` which common-exam course and topic list to use |

**How this differs from our individual RAG apps:** it isn't a chatbot. It compares several sources against a fixed topic list and reports coverage and gaps.

---

### B. Source manifest

**Rule we're following:** we don't import classmates' notes or anyone's private work. We use **our own notes** (each with the other's OK) or **labeled synthetic notes** written for the demo.

| ID | What | Origin / author | Date / version | Permission | Status |
|---|---|---|---|---|---|
| T01 | Exam topic list or syllabus for `[course]` | `[ ]` | `[ ]` | `[ ]` ask the instructor if it isn't public | to find |
| N01 | Notes, section A | `[ ]` (ours or synthetic, labeled) | `[ ]` | `[ ]` | to decide |
| N02 | Notes, section B | `[ ]` | `[ ]` | `[ ]` | to decide |
| N03 | Notes, section C | `[ ]` | `[ ]` | `[ ]` | to decide |
| N04 | One deliberately incomplete set (to test gap detection) | synthetic, labeled | `[ ]` | ours | to write |

---

### C. Workflow

**Proposed (Alex), to confirm with Dylan:**

```text
permission check → import topic list + notes (each with a source ID)
  → clean: strip formatting, split notes by topic/heading, remove duplicate passages
  → represent: topic list as fixed items; note passages as chunks with source ID + section + date
  → retrieve: for each exam topic, find the matching note passages
  → (no AI generation at first: show the source text itself)
  → review: a person marks a match as "covers it" / "doesn't really" before the report is trusted
  → deliver: coverage report page (topic → notes that cover it → gaps)
```

---

### D. Two approaches compared

| | Approach 1: keyword match | Approach 2: meaning match (vector) |
|---|---|---|
| How | Search each topic's words in the notes (SQLite FTS5) | Embed topics and note passages; match by similarity (sqlite-vec) |
| Strength | Simple, fast, easy to explain why something matched | Catches notes that cover a topic in different words ("differentiation" vs "derivative rules") |
| Weakness | Misses paraphrases, so gaps look bigger than they are | Can match something related but wrong; a similarity score isn't proof |
| Check | Topics with known coverage in our test notes | Same, plus paraphrased test cases |

**Proposed choice (Alex), to confirm:** vector matching, with keyword matching kept as the **baseline** to compare against. **Fallback:** if vector setup isn't working by Build, ship keyword matching alone with the gap report.

---

### E. Vector, graph or both

**Proposed: vector (with a keyword baseline).** Our question is "does any note talk about this topic," which is a meaning match. A **graph** (topic → subtopic → note passage) could help later to show how topics connect, but we don't need typed relationships to answer the main question. We'd revisit it at Improve.

---

### F. User stories

All `planned`. Format: story → observable check.

| # | Story | Check | Status |
|---|---|---|---|
| 1 | As a student, I can see which exam topics my notes don't cover. | With the deliberately incomplete note set (N04), the gap list shows exactly the topics we left out. | planned |
| 2 | As a student, I can open the exact passage that covers a topic. | Every match shows a source ID + section and opens the original passage. | planned |
| 3 | As a student, I can compare notes from different sections side by side for one topic. | Picking a topic shows passages from at least 2 note sets next to each other. | planned |
| 4 | As a reviewer, I can mark a match as wrong so it stops counting as coverage. | Mark a match "doesn't really cover it"; the topic moves to the gap list. | planned |
| 5 | As the admin, I can re-import notes without creating duplicates. | Import N01 twice: passage count stays the same. Import an empty file: a clear error, nothing saved. | planned |

---

### G. Evaluation set

Written **before** we run anything, once the demo course is chosen. `[ ]`

| # | Type | Topic / question | Expected result (source IDs) |
|---|---|---|---|
| E1 | Exact wording | `[ ]` | `[ ]` |
| E2 | Paraphrase (notes use different words) | `[ ]` | `[ ]` |
| E3 | Covered by only one section's notes | `[ ]` | `[ ]` |
| E4 | Not covered by any notes (a true gap) | `[ ]` | gap, no match |
| E5 | Off-topic passage that shouldn't match | `[ ]` | no match |

---

### H. Owner / reviewer split

Every part has an owner and a **different** reviewer. We'll link issues and PRs here as evidence.

| Part | Owner | Reviewer | Evidence (issue / PR) |
|---|---|---|---|
| Source manifest + permission + import | `[ ]` | `[ ]` | |
| Cleaning, duplicate check, bad-input test | `[ ]` | `[ ]` | |
| Matching (keyword baseline + vector) | `[ ]` | `[ ]` | |
| Review step + coverage report page | `[ ]` | `[ ]` | |
| Hosting + CI | `[ ]` | `[ ]` | |
| This record + evaluation set | both | both | |

---

### I. Peer check and changed / checked / next

**Peer check** (from a GitHub issue on this repo):
- Peer: `[ ]` · Date: `[ ]` · Issue: `[ ]`
- What they said: `[ ]`
- Our decision / what we changed: `[ ]`

**Changed:** chose the problem (common-exam note coverage, information isolation), the reader and the question; drafted the workflow, approach comparison, user stories and source rules.
**Checked (actually done):** `[ ]`
**Next:** confirm the proposals, pick the demo course and sources, split owner/reviewer, and build the first import → match → gap report path for **Build, Nov 24**.

**Help received:** Claude (an AI assistant) helped Alex prepare the meeting agenda and drafted this record's structure from our decisions. Items marked **Proposed** are suggestions until we both agree.

</details>
