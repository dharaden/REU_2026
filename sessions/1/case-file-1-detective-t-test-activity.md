# Case File #1: The Snickerdoodle Dispute

**A transition activity: from the kitchen to the detective's office — your first t-test**

*Format: 70–80 minute guided session | Prerequisite: the "Welcome to the Kitchen" session (panes, Projects, objects, scripts, plots)*

---

## Instructor Overview

**The handoff.** This session introduces a second metaphor without abandoning the first. The kitchen explained the *workflow* — where things live and how to run them. The detective explains the *reasoning* — how we decide whether an observed difference is real evidence or ordinary noise. The framing sentence to use verbatim and return to all semester: **"The kitchen is where you work; the case is why."** Nothing about RStudio changes today; students open the same Project, write the same kind of script. What changes is the question they're asking.

**Goals for today.** Students leave having (1) stepped into the detective role and understood the courtroom logic of inference — presumption of innocence (null), evidence evaluation (p-value), standard of proof (alpha), verdict language (reject / fail to reject); (2) walked a crime scene (descriptives and a plot *before* any test); (3) run and interpreted their first independent-samples t-test; (4) computed an effect size and understood why "guilty" and "a big deal" are different questions.

**Misconceptions to police actively** (these are the whole reason for the detective frame):

- **The prosecutor's fallacy.** The p-value is P(evidence this extreme | innocence), *not* P(innocence | evidence). When a student says "there's a 0.04% chance the null is true," correct it on the spot with the courtroom version: "No — IF the cookies were truly equal, evidence this strong would appear in fewer than 4 in 10,000 investigations. That's a statement about the evidence, not about guilt."
- **"Accepting" the null.** A not-guilty verdict is not a finding of innocence — it means insufficient evidence. We never *accept* the null; we *fail to reject* it. Drill the verdict language today while the stakes are low.
- **Significant = important.** With a big enough investigation you can convict someone of jaywalking. That's why the detective always asks two questions: *did it happen* (p) and *how big a deal is it* (effect size).

**Tone:** the same playfulness as day one. The "crime" is a kitchen squabble, the data are cookie ratings, and the first verdict should feel like solving something.

---

## Part 1: A Dispute Breaks Out (10 min)

Tell the story before showing any code:

> "Our restaurant has a problem. The pastry chef insists her snickerdoodles outscore the oatmeal cookies — she wants them featured on the menu. The owner says that's nonsense: 'Some nights any cookie rates well. You're seeing patterns in luck.' They've hired an investigator. That's you."

Then make the handoff explicit:

- "For weeks you've been cooks. The kitchen doesn't go away — your Project, your scripts, your data frames all work exactly as before. But today you put on a second hat: when the question is *'is this difference real?'*, you're a detective."
- Write the two competing claims on the board and name them:
  - **The owner's claim — "it's just luck, the cookies are truly equal"** → this is the **null hypothesis**. In court terms: the presumption of innocence. We *start* by assuming nothing is going on.
  - **The pastry chef's claim — "snickerdoodles really are rated higher"** → the **alternative hypothesis**. The accusation that must be proven.
- The key asymmetry, stated plainly: "We don't ask the chef to feel confident. We ask whether the evidence is strong enough to overturn the presumption that nothing is happening. No conviction without evidence."

**Talking point on standards of proof:** "Before we look at a single rating, we set our standard of proof: how surprising does the evidence have to be before we're willing to convict? In psychology the convention is alpha = .05 — if evidence this strong would turn up less than 5% of the time under innocence, we convict. We choose this *before* the trial. A detective who moves the standard after seeing the evidence is a corrupt detective."

---

## Part 2: Walk the Crime Scene (15 min)

Rule of the badge: **no detective accuses anyone before surveying the scene.** Descriptives and a plot come before any test, every time, forever. Students open their Project (the kitchen is still the kitchen), start a new script `case-file-1.R`, and build the evidence:

```r
# ==========================================================
# CASE FILE #1: The Snickerdoodle Dispute
# Detective: [student name]      Date opened: [today]
# Claim under investigation: snickerdoodles out-rate oatmeal
# ==========================================================

# --- The evidence: ratings from 12 tasters per cookie ---
snickerdoodle <- c(8, 9, 7, 8, 9, 10, 7, 8, 9, 8, 7, 9)
oatmeal       <- c(7, 6, 8, 7, 6, 7, 8, 5, 7, 6, 7, 6)

# --- Walk the scene: describe before you accuse ---
mean(snickerdoodle)   # 8.25
mean(oatmeal)         # 6.67
sd(snickerdoodle)     # 0.97
sd(oatmeal)           # 0.89

# --- Photograph the scene ---
boxplot(snickerdoodle, oatmeal,
        names = c("Snickerdoodle", "Oatmeal"),
        col = c("tan", "wheat"),
        main = "Case File #1: The Evidence",
        ylab = "Taster Rating (out of 10)")
```

**Discussion before the test — this is the heart of the session.** Ask the room: "The snickerdoodle mean is higher by about 1.6 points. Case closed?" Let them argue. Surface the owner's defense: *each cookie's ratings bounce around by about a point from taster to taster — couldn't a 1.6-point gap arise from that ordinary bounce?* That tension — observed difference vs. ordinary variability — **is the t-statistic**. Plant the ratio idea verbally: "The t-test asks: how big is the difference between groups, *compared to* the noise within groups? Big signal over small noise = strong evidence."

**Connect the SDs to noise explicitly:** "sd() is night-to-night, taster-to-taster bounce. It's the defense attorney's whole argument."

---

## Part 3: The Interrogation — Your First t-test (20 min)

One line:

```r
# --- The interrogation ---
t.test(snickerdoodle, oatmeal)
```

Project the output and read it line by line as a detective's report:

```
        Welch Two Sample t-test

data:  snickerdoodle and oatmeal
t = 4.18, df = 21.8, p-value = 0.0004
alternative hypothesis: true difference in means is not equal to 0
95 percent confidence interval:
 0.80 2.37
sample estimates:
mean of x mean of y
     8.25      6.67
```

Line-by-line translations to give:

- **t = 4.18** — "the signal-to-noise ratio. The difference between cookies is about four times larger than what ordinary taster-to-taster bounce would produce. The footprint is four times too big to be an accident."
- **p-value = 0.0004** — the careful sentence, stated twice and written on the board: "**IF** the cookies were truly equal, evidence this extreme would turn up in about 4 of every 10,000 investigations." Then immediately the warning: "That is NOT the probability the cookies are equal. Confusing those two is called the prosecutor's fallacy, and it's the most common statistical error in published psychology. You now know better than some journal articles."
- **95% CI: 0.80 to 2.37** — "our best estimate of the *size* of the crime: the true gap is plausibly somewhere between 0.8 and 2.4 rating points. Notice zero is nowhere in that interval — a gap of zero (the owner's claim) is not among the plausible values."
- **"Welch"** — anticipate the question: "R's default is the Welch version, which doesn't assume the two groups have equal variances. It's the safer modern default — your textbook's 'Student's t-test' is the same idea with one extra assumption." (Save the full Gosset story for Part 5.)

**The verdict (drill the language):**

- p = .0004 < our standard of proof (.05) → **we reject the null.** Verdict: the evidence is strong enough to overturn the presumption that the cookies are equal.
- Then the counterfactual, so the language sticks: "Suppose p had been .23. What's the verdict?" Push until someone produces *fail to reject* — and immediately stress what that does NOT mean: "Not guilty ≠ proven innocent. We'd say there's insufficient evidence that the cookies differ — never that we proved they're the same."

**APA-style write-up to model** (have students add it as a comment in their script — the detective's report must be readable by other detectives):

```r
# VERDICT: Snickerdoodles (M = 8.25, SD = 0.97) were rated
# significantly higher than oatmeal cookies (M = 6.67, SD = 0.89),
# Welch's t(21.8) = 4.18, p < .001, 95% CI [0.80, 2.37].
```

---

## Part 4: The Size of the Crime — Effect Size (10 min)

Frame: "We've established *that* something happened. A good detective also reports *how big a deal* it is — because with a large enough investigation, you can convict someone of jaywalking."

```r
# --- How big a deal is it? (Cohen's d) ---
d <- (mean(snickerdoodle) - mean(oatmeal)) /
     sqrt((sd(snickerdoodle)^2 + sd(oatmeal)^2) / 2)
d    # 1.71
```

Talking points: the difference is about 1.7 standard deviations — by Cohen's rough benchmarks (0.2 small / 0.5 medium / 0.8 large) this is a very large effect; cookie preferences are dramatic. Then the contrast that cements the lesson: "Imagine a study of 50,000 people finding a 0.05-point cookie gap, p = .003. Significant? Yes. Worth changing the menu over? The p-value can't answer that — only the effect size can. *Did it happen* and *does it matter* are different questions, and your report always answers both."

---

## Part 5: Case Closed + The Original Detective (10 min)

**The Gosset Easter egg — bridges both metaphors in one story:** "The t-test was invented in a kitchen. William Sealy Gosset was a brewer at Guinness in Dublin in 1908, trying to judge barley and hops quality from small samples — small-batch taste tests, exactly like ours. Guinness considered statistics a trade secret and barred employees from publishing, so Gosset published under the pen name 'Student.' That's why your textbook says *Student's t-test*: the most famous test in statistics was the work of an anonymous brewer who needed to be a detective."

**Preview the case board (the semester arc):**

- **Case File #2 (paired t-test):** "On day one you rated how intimidating R felt, 1–10. At midterm you'll rate it again. Same witnesses interviewed twice — did the suspect (this course) change you, or is the before–after difference just noise?"
- **Case File #3 (ANOVA):** "Five cookie recipes, one lineup. The F-test asks: did *someone in this room* do it? Then post-hoc interrogations find who — and you'll learn why interrogating enough innocent suspects eventually makes one look guilty by chance."
- **Case File #4 (correlation):** "Two variables always seen together. Partners in crime, or is a third party orchestrating both?"

**Exit ticket:**

1. In court terms, what is the null hypothesis? (Looking for: presumption of innocence / "nothing is going on.")
2. Your friend says "p = .0004 means there's a 0.04% chance the cookies are equal." What's wrong with that sentence?
3. Why do we report an effect size alongside a p-value?

---

## Appendix: Anticipated Questions & Snags

| Question / symptom | Response |
|---|---|
| "Why does it say Welch and not Student?" | R's safer default; doesn't assume equal variances. `t.test(x, y, var.equal = TRUE)` reproduces the textbook version — and sets up the Gosset story. |
| "Can we just say the cookies are different because the means are different?" | The owner's defense: means *always* differ a little by chance. The whole job is deciding whether the gap exceeds ordinary noise. |
| "Is p = .05 special?" | A convention, not physics — a community-agreed standard of proof, chosen in advance. Different fields set different standards (particle physics demands far stronger evidence). |
| "So we proved snickerdoodles are better?" | Soften: we found strong evidence against "they're equal" *for these tasters*. Detectives close cases; they stay humble about appeals (replication). |
| Student gets p of exactly 0 in output | R prints very small p-values as `< 2.2e-16`; teach reading scientific notation and reporting as p < .001. |
| "Why can't we accept the null when p is big?" | Absence of evidence ≠ evidence of absence: failing to find fingerprints doesn't prove no one was there — maybe the investigation was too small (underpowered). |
