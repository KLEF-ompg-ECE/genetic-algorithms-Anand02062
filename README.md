# Assignment 2 — Genetic Algorithm: Knapsack Problem
## Observation Report

**Student Name  :** ___C.ANANd _ _  
**Student ID    :** ___2310040129_________  
**Date Submitted:** _______12/03/2026_________  

---

## How to Submit

1. Run each experiment following the instructions below
2. Fill in every answer box — do not leave placeholders
3. Make sure the `plots/` folder contains all required images
4. Commit this README and the `plots/` folder to your GitHub repo

---

## Before You Begin — Read the Code

Open `ga_knapsack.py` and read through it. Then answer these questions.

**Q1. What does the `fitness()` function return? Why does an overweight solution score 0?**

```
The fitness() function returns the total value of all packed items if the total
weight is within the 15 kg limit, or 0 if the weight exceeds the limit.
An overweight solution scores 0 because it represents an invalid packing — the
trekker physically cannot carry more than 15 kg, so those solutions must be
penalised heavily enough that the algorithm never selects them for reproduction.
Returning 0 (the lowest possible score) ensures overweight chromosomes are
effectively excluded from the gene pool.
```

**Q2. What does `tournament_select()` do? Why are higher-fitness individuals more likely to be chosen?**

```
tournament_select() randomly picks k=3 individuals from the population and
returns a copy of whichever one has the highest fitness score among that group.
Higher-fitness individuals are more likely to win because they are evaluated
against only a small random subset of the population, not the whole field —
so a strong individual nearly always beats random competition. This mimics
natural selection: fitter organisms tend to survive and reproduce, but weaker
ones still have a small chance of being selected, preserving genetic diversity.
```

**Q3. Look at the `run_ga()` loop. Find this line:**
```python
next_gen = [best_chromosome[:]]
```
**What is this doing? Why is it important to always keep the best solution?**

```
This line seeds the next generation's population with an exact copy of the
best chromosome found so far — a technique called elitism. It is important
because crossover and mutation are stochastic operations that can accidentally
destroy a good solution; without elitism, the best-ever fitness could decrease
between generations. By guaranteeing the champion always survives, the algorithm
ensures that the best value found never goes backwards, which is why the
value_log curve in the plots only ever stays flat or rises — it never dips.
```

---

## Experiment 1 — Baseline Run

**Instructions:** Run the program without changing anything.
```bash
python ga_knapsack.py
```

**Fill in this table:**

| Metric | Your result |
|--------|-------------|
| Number of generations | 50 |
| Best value at generation 1 | 60 |
| Final best value | 77 |
| Total weight of best solution (kg) | 14.4 |
| Is solution valid (Yes / No) | Yes |

**Copy the printed packing list here:**
```
  Best Packing List
--------------------------------------
  + Water bottle
  + First aid kit
  + Sleeping bag
  + Torch
  + Energy bars (x6)
  + Rain jacket
  + Map & compass
  + Cooking stove
  + Rope (10 m)
  + Sunscreen
  + Power bank
--------------------------------------
  Weight : 14.4 / 15.0 kg
  Value  : 77
  Valid  : Yes
```

**Look at `plots/experiment_1.png` and describe what you see (2–3 sentences).**  
*Where does the biggest improvement happen? Does the curve flatten at some point?*
```
The curve rises steeply in the first 10 generations, jumping from 60 up toward
the high 70s as the algorithm quickly discovers good combinations of items.
After roughly generation 10–15 the line flattens completely into a plateau at
value 77, meaning the population has converged and no further improvement is
found in the remaining ~35 generations. This pattern is typical of a
well-tuned GA: rapid early exploration followed by stable exploitation of
the best solution found.
```

---

## Experiment 2 — Effect of Mutation Rate

**Instructions:** In `ga_knapsack.py`, find the `# EXPERIMENT 2` block in `__main__`.  
Copy it three times and run with `mutation_rate` = **0.01**, **0.05**, and **0.30**.  
Save plots as `experiment_2a.png`, `experiment_2b.png`, `experiment_2c.png`.

**Results table:**

| mutation_rate | Final best value | Weight (kg) | Valid? | Shape of curve |
|--------------|-----------------|-------------|--------|----------------|
| 0.01         | 75              | 14.9        | Yes    | Rises quickly then flatlines early (lower plateau) |
| 0.05         | 77              | 14.4        | Yes    | Steady rise then clean plateau — best convergence |
| 0.30         | 78              | 14.1        | Yes    | Noisy/jagged but occasionally finds higher values |

**Compare the three plots. What happens when mutation is too low? Too high? (3–4 sentences)**  
*Hint: Too low = no diversity, may get stuck. Too high = random search. What is the sweet spot?*
```
With mutation_rate=0.01, the curve converges very fast but plateaus at a lower
value (75) because the population loses diversity quickly — without enough
random bit-flips, the algorithm gets trapped in a local optimum and stops
exploring new combinations. With mutation_rate=0.30, nearly a third of all bits
are flipped every generation, which constantly destroys good solutions almost
as fast as they are found; the curve is noisy and erratic, essentially behaving
like a near-random search that stumbles on a good answer by luck rather than
directed evolution. The sweet spot at 0.05 balances exploration and exploitation:
enough mutation to escape local optima and maintain diversity, but not so much
that it undoes the progress made by selection and crossover.
```

**Which mutation_rate gave the best result? Why do you think that is?**
```
Technically mutation_rate=0.30 produced the highest final value (78) in this
particular run, but this is somewhat misleading — the result is inconsistent and
driven by luck rather than reliable convergence, and across different seeds or
more generations it would often perform worst. The mutation_rate=0.05 setting
is the most dependable choice: it consistently reaches value 77, converges
smoothly, and would generalise well to different random seeds. The 0.30 rate
"got lucky" by randomly flipping into a slightly better combination, but its
jagged curve shows it cannot be relied upon to do so systematically.
```

---

## Summary

**Complete this table with your best result from each experiment:**

| Experiment | Key setting | Final value | Main finding in one sentence |
|------------|-------------|-------------|------------------------------|
| 1 — Baseline | mutation_rate = 0.05 | 77 | The GA converges to a high-quality, valid solution in under 15 generations and holds it stably for the remaining 35. |
| 2 — Mutation rate | mutation_rate = 0.05 | 77 | Balanced mutation (0.05) gives the most reliable convergence; too low causes premature stagnation, too high introduces destructive noise. |

**In your own words — what is the most important thing you learned about Genetic Algorithms from these experiments? (3–5 sentences)**
```
The most important insight is that a Genetic Algorithm's performance is highly
sensitive to the balance between exploration and exploitation, and mutation rate
is the key dial that controls this balance. Too little mutation starves the
population of diversity, causing it to lock onto a mediocre solution early and
never escape; too much mutation effectively turns the GA into a random search
that ignores the useful information built up through selection. The elitism
strategy (always keeping the best chromosome) was also a crucial detail — it
ensures progress is never accidentally reversed by a bad crossover or mutation
event, giving the value curve its characteristic "staircase up, then flat"
shape. Finally, these experiments show that a good GA result requires tuning
several interacting parameters together; the default 0.05 mutation rate worked
well here, but the right value would differ for harder problems with more items
or tighter weight constraints.
```

---

## Submission Checklist

- [done] Student name and ID filled in
- [done] Q1, Q2, Q3 answered
- [done] Experiment 1: table filled, packing list pasted, plot observation written
- [done] Experiment 2: results table filled (3 rows), observation and answer written
- [done] Summary table completed and reflection written
- [done] `plots/` contains: `experiment_1.png`, `experiment_2a.png`, `experiment_2b.png`, `experiment_2c.png`
