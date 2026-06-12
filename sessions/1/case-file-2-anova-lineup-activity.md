# Case File #2: The Lineup

**Loading real evidence and running your first ANOVA — with a 300-row CSV**

*Format: 75–90 minute guided session | Prerequisites: Case File #1 (t-test, courtroom logic) | Materials: `cookie-lineup.csv` distributed via LMS*

---

## Instructor Overview

**The case.** The restaurant is redesigning its menu and all five cookies are under review. This time the evidence doesn't fit in a `c()` — 300 tasters' ratings arrive as a CSV file. Two new skills ride on one session: **loading external evidence** (the first real payoff of the Project habit) and **the lineup logic of ANOVA** (omnibus test, then corrected interrogations).

**Goals for today.** Students leave having (1) downloaded a data file into their Project and loaded it with `read.csv()` using a bare filename — and understood *why* that worked; (2) inspected a real data frame (chain of custody); (3) understood why running ten t-tests is detective malpractice (alpha inflation, with the 40% number); (4) run and read a one-way ANOVA; (5) followed up with Tukey HSD and interpreted a two-tier pattern; (6) reported eta-squared.

**The built-in plot twist (don't spoil it early).** The omnibus test is overwhelmingly significant — but the post-hoc interrogations reveal that *snickerdoodle and chocolate chip cannot be told apart* (p = .92), despite snickerdoodle's day-one fame. The data resolve into two tiers: {Snickerdoodle, Chocolate Chip, Peanut Butter} indistinguishable at the top, {Sugar, Oatmeal} indistinguishable at the bottom, and every cross-tier comparison significant. Let students discover this in the Tukey table themselves; it's the moment the lineup metaphor pays off ("the F-test said *someone in this room did it* — the interrogations tell us *who*, and the answer is more interesting than 'snickerdoodle wins'").

**Logistics before class.** Post `cookie-lineup.csv` on your LMS. The single most common failure today is a student whose file landed in Downloads while their script runs in the Project — which is not a bug in the lesson, it *is* the lesson. The error message is the teaching moment: "file not found = you're standing in the wrong kitchen, or the evidence never made it to the evidence locker."

---

## Part 1: The Evidence Box Arrives (10 min)

Frame: "In Case #1 we typed twelve numbers by hand. Real investigations don't work that way — evidence arrives in boxes. In our world, the box is a **CSV file**: comma-separated values, the plainest, most universal container for data. Every stats package on earth can open one."

**Metaphor mapping for the day:**

| Concept | Detective / kitchen equivalent |
|---|---|
| The CSV file | The evidence box delivered to the station |
| Saving it into the Project folder | Logging it into the evidence locker — *your kitchen's pantry* |
| `read.csv("cookie-lineup.csv")` | Signing the evidence in: it appears in the Environment pane |
| The bare filename (no `C:/Users/...`) | "It's in the pantry" — works because you walked in the front door (`.Rproj`) |
| `head()`, `str()`, `table()` | Chain of custody: verify nothing is missing or damaged before relying on it |

**Walk it live:**

1. Students download `cookie-lineup.csv` from the LMS and **move it into their Project folder** (show the Files pane refreshing — "the evidence is now in the locker").
2. Open the Project by double-clicking the `.Rproj` key (say it every time; it's still the habit under construction).
3. New script, `case-file-2.R`, then:

```r
# ==========================================================
# CASE FILE #2: The Lineup
# Detective: [name]            Date opened: [today]
# Evidence: cookie-lineup.csv (300 taster ratings, 5 cookies)
# ==========================================================

# --- Sign the evidence into the locker ---
evidence <- read.csv("cookie-lineup.csv")
```

4. **Pause on the bare filename.** "Notice what we did NOT type: no `C:/Users/yourname/Desktop/...`. Because you opened the Project, R is already standing in the right kitchen, so 'cookie-lineup.csv' is enough — like saying 'it's in the pantry.' This is the payoff of every `.Rproj` double-click you've done all semester. Your script will now run, unchanged, on my computer, your group-mate's computer, and the computer you'll have in grad school."
5. Point at the Environment pane: `evidence — 300 obs. of 3 variables`. "Three hundred witnesses just walked into the station."

---

## Part 2: Chain of Custody (12 min)

"Rule of the badge, extended: before you trust evidence, verify it wasn't damaged in transit. Never analyze a file you haven't looked at."

```r
# --- Chain of custody: inspect before you trust ---
head(evidence)          # first six witnesses
str(evidence)           # what kind of variables arrived?
table(evidence$cookie)  # 60 tasters per cookie -- a balanced lineup

# --- Walk the scene: describe before you accuse ---
aggregate(rating ~ cookie, data = evidence, FUN = mean)
aggregate(rating ~ cookie, data = evidence, FUN = sd)

# --- Photograph the scene ---
boxplot(rating ~ cookie, data = evidence,
        col = "lightsteelblue",
        main = "Case File #2: The Lineup",
        ylab = "Taster Rating (out of 10)",
        cex.axis = 0.8)
```

**What they'll see** (descriptives to have on hand):

| Cookie | M | SD | n |
|---|---|---|---|
| Snickerdoodle | 8.25 | 1.20 | 60 |
| Chocolate Chip | 8.07 | 1.04 | 60 |
| Peanut Butter | 7.80 | 1.16 | 60 |
| Sugar | 7.17 | 1.29 | 60 |
| Oatmeal | 6.82 | 1.21 | 60 |

**Teaching beats:** the `$` sign as "open the evidence box and pull out one item"; `rating ~ cookie` reads as "rating *as a function of* cookie — the tilde will be everywhere from now on"; the new syntax `aggregate(...)` is the same walk-the-scene ritual as Case #1, just scaled up to five groups at once. Ask the room: "Snickerdoodle is on top again. Case closed?" They should now say it in chorus: *describe before you accuse.*

---

## Part 3: The Temptation — Why Not Ten t-tests? (10 min)

This is the conceptual heart of the session. Pose it as a genuine question: "You know the t-test. Five cookies make ten possible pairs. Why not just run ten t-tests?"

Let them propose it, then walk through the interrogation analogy:

- "Each t-test at alpha = .05 carries a 5% chance of convicting an innocent suspect — a false alarm. Run one interrogation, fine. Run ten, and the chance that *at least one* innocent suspect cracks and looks guilty is 1 − (.95)^10 = **40%**."
- Write the number large: **a 40% chance of at least one wrongful conviction**, before you've looked at a single cookie. "A detective who interrogates enough innocent people will eventually get a false confession. That's not investigation — that's how scandals happen." (Optional dark aside for the curious: this is also the mechanism behind p-hacking in published research.)
- The fix is two-stage: "First, one **omnibus** question for the whole room: *did anyone in this lineup do it?* That's ANOVA. Only if the room is guilty do we interrogate individuals — and when we do, we use a **corrected standard of proof** so the family of ten interrogations together keeps the overall false-alarm rate at 5%. That's Tukey's HSD."

---

## Part 4: The Lineup — One-Way ANOVA (15 min)

```r
# --- The lineup: did anyone in this room do it? ---
lineup <- aov(rating ~ cookie, data = evidence)
summary(lineup)
```

Output to project (these are the real values for this dataset):

```
             Df Sum Sq Mean Sq F value   Pr(>F)
cookie        4   88.8  22.195   15.82 9.63e-12 ***
Residuals   295  413.9   1.403
```

**Line-by-line translations:**

- **F = 15.82 — the same ratio as Case #1, scaled to a room.** "F = variability *between* the cookie means ÷ ordinary bounce *within* each cookie. Signal over noise, again. The differences between cookies are almost sixteen times larger than taster-to-taster bounce can explain." (Point back to the `t = signal ÷ noise` slide from Case #1 — every test this semester is a version of that slide.)
- **Mean Sq column** — the two ingredients of the ratio, visible: 22.195 / 1.403 = 15.82. Students can verify by division; let one do it out loud.
- **p = 9.63e-12** — teach scientific notation here: "9.63 × 10⁻¹², about one in a hundred billion. If all five cookies were truly equal, evidence this extreme would essentially never occur." Then the careful sentence, every time: that is a statement about the evidence under innocence, not the probability of innocence.
- **The verdict, and its limit:** "We reject the null that all five means are equal. But notice what the F-test did NOT tell us: *who*. The omnibus verdict is only 'someone in this room did it.' Lineups don't name names — interrogations do."

---

## Part 5: The Interrogations — Tukey HSD (15 min)

```r
# --- The interrogations: WHO did it? (corrected standard of proof) ---
TukeyHSD(lineup)
```

Key rows of the output (full table on the student handout):

```
                                  diff    lwr    upr  p adj
Oatmeal-Chocolate Chip          -1.250 -1.844 -0.656  .000
Peanut Butter-Chocolate Chip    -0.267 -0.860  0.327  .732
Snickerdoodle-Chocolate Chip     0.183 -0.410  0.777  .915
Sugar-Chocolate Chip            -0.900 -1.494 -0.306  .000
Peanut Butter-Oatmeal            0.983  0.390  1.577  .000
Snickerdoodle-Oatmeal            1.433  0.840  2.027  .000
Sugar-Oatmeal                    0.350 -0.244  0.944  .487
Snickerdoodle-Peanut Butter      0.450 -0.144  1.044  .231
Sugar-Peanut Butter             -0.633 -1.227 -0.040  .030
Sugar-Snickerdoodle             -1.083 -1.677 -0.490  .000
```

**Run the reveal as a structured hunt.** Give them three minutes in pairs with one instruction: "Six pairs differ; four don't. Find the pattern." The pattern:

- **Top tier:** Snickerdoodle ≈ Chocolate Chip ≈ Peanut Butter (all three mutual comparisons non-significant).
- **Bottom tier:** Sugar ≈ Oatmeal (non-significant).
- **Every cross-tier comparison is significant.** The lineup found two gangs.

**The twist to land hard:** "Snickerdoodle — our Case #1 celebrity — cannot be statistically distinguished from chocolate chip (p = .92). A mean of 8.25 vs 8.07 looks like a winner on a bar chart; the interrogation says we have no evidence they differ. If the owner had featured snickerdoodles as 'the proven best cookie,' that claim would not survive cross-examination. Headlines about 'the #1 ranked X' commit this error constantly."

**Reading notes:** `p adj` = the corrected standard of proof in action — these p-values are adjusted so the *family* of ten interrogations keeps a 5% overall false-alarm rate. And the CI rule carries over from Case #1: pairs whose interval contains zero are the ones we can't convict.

---

## Part 6: The Size of the Crime — Eta-Squared (8 min)

```r
# --- How big a deal is the cookie factor overall? ---
# eta^2 = SS_between / SS_total  (pull the numbers from summary(lineup))
88.8 / (88.8 + 413.9)    # 0.177
```

"About **18% of all the variation** in ratings is explained by which cookie people ate; the other 82% is everything else — sweet tooth, mood, what they had for lunch. Rough benchmarks for eta-squared: .01 small, .06 medium, .14 large. Cookie identity is a large effect — and even a *large* effect leaves most variation unexplained. That ratio (some explained, most not) is what almost every result in psychology looks like; better to learn it from cookies than be demoralized by it in a thesis."

**Verdict write-up to model (students add as a comment):**

```r
# VERDICT: Cookie type significantly affected ratings,
# F(4, 295) = 15.82, p < .001, eta^2 = .18. Tukey HSD showed
# two tiers: Snickerdoodle, Chocolate Chip, and Peanut Butter
# were rated higher than Sugar and Oatmeal (all cross-tier
# ps < .05); cookies within each tier did not differ.
```

---

## Wrap-Up & Exit Ticket (5 min)

1. Why is running ten t-tests on five groups detective malpractice? (Looking for: false-alarm rate compounds; ~40%.)
2. The F-test came back significant. What does that verdict tell you — and what does it NOT tell you?
3. Snickerdoodle's mean (8.25) is higher than chocolate chip's (8.07). Why can't we feature snickerdoodle as "the proven best cookie"?

**Closing line:** "Case #1, you interrogated one suspect. Today you ran a lineup of five, kept your false-alarm rate honest, and found two gangs instead of one hero. Next case: the same witnesses interviewed twice — your own R-anxiety ratings, before and after."

---

## Appendix: Anticipated Questions & Snags

| Question / symptom | Response |
|---|---|
| `cannot open file 'cookie-lineup.csv'` | The session's most valuable error. Either (a) wrong kitchen — they opened RStudio directly instead of the `.Rproj`, or (b) the evidence never made the locker — file is still in Downloads. Files pane shows the truth. |
| File opens in Excel when double-clicked | They double-clicked the CSV, not the `.Rproj`. CSVs are boxes, not doors — close Excel (without saving!) and load it through R. |
| "Why `aov()` and not `anova()`?" | `aov()` fits the model; `anova()`/`summary()` print tables from fitted models. The lineup vs. the lineup's report. |
| "Do the Tukey p-values match ten separate t-tests?" | No — they're deliberately more conservative. That's the corrected standard of proof; some pairs significant under naive t-tests stop being significant under Tukey, on purpose. |
| "What if the F-test had NOT been significant?" | Then the investigation stops — no interrogations. Post-hoc hunting after a non-significant omnibus is interrogating a room the evidence already cleared. |
| "Is 18% explained variance good?" | Large by behavioral-science benchmarks. Most published psychology effects explain far less; calibrate expectations now. |
| Student asks about assumptions | Equal-ish group sizes (ours are exactly equal: 60 each) and similar spreads (SDs 1.04–1.29) keep ANOVA well-behaved; formal assumption checks ("chain-of-custody paperwork") get their own session. |
