# The Unofficial Guide

Minh Thien Pham — corpus: `city_guides`

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

This project builds a retrieval-augmented generation (RAG) system over the
`city_guides` corpus. It answers factual questions about transportation, food,
opening hours, walking, and other practical information found in the guides.
The system retrieves relevant document chunks, checks them with a relevance
gate, and only sends supported questions to the language model. Generated
answers are grounded in the retrieved documents and name their source files.

## Chunking Strategy

**Chunk size:** 750 characters
**Overlap:** 0 characters

The `city_guides` corpus is organized into labelled Markdown sections such as
"Getting there," "Getting around," "Eat and drink," and "When to go." With the
starter chunker, the corpus produced 51 chunks with an average size of 650
characters, including a chunk only 24 characters long. Several chunks were cut
in the middle of sentences, words, or even section headings.

I changed the chunker to split on `##` section boundaries first and then pack
neighboring complete sections together when they fit within 750 characters.
I use no overlap because the new chunker uses meaningful section boundaries
instead of arbitrary character positions. After the change, the corpus produces
53 chunks averaging 544 characters, with the shortest at 174 characters and the
longest at 741 characters.

## Sample Chunks

<!-- Five chunks, pasted as text. Label each one and name the file it came from
     AND the function that produced it — the grader checks your code against
     what you claim here.

     `python app.py chunks -n 5` prints all three for you. Copy them straight
     across.

     Milestone 3. -->

**Chunk 1** — source: `guide_accessibility.md#0` — produced by: `chunker.py::split_documents`

```
# Getting around the region with limited mobility

An honest assessment rather than a promotional one. Some of these places are
difficult and it is better to know in advance.
```

**Chunk 2** — source: `guide_corry_vale.md#1` — produced by: `chunker.py::split_documents`

```
## Getting around

Nothing within the valley is walkable from anything else — the villages are two to four miles apart. There is one taxi, based in the largest village, and it must be booked a day ahead. Most visitors drive between villages and walk the footpaths in between.

## Eat and drink

One pub in the largest village serves food seven days a week. A second, in the third village, opens Thursday to Sunday. There is a farm shop at the valley mouth that sells bread, cheese and little else, and it closes at 4pm. Bring supplies; this is not a place with options.
```

**Chunk 3** — source: `guide_givens_mill.md#0` — produced by: `chunker.py::split_documents`

```
# Givens Mill

Givens Mill is a village of 700 built around a working watermill that still grinds flour commercially. It is the sort of place people visit for an afternoon and then talk about for longer than the visit lasted.

## Getting there

No station and no bus on Sundays; four buses a day from Brightwater on weekdays, taking 30 minutes. Driving is 20 minutes. The village car park holds about forty cars and is full by 11am on summer Saturdays.

## Getting around

Everything is on one street along the river. The mill is at one end and the church at the other, eight minutes apart. Theriverside path continues in both directions for as far as you want to walk.
```

**Chunk 4** — source: `guide_kestrelford.md#3` — produced by: `chunker.py::split_documents`

```
## When to go

Late spring and early autumn. The Saturday market runs year-round but is much reduced from November to February. August is busy with walkers. The single-track approach road is genuinely difficult in snow and the town can be cut off for a day or two most winters.

## Practical notes

Cash is still useful at the market and in smaller places, though cards are
accepted almost everywhere now. Mobile coverage is good in the centre and
patchy on the outskirts. The nearest full hospital is in Brightwater; there is
a minor injuries unit locally with limited hours.
```

**Chunk 5** — source: `guide_regional_transport.md#1` — produced by: `chunker.py::split_documents`

```
## Buses

Three operators run in the region and they do not accept each other's tickets,
which is the single most common source of confusion for visitors. Services
concentrate on weekday daytimes. Sunday service is minimal to non-existent
outside the Brightwater town routes.

The Kestrelford service is hourly on weekdays, two-hourly on Saturdays, and
does not run on Sundays. The Halden Bay coast service runs four times daily
year-round.
```

## Sample Answer

<!-- One complete question and answer, pasted as text, with the source line
     visible. Milestone 4. -->

**Question:**
How often do buses run from Brightwater to Kestrelford on Saturdays?

**Answer:**

```
Based on the documents, buses run from Brightwater to Kestrelford every two hours on Saturdays (*guide_regional_transport.md* and *guide_kestrelford.md*).

Sources retrieved: guide_brightwater.md, guide_kestrelford.md, guide_regional_transport.md, guide_walking.md
```

**My relevance cutoff:** `0.62`

The highest best-distance among my five in-corpus questions was `0.4171`, while
the lowest best-distance among the five out-of-scope questions was `0.8284`.
That left a clear gap of `0.4113`. I chose `0.62`, which is close to the
midpoint of that gap, so the gate has room on both sides between supported and
unsupported questions.

| Question | In corpus? | Best distance |
|---|---|---|
| How often do buses run from Brightwater to Kestrelford on Saturdays? | Yes | 0.2175 |
| What time does Kestrelford's bakery usually sell out? | Yes | 0.4171 |
| Where can visitors find cheaper food than Brightwater's riverside strip? | Yes | 0.3605 |
| What time do Brightwater's local buses stop running? | Yes | 0.2980 |
| How many train services run between Brightwater and the regional hub on weekdays? | Yes | 0.2924 |
| What is the capital of Mongolia? | No | 0.8284 |
| How do I change the oil in a diesel engine? | No | 0.9124 |
| Who won the 1994 World Cup? | No | 0.9999 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.8459 |
| How do I write a for loop in Rust? | No | 0.8434 |

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1.** I asked Claude to help me inspect the starter chunking behavior and
compare it with a strategy that better matched the structure of `city_guides`.
It suggested using the Markdown `##` headings as semantic boundaries. I then
checked the actual baseline chunks and changed the implementation to split on
section boundaries and pack neighboring complete sections up to 750 characters.

**2.** I asked Claude to help me interpret the retrieval distances from my five
in-corpus questions and five out-of-scope questions. It identified the highest
in-corpus distance as 0.4171 and the lowest out-of-scope distance as 0.8284.
I used those measured results to choose a relevance cutoff of 0.62 and then
verified it by testing both a supported question and an unsupported Rust
question.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 4/5 | 4/5 | 4/5 | MET |
| 2. Every answer names a source | 5 of 5 | 4/5 | 5/5 | 5/5 | MISSED |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Sampled chunks contain no cut-off sentence | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 5. Named source contains the supporting fact | 5 of 5 | 4/5 | 4/5 | 4/5 | MISSED |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

Real Output

Produced by `run_eval.py::main`, using retrieval from `store.py::search` and chunks from `chunker.py::split_documents`.

**Criterion 1**

For the Brightwater bus question, retrieval returned `guide_brightwater.md#0` instead of the chunk containing the `7pm` answer:

```
1   0.2980   guide_regional_transport.md
2   0.3240   guide_brightwater.md   # Brightwater...
```

**Criterion 2**

Brightwater bus question, Run 1:

```
Based on the provided documents, there is no mention of what time Brightwater's local buses stop running. Therefore, I do not have enough information to answer the question.
```

**Criterion 3**

```
gate refused 5 of 5
```

**Criterion 4**

Sample output from `chunker.py::split_documents`:

```
## Getting around

Nothing within the valley is walkable from anything else — the villages are two to four miles apart. There is one taxi, based in the largest village, and it must be booked a day ahead. Most visitors drive between villages and walk the footpaths in between.

## Eat and drink

One pub in the largest village serves food seven days a week. A second, in the third village, opens Thursday to Sunday. There is a farm shop at the valley mouth that sells bread, cheese and little else, and it closes at 4pm. Bring supplies; this is not a place with options.
```

**Criterion 5**

Brightwater bus question, Run 2:

```
I do not have enough information to answer what time Brightwater's local buses stop running (source: guide_brightwater.md, guide_regional_transport.md, guide_kestrelford.md, guide_marchwood.md).
```

The expected answer was `7pm`.

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunks contain the answer | MET | Four of five questions retrieved a chunk containing the expected answer, meeting the target. |
| 2 | Every answer names a source | MISSED | One answer in Run 1 did not name a source, so the 5-of-5 target did not hold across all runs. |
| 3 | Gate stops out-of-corpus questions | MET | The gate refused all five out-of-scope questions. |
| 4 | Sampled chunks contain no cut-off sentence | MET | All five sampled chunks contained complete sentences. |
| 5 | Named source contains the supporting fact | MISSED | The Brightwater bus question failed to return the expected `7pm` answer in all three runs. |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->
### Criterion 2 — Every answer names a source

**Stage:** Generation

Criterion 2 was missed because the first run of the Brightwater local bus question did not name a source document. Runs 2 and 3 did name sources, so source attribution was not consistent across all three runs.

### Criterion 5 — Named source contains the supporting fact

**Stage:** Retrieval

The Brightwater local bus question asks what time the buses stop running. The answer, 7pm, is in `guide_brightwater.md#1`, but semantic retrieval returned `guide_brightwater.md#0` instead. Because the chunk containing the answer was not retrieved, the generation stage did not receive the information needed to answer correctly.

## The Improvement

**What I changed:** I changed the chunking strategy so each Markdown `##` section becomes its own chunk instead of packing neighboring sections together.

**Why I picked it:** The Brightwater bus answer is in the `## Getting around` section, but that section was packed together with `## Eat and drink` and was not retrieved in the top results.

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 4/5 | 4/5 | 4/5 | MET |
| 2. Every answer names a source | 5 of 5 | 4/5 | 4/5 | 4/5 | MISSED |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Sampled chunks contain no cut-off sentence | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 5. Named source contains the supporting fact | 5 of 5 | 4/5 | 4/5 | 4/5 | MISSED |

### Real Output — After

Produced by `run_eval.py::main`, using retrieval from `store.py::search` and chunks from `chunker.py::split_documents`.

**Criterion 1**

Four of five questions retrieved the expected answer.
```
1   0.2562   guide_kestrelford.md
2   0.2816   guide_marchwood.md
3   0.2913   guide_regional_transport.md
4   0.3332   guide_givens_mill.md
5   0.3556   guide_thornby_wells.md
```

**Criterion 2**

Brightwater bus question, Run 1:

```
Based on the provided documents, there is no mention of the time that Brightwater's local buses stop running. Therefore, I do not have enough information to answer your question.
```

**Criterion 3**
```
gate refused 5 of 5
```

**Criterion 4**
Sample output from chunker.py::split_documents:
```
## When to go

Late spring and early autumn. The Saturday market runs year-round but is much reduced from November to February. August is busy with walkers. The single-track approach road is genuinely difficult in snow and the town can be cut off for a day or two most winters.
```

**Criterion 5**
Brightwater bus question, Run 2:
```
I do not have enough information to answer what time Brightwater's local buses stop running.
```

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

No. The Brightwater bus question still did not retrieve the chunk containing `7pm`. Criterion 2 also remained missed because the Brightwater answers did not name a source in any of the three runs.

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

Criterion 2 is still missed because the Brightwater bus answers do not consistently name a source. I would next tighten the generation prompt to require a source even when it cannot answer.

Criterion 5 is still missed because the correct Brightwater chunk containing `7pm` is not retrieved. I would next try a different retrieval strategy, such as hybrid semantic and keyword search.

I stopped after one improvement because this unit asks for one change to be tested and measured.

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->

I would rewrite Criterion 5 to directly measure whether the generated answer contains the expected fact and cites a source that supports it. That would make answer correctness and source support easier to evaluate together.
