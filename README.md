# Provenance Guard
XinBao Chen

**Repository URL:** https://github.com/XinBaoChen/ai201-project4-provenance-guard-starter-v2026
> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first.**
>
> Once `python test.py` passes, in one terminal:
>
> ```bash
> python app.py
> ```
>
> and in another:
>
> ```bash
> curl -X POST http://127.0.0.1:5000/ping \
>   -H "Content-Type: application/json" \
>   -d '{"message": "hello"}'
> ```
>
> That's a web service. Something sent it text and it sent something back.
> Everything this unit is filling in the middle.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     Unit 7 asks for the first five sections. Unit 8 adds the five below.

     Everything is pasted as TEXT. A pasted curl response gets full credit;
     a picture of your terminal gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 7 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- The path a submission takes, from arriving to the label a reader sees.
     Six or seven steps — this is the trace you wrote in Milestone 1. -->

1. A writer sends `text` and `creator_id` as JSON to `POST /submit`.
2. The route validates and extracts the submission.
3. `detector.model_signal` measures how predictable the text is to the local
   language model and returns a 0–1 model score.
4. `stylometry.style_signal` measures sentence, vocabulary, and punctuation
   patterns and returns a 0–1 style score.
5. `scoring.combine_signals` combines both scores into one AI-likelihood score.
6. `scoring.score_to_label` maps that score to a guess and reader-facing label.
7. `audit.log_decision` records the scores and decision; `/submit` returns it.


---

## Signals and Scoring

<!-- For each signal: what property it measures, why that might differ between
     human and AI writing, and WHAT IT CAN'T SEE.

     The blind spot is not optional. If you can't name it you don't understand
     the signal yet — and unit 8's attack set is built out of blind spots. -->

### Signal one — the model signal

**What it measures:** How predictable the text is using model perplexity. Lower
perplexity becomes a higher AI-likelihood score.

**Why that might differ between human and AI writing:** AI text often uses
common, highly predictable word sequences, while human writing is often more
surprising and varied.

**What it can't see:** Predictability is not authorship. Plain writing, short
sentences, second-language writing, or familiar vocabulary can look
machine-written; unusual human prose can look human because it is hard to
predict.

### Signal two — the style signal

**What it measures:** Sentence-length variation, type-token ratio (vocabulary
variety), and punctuation density, normalized into a 0–1 AI-likelihood score.

**Why that might differ between human and AI writing:** AI text often has
regular sentence shapes, repeated vocabulary, and consistent punctuation,
while human drafts tend to vary more.

**What it can't see:** It cannot understand meaning, facts, intent, or
authorship. Typos, pasted quotations, formatting changes, and broken-up
sentences can change the score without changing who wrote the text.

### The combining rule

<!-- Write the rule in words, then give the numbers. What happens when the two
     signals disagree? -->

**The rule:** Treat both signals as evidence, not proof. Use a weighted average;
when they disagree, the result moves toward the middle and is more likely to
be labeled unsure.

**The numbers:** `0.50 * model_score + 0.50 * style_score`

**Where it lives:** `scoring.py::combine_signals`

<!-- ⚠️ The grader checks your code against that line. If your rule lives
     somewhere else, say where. -->

**A case where my two signals split, and what my rule does with it:**

For a plain human submission, the model score might be `0.72` while the style
score is `0.28`. The combined score is `0.50`, so the service says unsure
instead of making a confident accusation.

### False-positive trace

A real writer submits plain, carefully edited work. The model returns `0.78`
because the word choices are predictable; the style signal returns `0.66`.
The combined score is `0.72`, so the writer sees a high-confidence AI label.
They can send the `content_id` and an explanation to `POST /appeal`; the
original decision stays in the append-only log and the item becomes
`under_review`.

### Endpoint contracts

#### `POST /submit`

**Request**

```json
{
  "text": "The text to evaluate.",
  "creator_id": "writer-123"
}
```

**Response**

```json
{
  "content_id": "generated-id",
  "guess": "human | unsure | ai",
  "confidence": 0.72,
  "label": "Reader-facing explanation.",
  "model_score": 0.78,
  "style_score": 0.66
}
```

All scores are numbers from `0.0` to `1.0`; `content_id` is needed to appeal.

#### `POST /appeal`

**Request**

```json
{
  "content_id": "generated-id",
  "reasoning": "I wrote and revised this submission myself."
}
```

**Response**

```json
{
  "ok": true,
  "content_id": "generated-id",
  "status": "under_review"
}
```

The endpoint requires an existing content ID, logs the reasoning, and does not
automatically re-score the text.


---

## Label Variants

<!-- The exact text of all three labels, as a reader would see them, and the
     score range each covers.

     Write them for someone who has never heard the word "threshold". -->

| Label | Score range | The exact text a reader sees |
|---|---|---|
| high-confidence human | 0.00 – 0.35 | We think this text was probably written by a person. |
| unsure | above 0.35 and below 0.65 | We cannot tell confidently whether this text was written by a person or generated by AI. |
| high-confidence AI | 0.65 – 1.00 | We think this text was probably generated by AI. |

**Why I worded the "unsure" one this way:**
<!-- It's the hardest of the three. It has to admit uncertainty without
     sounding like an accusation, because the person reading it may well have
     written every word themselves. -->

It says plainly that the system cannot tell, without accusing the writer or
pretending that a score is proof.


---

## Sample Run

<!-- One submission and one appeal, pasted as text: the request, the response,
     and the log entries. -->

**A submission**

<!-- The response needs content_id, guess, confidence, label, AND both
     signal scores - model_score and style_score. Unit 8's tools read those
     two off the response, so leaving them out costs you next unit's
     diagnostics. -->

```bash
$ curl -X POST http://127.0.0.1:5000/submit \
  -H "Content-Type: application/json" \
  -d '{"text": "...", "creator_id": "..."}'
```

```json

{
  "content_id": "8c2c2d7f0a6d4bbf9d5a12b1c8b7e321",
  "guess": "unsure",
  "confidence": 0.4277,
  "label": "We cannot tell confidently whether this text was written by a person or generated by AI.",
  "model_score": 0.3549,
  "style_score": 0.5005
}

```

**An appeal**

```bash
$ curl -X POST http://127.0.0.1:5000/appeal \
  -H "Content-Type: application/json" \
  -d '{"content_id": "...", "reasoning": "..."}'
```

```json

{
  "ok": true,
  "content_id": "8c2c2d7f0a6d4bbf9d5a12b1c8b7e321",
  "status": "under_review"
}

```

**What the log shows afterwards**

```bash
$ curl "http://127.0.0.1:5000/log?limit=3"
```

```json

{
  "count": 2,
  "entries": [
    {
      "timestamp": "2026-10-28T21:00:00+00:00",
      "content_id": "8c2c2d7f0a6d4bbf9d5a12b1c8b7e321",
      "creator_id": "writer-1",
      "guess": "unsure",
      "model_score": 0.3549,
      "style_score": 0.5005,
      "combined_score": 0.4277,
      "label": "We cannot tell confidently whether this text was written by a person or generated by AI.",
      "status": "decided"
    },
    {
      "content_id": "8c2c2d7f0a6d4bbf9d5a12b1c8b7e321",
      "creator_id": "writer-1",
      "event": "appeal",
      "status": "under_review",
      "reasoning": "I wrote and revised this submission myself."
    }
  ]
}

```

### Milestone 4 calibration

The four calibration inputs produced these local style scores:

| Input | Model score | Style score | Combined score |
|---|---:|---:|---:|
| clearly AI | 0.5896 | 0.6071 | 0.5983 |
| clearly human | 0.3210 | 0.3095 | 0.3153 |
| formal human | 0.3238 | 0.3409 | 0.3323 |
| lightly-edited AI | 0.4793 | 0.3566 | 0.4179 |

The signals split most clearly on formal human writing: its style score was
human-leaning, while the model score remained near the middle. The 50/50
average keeps that disagreement in the unsure range instead of escalating one
signal into a confident accusation.

---

## How I Used AI

**Moment 1**

- *What I asked for:* I asked for help tracing the submission path and naming
  the blind spots of the model and style signals.
- *What came back:* The response identified the seven pipeline stages and
  explained that predictability is not authorship, while stylometry cannot
  understand meaning.
- *What I changed:* I checked those claims against `detector.py` and
  `stylometry.py`, then wrote the final path and blind spots in my own words.

**Moment 2**

- *What I asked for:* I asked for a focused check of the `/submit` and
  `/appeal` contracts after implementing the signals.
- *What came back:* The response found that both scores needed to appear in the
  response and audit log, and suggested boundary tests for all three labels.
- *What I changed:* I ran the tests, kept the 50/50 rule, rewrote the labels to
  avoid accusing writers, and verified the appeal changes status to
  `under_review`.

<!-- ═══════════════════════ UNIT 8 — THE TEST ═══════════════════════ -->

---

## Rate Limiting

**My limits:** ___ per minute, ___ per day

**Why those numbers and not others:**
<!-- Think about two people: a real writer submitting their own work a few
     times an hour, and a script sending a thousand variations to map your
     thresholds. Your numbers have to be liveable for the first and hostile to
     the second. -->

**What a caller counts as:** <!-- per address, or per creator_id? They fail
differently — one script can look like a thousand callers, and one household
can look like one. -->

**The run of status codes when I pushed past it:**

<!-- python run_attacks.py --flood 15 -->

```

```

---

## Attack Run — Before

<!-- Every attack with its outcome and a held-or-broke mark.
     `run_attacks.py --label before` produces the table. -->



**Ten audit log entries from the run**

<!-- Choose entries that show the interesting failures, not the ten easiest. -->

```json

```

**The single worst outcome** — the one I'd least want to explain to a writer
whose work got caught by it:



---

## Run Log and Verdicts

<!-- Your five criteria across three trials. `run_eval.py --label before`. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Real output from one trial**, pasted as text, naming the file and function
that produced it:

```json

```

**Diagnoses**

<!-- For each miss AND each broken attack: the stage and the mechanism.

     Your service has five stages: the route, signal one, signal two, the
     combining rule, and the label mapping.

     Not a diagnosis:  "The evasion attacks got through."
     A diagnosis:      "Four evasion inputs got through. All four had typos
                        added to AI-generated text. My second signal measures
                        word variety, and typos raise word variety — so it read
                        them as more human. My combining rule weights that
                        signal at 0.6, so it dragged the whole score down."

     Look for a pattern. Five failures on one signal is one problem, not five. -->



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Attack Run — After

<!-- Same format. `run_attacks.py --label after` -->



**Did it help, and how do I know:**

<!-- Widening the unsure band usually fixes one problem and creates another.
     Reporting that trade honestly is worth full credit. -->



---

## What's Still Broken

<!-- For each attack still getting through and each criterion still missed:
     what you'd do, and why you stopped. -->



**The trade I'd make if this ran for real:**

<!-- Two sentences. Fewer wrong accusations means more AI text getting through.
     Say which way you'd go and WHO PAYS FOR IT. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 7

       [ ] criteria.md has five numbered criteria, each naming a target
       [ ] Each has a reason underneath
       [ ] POST /submit returns content_id, guess, confidence, label,
           model_score and style_score
       [ ] Two signals that measure DIFFERENT things, each with a named blind spot
       [ ] Three label variants, all reachable, written out in full
       [ ] POST /appeal changes status and writes to the log
       [ ] Signals and Scoring names scoring.py::combine_signals
       [ ] Sample Run: a submission AND an appeal, as pasted text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 8

       [ ] Rate limiting on /submit, with your numbers justified
       [ ] The pasted run of status codes showing 429
       [ ] Audit log with all nine fields, recording rejections too
       [ ] The attack set run IN FULL, every input with an outcome
       [ ] Ten audit log entries pasted as text
       [ ] Five criteria across three trials
       [ ] A diagnosis for every miss and every broken attack — stage AND mechanism
       [ ] One improvement, with Attack Run — After
       [ ] What's Still Broken, including the production trade
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
