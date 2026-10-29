# Acceptance criteria — Provenance Guard

Five criteria that say what "working" means for this service, written in unit 7
**before** you built it.

**All five are yours.** None are given.

An acceptance criterion names a number or something plainly observable. *"The
detector is accurate"* is an opinion — and perfect AI detection isn't possible
anyway, so it's an opinion about something nobody can deliver. *"Of 10 pieces I
wrote myself, at most 1 is labelled high-confidence AI"* is a criterion.

Under each, write a sentence or two on **why that target**. *"I allow at most 1
false positive in 10 because on a writing site a wrong accusation costs more
than a missed detection"* is a real answer.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

**Cover at least three of these five areas.** They're prompts, not a form.

| Area | A question it could answer |
|---|---|
| False positives | How often may a human's writing get called AI? |
| Score spread | How far apart should clearly-AI and clearly-human text score? |
| Label coverage | Is every one of your three labels actually reachable? |
| The appeal path | Does an appeal always change status and always get recorded? |
| Bad input | What happens when `text` is missing, empty, or enormous? |

Two things worth knowing before you pick numbers:

- **The expensive mistake is one-sided.** Missing some AI text costs the site a
  little. Accusing a real writer costs that person a lot. A criterion that
  treats both errors as equally bad is describing a system nobody would ship.
- **Unit 8 tests across three trials, and the target has to hold across all
  three.** Your first signal runs a model, so identical input can score
  differently twice. A target of "at most 1 in 10" against trials of 2, 1, 2 is
  a **miss**.

---
## 1.

<!-- Your criterion. It must name a number. -->

Of 10 pieces of writing I wrote myself, at most 1 is labelled
`high-confidence AI` in each evaluation trial.



**Why this target:**

One false positive in ten is already serious for a real writer, so this keeps
the system cautious even if it lets some AI text through.



---

## 2.

<!-- Your criterion. -->

At least 8 of 10 clearly AI-written samples score at or above `0.65`, and at
least 8 of 10 clearly human-written samples score at or below `0.35`.



**Why this target:**

The two groups should be separated enough to make the high-confidence labels
useful, while leaving the `0.35` to `0.65` band available for uncertainty.



---

## 3.

<!-- Your criterion. -->

All three labels—`high-confidence human`, `unsure`, and `high-confidence AI`
—must each be produced by at least one valid submission in a calibration run.



**Why this target:**

Every label is part of the promised contract. If one is unreachable, readers
cannot receive the decision the interface claims to support.



---

## 4.

<!-- Your criterion. -->

For 5 valid submissions followed by 5 valid appeals, all 5 appeals must return
success, change the matching content to `under_review`, and create a separate
appeal entry in the audit log.



**Why this target:**

An appeal is the safety net for false positives, so it must be observable and
reliable rather than merely accepted with a polite JSON shrug.



---

## 5.

<!-- Your criterion. -->

For each of these malformed requests—missing `text`, empty `text`, non-string
`text`, and text over the configured maximum—the service must return a
documented 4xx response, never a 5xx response, and write a rejection entry
when the request is refused.



**Why this target:**

Bad input is normal at an API boundary. A clear client error protects the
service from crashes and gives callers a traceable explanation of rejection.



---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 8 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Uncertain cases land in the unsure band

         The system handles uncertain cases well.

         **Why this target:** ...

         > **Revised in unit 8:** Text I judged borderline scores between
         > 0.4 and 0.6 in at least 4 of 5 cases.
         >
         > **Why revised:** "well" gave me nothing to check. I couldn't
         > reach a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "At most 1 of 10 of my own pieces flagged as AI" → "at most 3
            of 10", because 1 of 10 turned out to be hard.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
