# PSSP — Probability, Statistics, Sampling & Linear Algebra
### Interview Prep Notes — ZS / Fractal / Tiger Analytics / PwC
> **Depth**: Intermediate → Advanced | **Rounds**: R1 (30 min screening) + R2 (60 min deep dive)
> Hypothesis Testing, p-values, α, and significance levels are **heavily tested**. Know them cold.

---

# ═══════════════════════════════════════════════
# TOPIC 1: PROBABILITY FUNDAMENTALS
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

**What Probability Really Is:**
Probability measures how likely an event is to occur. Two schools of thought:
- **Frequentist**: Long-run frequency of an event. If you flip a coin 10,000 times, the fraction of heads ≈ 0.5. That's the probability.
- **Bayesian**: Degree of belief updated with evidence. "I believe there's a 70% chance it rains tomorrow" — even though tomorrow only happens once.

**Three Axioms (Kolmogorov) — The Foundation:**
1. $P(A) \geq 0$ for any event $A$ — probabilities can't be negative
2. $P(\Omega) = 1$ (something must happen — the sample space has probability 1)
3. $P(A \cup B) = P(A) + P(B)$ if $A \cap B = \emptyset$ (you can add probabilities of **mutually exclusive** events)

**Key Concepts to Nail:**

| Concept | Definition | Interview Trap |
|---|---|---|
| **Independence** | $P(A \cap B) = P(A) \cdot P(B)$ | Independent ≠ Mutually Exclusive. If A,B are M.E. and both have P>0, they are **dependent** |
| **Conditional Probability** | $P(A \mid B) = \frac{P(A \cap B)}{P(B)}$ | $P(A \mid B) \neq P(B \mid A)$ — this is the **prosecutor's fallacy** |
| **Law of Total Probability** | $P(A) = \sum_i P(A \mid B_i) P(B_i)$ | Partition must be exhaustive & mutually exclusive |
| **Complement Rule** | $P(A') = 1 - P(A)$ | Use for "at least one" problems: $P(\geq 1) = 1 - P(\text{none})$ |

### Independent vs Mutually Exclusive — The #1 Confusion

These are **completely different concepts**. Let's nail this:

- **Mutually Exclusive (M.E.)**: Events that **cannot happen together**. If A happens, B cannot. $P(A \cap B) = 0$.
  - Example: Rolling a 3 and rolling a 5 on the same die. Can't happen simultaneously.
- **Independent**: Events that **don't affect each other's probability**. $P(A|B) = P(A)$, which means $P(A \cap B) = P(A) \cdot P(B)$.
  - Example: Flipping heads on coin 1 and rolling a 6 on a die. One doesn't affect the other.

**Why M.E. events with P>0 are DEPENDENT:**
If A and B are mutually exclusive and both have positive probability, then knowing A happened tells you B **definitely didn't** happen. That means $P(B|A) = 0 \neq P(B)$. They affect each other → **dependent**!

Think of it this way:
- Independent = "I don't care what happened to you" → $P(A \cap B) = P(A) \cdot P(B) > 0$
- Mutually Exclusive = "If you happened, I definitely didn't" → $P(A \cap B) = 0$
- These two can't both be true (unless one event has P=0)

**Geometric Intuition:**
```
Sample Space Ω (entire rectangle)
┌─────────────────────────┐
│         ┌───┐           │
│    A    │A∩B│    B      │  ← Independent events CAN overlap
│         └───┘           │
│                         │
│      A' ∩ B'            │
└─────────────────────────┘

┌─────────────────────────┐
│  ┌────┐      ┌────┐     │
│  │  A │      │  B │     │  ← Mutually Exclusive: NO overlap
│  └────┘      └────┘     │
│                         │
└─────────────────────────┘

P(A|B) = shaded A∩B / entire B region
```

## 2. MATHEMATICS THAT MATTERS

**Conditional Probability Chain Rule:**
$$P(A_1 \cap A_2 \cap \ldots \cap A_n) = P(A_1) \cdot P(A_2|A_1) \cdot P(A_3|A_1 \cap A_2) \cdots$$

This lets you break down complex joint probabilities step-by-step. Example: P(rain AND carry umbrella AND stay dry) = P(rain) × P(carry umbrella | rain) × P(stay dry | rain AND carry umbrella).

**Inclusion-Exclusion (2 events):**
$$P(A \cup B) = P(A) + P(B) - P(A \cap B)$$
Why subtract? Because when you add P(A) and P(B), the overlap region gets counted **twice**. You subtract it once to fix that.

**Expected Value & Variance:**

**Expected Value** = the "average outcome" if you repeated the experiment infinitely:
- $E[X] = \sum x_i \cdot P(x_i)$ — weighted average, where weights are probabilities
- Example: Fair die → $E[X] = 1(1/6) + 2(1/6) + ... + 6(1/6) = 3.5$. You never actually roll 3.5, but it's the long-run average.
- $E[aX + b] = aE[X] + b$ — expectation is **linear** (this is very useful!)

**Variance** = how spread out the values are around the mean:
- $\text{Var}(X) = E[(X - \mu)^2] = E[X^2] - (E[X])^2$ — average squared deviation from mean
- $\text{Var}(aX + b) = a^2 \text{Var}(X)$ — shifting by $b$ doesn't change spread, scaling by $a$ squares the effect
- **Standard Deviation** $\sigma = \sqrt{\text{Var}(X)}$ — same units as the data (more interpretable than variance)

**Why two formulas for Variance?**
$E[(X-\mu)^2]$ is the definition (average squared distance from mean). $E[X^2] - (E[X])^2$ is the **computational shortcut** — easier to calculate. They're algebraically identical.

## 3. TOP INTERVIEW QUESTIONS

**Q1 (Easy):** _You flip a fair coin 3 times. What's P(at least one head)?_
> $P(\geq 1H) = 1 - P(\text{no heads}) = 1 - (1/2)^3 = 7/8$

**Q2 (Medium):** _Two dice are rolled. Given the sum is ≥ 10, what's P(at least one 6)?_
> Sum ≥ 10: {(4,6),(5,5),(5,6),(6,4),(6,5),(6,6)} = 6 outcomes. With at least one 6: {(4,6),(5,6),(6,4),(6,5),(6,6)} = 5. P = 5/6.

**Q3 (Hard — Bayesian setup):** _1% of population has disease. Test is 99% sensitive, 95% specific. You test positive. What's P(disease)?_
> Use Bayes' Theorem → P(D|+) = (0.99 × 0.01) / (0.99×0.01 + 0.05×0.99) ≈ **16.7%** — This shocks interviewers. Low base rate kills precision.

**Red Flags:**
- Confusing independent with mutually exclusive
- Forgetting to check if partition is exhaustive before applying total probability
- Not recognizing base rate problems

---

# ═══════════════════════════════════════════════
# TOPIC 2: PROBABILITY DISTRIBUTIONS
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

### Discrete Distributions

| Distribution | Use Case | PMF / Key Formula | Parameters |
|---|---|---|---|
| **Bernoulli** | Single yes/no trial | $P(X=1) = p$ | $p$ = success prob |
| **Binomial** | # successes in $n$ independent trials | $\binom{n}{k} p^k (1-p)^{n-k}$ | $n, p$ |
| **Poisson** | # events in fixed interval (rare events) | $\frac{\lambda^k e^{-\lambda}}{k!}$ | $\lambda$ = avg rate |
| **Geometric** | # trials until first success | $(1-p)^{k-1} p$ | $p$ |

**Key Relationship:** Binomial → Poisson when $n \to \infty$, $p \to 0$, $np = \lambda$ stays constant.

### Continuous Distributions

| Distribution | Use Case | PDF Key Characteristic | Parameters |
|---|---|---|---|
| **Uniform** | Equal likelihood | $f(x) = \frac{1}{b-a}$ | $a, b$ |
| **Normal (Gaussian)** | Natural phenomena, CLT | Bell curve, symmetric | $\mu, \sigma^2$ |
| **Exponential** | Time between Poisson events | $f(x) = \lambda e^{-\lambda x}$ | $\lambda$ = rate |
| **Log-Normal** | Incomes, stock prices (multiplicative processes) | Right-skewed, $\ln(X) \sim N$ | $\mu, \sigma$ of $\ln(X)$ |

### The Normal Distribution — Know This Cold

**68-95-99.7 Rule:**
```
         99.7%
      ┌───────────────┐
      │    95%         │
      │  ┌─────────┐   │
      │  │  68%    │   │
──────┼──┼────┼────┼──┼──────
    μ-3σ μ-2σ  μ  μ+2σ μ+3σ
```

**Standard Normal:** $Z = \frac{X - \mu}{\sigma}$ — converts any Normal to $N(0,1)$

**Why Normal is Everywhere:** Central Limit Theorem (covered below).

## 2. MATHEMATICS THAT MATTERS

**Gaussian PDF:**
$$f(x) = \frac{1}{\sigma\sqrt{2\pi}} \exp\left(-\frac{(x-\mu)^2}{2\sigma^2}\right)$$

**Key Properties:**
- Sum of independent normals: $X \sim N(\mu_1, \sigma_1^2), Y \sim N(\mu_2, \sigma_2^2) \Rightarrow X+Y \sim N(\mu_1+\mu_2, \sigma_1^2+\sigma_2^2)$
- **Skewness** = 0 (symmetric), **Kurtosis** = 3 (mesokurtic)

**Skewness (Frequently Asked):**
$$\text{Skewness} = \frac{E[(X-\mu)^3]}{\sigma^3}$$
- Positive skew → right tail longer → **Mean > Median > Mode**
- Negative skew → left tail longer → **Mean < Median < Mode**
- Zero skew → symmetric (Normal)

```
Positive Skew        Negative Skew        Normal (Zero Skew)
     ╱╲                    ╱╲                  ╱╲
    ╱  ╲___           ___╱  ╲               ╱    ╲
   ╱       ╲         ╱       ╲            ╱        ╲
  ╱_________╲       ╱_________╲         ╱____________╲
  Mo Md  Mean       Mean Md  Mo          Mo=Md=Mean
```

## 3. TOP INTERVIEW QUESTIONS

**Q1:** _When would you use Poisson over Binomial?_
> When $n$ is very large, $p$ is very small, and you're counting rare events in a fixed window (e.g., # server crashes/day, # typos/page). Poisson is the limiting case of Binomial.

**Q2:** _Data is right-skewed. What transformation?_
> Log transform, square root, or Box-Cox. Log works when data is log-normally distributed (e.g., income, prices).

**Q3:** _Name a real-world distribution that is NOT Normal._
> Income (log-normal), time-to-failure (exponential/Weibull), count of website clicks (Poisson), customer ratings (uniform-ish or bimodal).

---

# ═══════════════════════════════════════════════
# TOPIC 3: SAMPLING
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

**Why Sampling Matters:** We can't measure the entire population. Sampling lets us infer population parameters from a subset.

| Sampling Method | How It Works | When To Use | Pitfall |
|---|---|---|---|
| **Simple Random (SRS)** | Every member has equal probability | Homogeneous population | Expensive for large populations |
| **Stratified** | Divide into strata → SRS within each | Heterogeneous population with known subgroups | Requires known strata boundaries |
| **Cluster** | Divide into clusters → randomly pick entire clusters | Geographically dispersed | High within-cluster variance reduces accuracy |
| **Systematic** | Every $k$-th element from list | Quick, ordered lists | Periodicity in data can bias results |
| **Convenience** | Whatever is easiest to access | Pilot studies, EDA | **Never** for inference — heavy selection bias |

**Stratified vs. Cluster — Classic Interview Confusion:**
```
STRATIFIED:                    CLUSTER:
┌──────┬──────┬──────┐        ┌──────┐ ┌──────┐ ┌──────┐
│Strata│Strata│Strata│        │Clust.│ │Clust.│ │Clust.│
│  1   │  2   │  3   │        │  1   │ │  2   │ │  3   │
│ ●●●  │ ●●●  │ ●●●  │        │ ●●●● │ │      │ │ ●●●● │
│sample│sample│sample│        │ ALL  │ │SKIP  │ │ ALL  │
│ few  │ few  │ few  │        │taken │ │      │ │taken │
└──────┴──────┴──────┘        └──────┘ └──────┘ └──────┘
Sample FROM each stratum      Sample ENTIRE clusters
```

## 2. MATHEMATICS THAT MATTERS

**Sample Mean:**
$$\bar{X} = \frac{1}{n}\sum_{i=1}^{n} X_i$$
Just the average of your sample. It's your **best estimate** of the true population mean $\mu$.

**Standard Error (SE) — What It Actually Means:**
$$SE(\bar{X}) = \frac{\sigma}{\sqrt{n}}$$

- $\sigma$ = population standard deviation (how spread out individual data points are)
- $n$ = sample size

**Intuition:** Imagine you take 100 different samples of size $n$ from the same population and compute $\bar{X}$ for each. Those 100 sample means will themselves have some spread. The **standard error IS that spread** — it tells you how much your sample mean would vary if you repeated the sampling.

- Small SE → your estimate is **precise** (repeated samples give similar $\bar{X}$)
- Large SE → your estimate is **imprecise** (repeated samples give wildly different $\bar{X}$)

**Key Insight:** SE ↓ as $n$ ↑ → more data = more precise estimates. BUT there are **diminishing returns**:
- **Quadratic cost:** To halve the SE, you need **4× the sample size** (because $\sqrt{4n} = 2\sqrt{n}$)
- Going from n=100 to n=400 halves the SE. Going from n=400 to n=1600 halves it again. Expensive!

**Standard Error vs Standard Deviation — Common Confusion:**
| | Standard Deviation (σ or s) | Standard Error (SE) |
|---|---|---|
| Measures | Spread of **individual data points** | Spread of **sample means** |
| Formula | $\sqrt{\frac{1}{n}\sum(x_i - \bar{x})^2}$ | $\frac{\sigma}{\sqrt{n}}$ |
| Depends on n? | No (it's a population property) | Yes (↓ as n ↑) |
| Used for | Describing data variability | Constructing CIs, hypothesis tests |

**Sampling Bias Types:**
- **Selection Bias**: Non-random selection (e.g., surveying only online users about internet usage → overestimates)
- **Survivorship Bias**: Only studying "survivors" (e.g., only looking at successful startups → concluding entrepreneurship is easy)
- **Non-response Bias**: Systematic differences between responders & non-responders (e.g., satisfied customers are more likely to respond to surveys)

## 3. TOP INTERVIEW QUESTIONS

**Q1:** _You need to survey customer satisfaction across 5 cities with different demographics. Which sampling method?_
> **Stratified sampling** — stratify by city (and possibly demographics within each city), then SRS within each stratum. Ensures representation from all subgroups.

**Q2:** _What happens to standard error if you quadruple sample size?_
> SE = σ/√n. Quadrupling n → SE halves. Diminishing returns on precision.

---

# ═══════════════════════════════════════════════
# TOPIC 4: CORRELATIONS
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

| Measure | Type | Range | Assumption | Use Case |
|---|---|---|---|---|
| **Pearson $r$** | Linear | $[-1, +1]$ | Both variables ~Normal, linear relationship | Continuous, normally distributed |
| **Spearman $\rho$** | Monotonic | $[-1, +1]$ | None (rank-based) | Ordinal data, non-linear monotonic |
| **Kendall $\tau$** | Concordance | $[-1, +1]$ | None (rank-based) | Small samples, robust to ties |
| **Point-Biserial** | Linear | $[-1, +1]$ | One binary, one continuous | Binary × continuous |

## 2. MATHEMATICS THAT MATTERS

**Pearson Correlation:**
$$r = \frac{\sum(x_i - \bar{x})(y_i - \bar{y})}{\sqrt{\sum(x_i-\bar{x})^2 \cdot \sum(y_i-\bar{y})^2}} = \frac{\text{Cov}(X,Y)}{\sigma_X \cdot \sigma_Y}$$

**Critical Insight: Correlation ≠ Causation**
- Ice cream sales & drowning deaths are correlated. Confounder: **temperature**.
- Always ask: _Is there a lurking variable?_

**Correlation vs. Regression:**
- Correlation = **strength & direction** of association (symmetric: $r_{XY} = r_{YX}$)
- Regression = **prediction** (asymmetric: regressing Y on X ≠ X on Y)

## 3. TOP INTERVIEW QUESTIONS

**Q1:** _Pearson r = 0. Does that mean no relationship?_
> **No!** It means no **linear** relationship. There could be a perfect quadratic, sinusoidal, or other non-linear relationship. Always plot first.

**Q2:** _When would you prefer Spearman over Pearson?_
> When data is ordinal, has outliers, or the relationship is monotonic but non-linear (e.g., logarithmic). Spearman is robust to outliers because it works on ranks.

---

# ═══════════════════════════════════════════════
# TOPIC 5: HYPOTHESIS TESTING — HEAVILY TESTED
# ═══════════════════════════════════════════════

## 1. INTUITION & REAL WORLD EXAMPLES

> This is the #1 tested topic. Master this inside-out.

**What is Hypothesis Testing? (Beginner Friendly!)**
Hypothesis Testing is just a formal way of asking: *"Is this new thing actually working, or did it just happen by random luck?"*

**1. The Magic Coin Analogy:**
Imagine you find a coin and suspect it's "magic" (weighted to land on Heads more often).
*   **H₀ (Null Hypothesis):** It's just a normal, boring coin. (Status quo, no effect).
*   **H₁ (Alternative Hypothesis):** It's a magic coin! (The effect exists).

You flip it 10 times, and it lands on Heads 9 times. 
If it were a normal coin (H₀ is true), getting 9 Heads by pure luck is extremely rare (this tiny probability is your **p-value**). 
Because seeing 9 Heads is so incredibly unlikely for a normal coin, you stop believing it's normal (**Reject H₀**) and conclude it probably is magic!

**2. The Courtroom Analogy:**
*   **H₀:** The defendant is innocent.
*   **H₁:** The defendant is guilty.
The **p-value** asks: "If this person were truly innocent (H₀), how likely is it that we'd find all this DNA evidence against them?" If it's highly unlikely (e.g., p < 0.05), we reject innocence (reject H₀) and declare them guilty. Note: we're NOT saying "there's a 5% chance they are innocent." We are saying the *evidence* would be very rare if they were innocent.

---

## 1.5. THE FORMAL DEFINITIONS (Cheat Sheet)

Once you understand the intuition, here is how it is formally written and tested in interviews:

**The Framework:**
```text
[State Hypotheses] → [Choose Test & α] → [Compute Test Statistic] → [Find p-value] → [Decision]
```

**Key Definitions & Terminology:**
| Term | Symbol | Meaning |
|---|---|---|
| **Null Hypothesis** | $H_0$ | The boring status quo. No effect exists. |
| **Alternative Hypothesis** | $H_1$ | The effect exists! (What you are trying to prove). |
| **Significance Level** | $\alpha$ | The threshold of "rare". Usually 0.05 (5%). If p-value < α, you reject $H_0$. |
| **p-value** | $p$ | P(observing this evidence \| H₀ is true). (NOT the probability that $H_0$ is true!) |
| **Type I Error** | $\alpha$ | False Positive (Convicting an innocent person, or saying a normal coin is magic). |
| **Type II Error** | $\beta$ | False Negative (Letting a guilty person go, or saying a magic coin is normal). |
| **Power** | $1 - \beta$ | P(correctly rejecting a false H₀). You want this to be high (≥ 80%). |

**What p-value IS and IS NOT:**
- ✅ P(data this extreme | H₀ true) — evidence against H₀
- ❌ NOT P(H₀ is true) — that's the posterior, requires Bayesian framework
- ❌ NOT the probability that results are "due to chance"
- ❌ NOT a measure of effect size — statistical significance ≠ practical significance

**One-tailed vs Two-tailed Tests:**
- **Two-tailed**: H₁ says "parameter ≠ some value" (effect in either direction). p-value counts extreme values on BOTH sides.
- **One-tailed**: H₁ says "parameter > value" or "parameter < value" (you have a specific direction in mind). p-value counts only one side.
- **When to use which?** Default to two-tailed unless you have a strong a priori reason to test only one direction. One-tailed is more powerful (lower p for same effect) but risky if the effect is in the opposite direction.

## 2. CHOOSING THE RIGHT TEST (Step-by-Step)

Before looking at the math, think of these statistical tests as **tools in a toolbox**. You wouldn't use a hammer to drive a screw. Similarly, you choose a test based exactly on what you are trying to compare.

**Step 1: The Intuition (Which tool do I need?)**
*   **Comparing ONE thing to a standard?** (e.g., "Is our new drug better than the national average?") -> **Use a 1-sample t-test** (or Z-test if you have tons of data).
*   **Comparing TWO things against each other?** (e.g., "Is Drug A better than Drug B?") -> **Use a 2-sample t-test.**
*   **Comparing THREE OR MORE things?** (e.g., "Which is best: Drug A, Drug B, or Drug C?") -> **Use ANOVA.** (ANOVA tells you if at least one is different. Why not just run multiple t-tests? Because doing A vs B, B vs C, and A vs C increases the chance of a false alarm/Type I error!).
*   **Dealing with Categories instead of numbers?** (e.g., "Does gender affect voting preference (Red/Blue)?") -> **Use Chi-Square.**

**Step 2: The Decision Tree (A Simple Guide)**
```text
Are you measuring NUMBERS (like height, salary, time)?
├── YES! How many groups are you comparing?
│   ├── 1 group (vs a known target) → One-Sample t-test
│   ├── 2 groups (A vs B)           → Two-Sample t-test
│   └── 3+ groups (A vs B vs C)     → ANOVA
│
└── NO! I'm measuring CATEGORIES (like Colors, Yes/No, Brands)
    └── Chi-Square Test
```

**Step 3: The Math (For Completeness)**
Once you've chosen your tool, here are the underlying formulas that the computer calculates for you:

**Z-test (known σ, large n):**
$$Z = \frac{\bar{X} - \mu_0}{\sigma / \sqrt{n}}$$

**t-test (unknown σ, small n - most common):**
$$t = \frac{\bar{X} - \mu_0}{s / \sqrt{n}}, \quad df = n - 1$$
- $\bar{X}$ = sample mean, $\mu_0$ = hypothesized mean, $s$ = sample std dev
- t-distribution has heavier tails than Normal → more conservative with small samples

**Two-Sample t-test (Independent):**
$$t = \frac{\bar{X}_1 - \bar{X}_2}{\sqrt{\frac{s_1^2}{n_1} + \frac{s_2^2}{n_2}}}$$

**ANOVA (Analysis of Variance):**
- Uses the **F-statistic** = (variance BETWEEN groups) / (variance WITHIN groups)
- Large F → the differences between the groups are bigger than the random noise inside the groups → likely significant.

**Chi-Square Test:**
$$\chi^2 = \sum \frac{(O_i - E_i)^2}{E_i}$$
- $O_i$ = Observed frequency, $E_i$ = Expected frequency
- Large $\chi^2$ → What you observed differs significantly from what you expected by random chance.

## 3. ALGORITHM WORKFLOW

```
[Business Question]
       │
       ▼
[Formulate H₀ and H₁]
       │
       ▼
[Choose α (usually 0.05)]
       │
       ▼
[Select Appropriate Test]     ← based on data type, sample size, # groups
       │
       ▼
[Check Assumptions]           ← normality, equal variance, independence
       │
       ▼
[Calculate Test Statistic]
       │
       ▼
[Compute p-value]
       │
       ▼
[p < α?]──── YES ──→ Reject H₀ (statistically significant)
   │
   NO
   │
   ▼
Fail to Reject H₀ (NOT "accept H₀")
```

## 4. REAL-WORLD APPLICATIONS

**A/B Testing (Frequently Asked):**
- H₀: No difference between control (A) and treatment (B)
- H₁: Treatment has a different conversion rate
- **Pitfalls:**
  - **Peeking**: Checking results before reaching required sample size inflates Type I error
  - **Multiple Comparisons**: Testing 20 variants → expect 1 false positive at α=0.05 → use **Bonferroni correction** ($\alpha_{adj} = \alpha / m$)
  - **Novelty Effect**: Users try new features just because they're new — wait for effect to stabilize
  - **Simpson's Paradox**: Aggregate results can reverse when data is segmented

## 5. TOP INTERVIEW QUESTIONS

**Q1 (Easy):** _What's the difference between Type I and Type II error?_
> Type I = False Positive (convicting an innocent person). Type II = False Negative (letting a guilty person go free). α controls Type I. β is Type II. As α ↓, β ↑ (and power ↓) — there's a trade-off.

**Q2 (Medium):** _p-value = 0.03 at α = 0.05. What do you conclude?_
> Reject H₀. The observed data is unlikely under H₀ (only 3% chance of seeing this extreme result if H₀ were true). BUT: check effect size — statistical significance doesn't mean practical significance. A tiny effect can be significant with large n.

**Q3 (Hard):** _You run an A/B test with 1000 users/group. Result is significant (p=0.04) but the lift is 0.1%. What do you recommend?_
> **Don't ship.** Statistically significant ≠ practically significant. 0.1% lift is negligible. With 2000 users, even tiny effects become significant. Check: Is the lift meaningful for business KPIs? What's the cost of implementation? Use **Minimum Detectable Effect (MDE)** in power analysis upfront.

**Q4 (Follow-up):** _"How would you design the A/B test properly?"_
> 1. Define MDE (e.g., 2% lift in conversion)
> 2. Power analysis to determine sample size (need α=0.05, power=0.80, baseline rate, MDE)
> 3. Randomize properly (avoid selection bias)
> 4. Run for full business cycle (capture weekday/weekend effects)
> 5. Don't peek — commit to a sample size upfront
> 6. Use Bonferroni/FDR correction if testing multiple metrics

**Red Flags:**
- Saying "accept H₀" instead of "fail to reject H₀"
- Interpreting p-value as P(H₀ is true)
- Not mentioning effect size alongside significance
- Ignoring multiple comparison correction

## 6. PRACTICAL CODING & CHEAT SHEET

```python
import numpy as np
from scipy import stats

# --- One-Sample t-test ---
sample = np.array([23.5, 25.1, 24.8, 22.9, 26.0, 24.3])
t_stat, p_value = stats.ttest_1samp(sample, popmean=24.0)
print(f"t={t_stat:.3f}, p={p_value:.3f}")
# Reject H₀ if p < 0.05

# --- Two-Sample Independent t-test (Welch's) ---
group_a = np.array([5.2, 5.8, 6.1, 5.5, 5.9])
group_b = np.array([6.5, 6.8, 7.1, 6.3, 6.9])
t_stat, p_value = stats.ttest_ind(group_a, group_b, equal_var=False)
print(f"Welch's t={t_stat:.3f}, p={p_value:.3f}")

# --- Paired t-test ---
before = np.array([120, 135, 128, 140, 132])
after  = np.array([115, 130, 125, 135, 128])
t_stat, p_value = stats.ttest_rel(before, after)
print(f"Paired t={t_stat:.3f}, p={p_value:.3f}")

# --- Chi-Square Test of Independence ---
observed = np.array([[50, 30], [20, 40]])  # 2x2 contingency table
chi2, p, dof, expected = stats.chi2_contingency(observed)
print(f"χ²={chi2:.3f}, p={p:.3f}, dof={dof}")

# --- Power Analysis (sample size calculation) ---
from statsmodels.stats.power import TTestIndPower
analysis = TTestIndPower()
n = analysis.solve_power(effect_size=0.5, alpha=0.05, power=0.8)
print(f"Required n per group: {n:.0f}")  # ~64
```

**5-Bullet Quick Revision:**
1. **p-value** = P(data this extreme | H₀ true). Reject H₀ if p < α. NEVER say "accept H₀".
2. **α** = Type I error rate (false positive). **β** = Type II error rate (false negative). **Power = 1 - β**.
3. **Statistical Significance ≠ Practical Significance** — always report effect size alongside p-value.
4. **Use Welch's t-test** by default (doesn't assume equal variances). Use paired t-test when observations are matched.
5. **Multiple comparisons inflate Type I error** — apply Bonferroni ($\alpha/m$) or FDR (Benjamini-Hochberg) correction.

---

# ═══════════════════════════════════════════════
# TOPIC 6: STATISTICAL SIGNIFICANCE & CONFIDENCE INTERVALS
# ═══════════════════════════════════════════════

## 1. INTUITION & REAL WORLD EXAMPLE (Step-by-Step)

**The Problem:** We almost never know the *exact* true number for an entire population (like the average height of every human on earth, or exactly how an entire country will vote). We can only take a small sample. But samples are noisy.

**The Solution (Confidence Interval):** Instead of guessing one single, highly specific number, we give a **Range** and say how confident we are that the true number lives inside that range.

**The Election Polling Analogy:**
Imagine you poll 1,000 people and find that 52% of them will vote for Candidate A.
You don't confidently say "Candidate A will get exactly 52% on election day." You say: 
*"Candidate A will get 52%, **plus or minus 3%**."*
*   That "plus or minus 3%" is called the **Margin of Error**.
*   The final range (49% to 55%) is your **Confidence Interval (CI)**.

**What a "95% Confidence Interval" Actually Means (Highly Tested!):**
- ✅ It means our *process* is reliable. If we ran this exact same poll 100 different times, about 95 of those polls would successfully trap the *true* final election result inside their generated ranges.
- ❌ It does NOT mean "there is a 95% probability the true value is in this specific interval" (a very common trap in interviews).

## 2. THE MATHEMATICS (For Completeness)

**Confidence Interval (CI) Formula:**
$$CI = \bar{X} \pm z_{\alpha/2} \cdot \frac{\sigma}{\sqrt{n}}$$
*(In plain English: The Range = Your Sample Average $\pm$ Margin of Error)*

**Relationship to Hypothesis Testing:**
- If your Null Hypothesis ($H_0$) guess falls **outside** the Confidence Interval → Reject $H_0$. (Your guess is too far away to be reasonable).
- If it falls **inside** the CI → Fail to reject $H_0$. (Your guess is within the reasonable range).

**Calculating Margin of Error:**
$$ME = z_{\alpha/2} \cdot \frac{\sigma}{\sqrt{n}}$$

| Confidence Level | $z_{\alpha/2}$ |
|---|---|
| 90% | 1.645 |
| 95% | 1.96 |
| 99% | 2.576 |

**Factors Affecting CI Width:**
- ↑ Confidence level → wider CI (more certainty = wider net)
- ↑ Sample size $n$ → narrower CI (more data = more precise)
- ↑ Variability $\sigma$ → wider CI (noisy data = less precision)

---

# ═══════════════════════════════════════════════
# TOPIC 7: CENTRAL LIMIT THEOREM (CLT)
# ═══════════════════════════════════════════════

## 1. INTUITION & REAL WORLD EXAMPLE (Step-by-Step)

> **CLT is the single most important rule in statistics. It is the reason any of this works.**

**The Problem:** In the real world, data is messy. People's incomes are wildly skewed, website clicks are random, and things rarely look like a perfect, symmetric "Bell Curve" (Normal Distribution). The problem is, all our easy math tools (like Z-tests and t-tests) *require* a Bell Curve to work! 

**The Solution (CLT):** The Central Limit Theorem is a mathematical miracle that turns messy, ugly data into a perfect Bell Curve, *as long as you take averages of large enough samples*.

**The Dice Rolling Analogy:**
1.  **Roll 1 Die:** You have an equal 1/6 chance of rolling a 1, 2, 3, 4, 5, or 6. If you graph this, it looks like a flat, boring rectangle. It is completely flat, NOT a Bell Curve.
2.  **Roll 100 Dice and take the Average:** If you roll 100 dice and average the result, it will almost *always* be exactly around 3.5. Getting an average of 1.0 (which would mean rolling 100 ones in a row) is practically impossible. 
3.  **The Result:** If you graph these *averages*, they suddenly create a perfect, beautiful Bell Curve peaking exactly at 3.5!

**Why It Matters:**
Because of the CLT, we don't care if the original population data is heavily skewed, completely flat, or totally weird. As long as we collect a large enough sample (usually $n \geq 30$) and look at the *average*, that average will follow a Bell Curve. This allows us to use Z-tests, t-tests, and Confidence Intervals safely on any real-world data.

## 2. THE MATHEMATICS (For Completeness)

**Statement:** Regardless of the population distribution, the sampling distribution of the sample mean $\bar{X}$ approaches a Normal distribution as $n \to \infty$:
$$\bar{X} \sim N\left(\mu, \frac{\sigma^2}{n}\right) \quad \text{for large } n$$

**Conditions for it to work:**
1. Samples must be **independent** (one dice roll doesn't affect the next).
2. Sample size is **sufficiently large** ($n \geq 30$ is the golden rule).
3. Population has a **finite variance**.

```
Population: Skewed Right          Sampling Distribution of Averages (n=30+)
    ╱╲                                      ╱╲
   ╱  ╲___                               ╱    ╲
  ╱       ╲                             ╱        ╲
 ╱_________╲                          ╱____________╲
                     CLT →            Beautiful, Perfect Bell Curve!
```

## 2. TOP INTERVIEW QUESTIONS

**Q1:** _Can CLT be applied to sample medians?_
> Not directly. CLT specifically applies to **sample means** (and sums). For medians, you'd use bootstrap methods or order statistics.

**Q2:** _Your data is Bernoulli (0/1). Can you use CLT?_
> Yes! Sample proportion $\hat{p} = \bar{X}$ for Bernoulli data. By CLT, $\hat{p} \sim N(p, \frac{p(1-p)}{n})$ for large $n$. Rule of thumb: $np \geq 10$ and $n(1-p) \geq 10$.

---

# ═══════════════════════════════════════════════
# TOPIC 8: PAIRED MEANS TESTS
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

**When to Use:** Observations are **naturally paired** (before/after, matched subjects, left/right eye).

**Key Idea:** Reduce to a one-sample problem by computing differences $d_i = X_{i,\text{after}} - X_{i,\text{before}}$, then test if $\bar{d} = 0$.

$$t = \frac{\bar{d} - 0}{s_d / \sqrt{n}}, \quad df = n - 1$$

Where $\bar{d}$ = mean of differences, $s_d$ = std dev of differences, $n$ = number of pairs.

**Paired vs. Independent:**

| Feature | Paired t-test | Independent t-test |
|---|---|---|
| Design | Same subjects measured twice | Different subjects in each group |
| Data structure | Differences $d_i$ | Two separate samples |
| Power | **Higher** (removes between-subject variability) | Lower |
| Example | Blood pressure before/after drug | Drug group vs. placebo group |

## 2. TOP INTERVIEW QUESTIONS

**Q1:** _A drug trial measures blood pressure before and after treatment for the same 20 patients. Which test?_
> **Paired t-test** — same subjects measured twice. Computing differences removes inter-patient variability, increasing power.

**Q2:** _When should you NOT use a paired t-test?_
> When observations are independent (different subjects), when differences are heavily non-normal with small n (use Wilcoxon signed-rank instead), or when you have more than 2 time points (use repeated measures ANOVA).

---

# ═══════════════════════════════════════════════
# TOPIC 9: BAYES' THEOREM
# ═══════════════════════════════════════════════

## 1. CORE MECHANICS & INTUITION

$$P(A|B) = \frac{P(B|A) \cdot P(A)}{P(B)}$$

**In ML/Stats Language:**
$$\text{Posterior} = \frac{\text{Likelihood} \times \text{Prior}}{\text{Evidence}}$$

| Term | Symbol | Meaning |
|---|---|---|
| **Prior** | $P(A)$ | Belief before seeing data |
| **Likelihood** | $P(B \mid A)$ | How likely is this data given hypothesis A |
| **Evidence (Marginal)** | $P(B)$ | Total probability of data across all hypotheses |
| **Posterior** | $P(A \mid B)$ | Updated belief after seeing data |

**Intuition:** Bayes' updates your belief. Strong prior + weak evidence → posterior ≈ prior. Weak prior + strong evidence → posterior ≈ likelihood.

**Classic Application — Medical Testing (Revisited):**
```
Population: 10,000 people, 1% disease rate (100 sick, 9900 healthy)

Test: 99% Sensitivity (TPR), 95% Specificity (TNR)

                    Test +      Test -
Sick (100)           99           1        (Sensitivity = 99/100)
Healthy (9900)      495        9405        (Specificity = 9405/9900)
                   ─────
Total Test +:       594

P(Sick | Test +) = 99 / 594 = 16.7%   ← ONLY 16.7%!
```

**Why so low?** The **base rate** (prior) is tiny (1%). Even with a great test, the false positives from the huge healthy group overwhelm the true positives.

## 2. MATHEMATICS THAT MATTERS

**Extended Form (multiple hypotheses):**
$$P(A_i|B) = \frac{P(B|A_i) P(A_i)}{\sum_j P(B|A_j) P(A_j)}$$

**Connection to ML:**
- **Naive Bayes Classifier:** Assumes feature independence → $P(C|x_1,...,x_n) \propto P(C) \prod_i P(x_i|C)$
- **MAP (Maximum A Posteriori):** Find $\theta$ that maximizes $P(\theta|D) \propto P(D|\theta) P(\theta)$
- **MLE (Maximum Likelihood):** Special case of MAP with uniform prior → just maximize $P(D|\theta)$

## 3. TOP INTERVIEW QUESTIONS

**Q1:** _What's the difference between MLE and MAP?_
> MLE maximizes $P(D|\theta)$ — just the likelihood, no prior. MAP maximizes $P(\theta|D) \propto P(D|\theta) \cdot P(\theta)$ — includes a prior, which acts as **regularization**. MAP with Gaussian prior → equivalent to L2 regularization. MAP with Laplace prior → equivalent to L1 regularization.

**Q2:** _When would Bayesian methods outperform frequentist?_
> Small data (strong prior helps), online learning (natural sequential updating), hierarchical models, when prior knowledge is available and informative.

---

# ═══════════════════════════════════════════════
# TOPIC 10: LINEAR ALGEBRA (SVD, Eigenvalue Decomposition, Matrix Ops)
# ═══════════════════════════════════════════════

## 1. INTUITION & REAL WORLD EXAMPLES (Step-by-Step)

Before looking at formulas, you need to understand what a "Matrix" actually does geometrically.
**A Matrix is just a machine that squishes, stretches, or rotates space.**
If you put a set of data points into a matrix, they come out looking stretched or rotated.

### Eigenvectors and Eigenvalues (The Rubber Sheet Analogy)
Imagine drawing many arrows on a rubber sheet, and then stretching the sheet from the edges.
Most arrows will change the direction they are pointing.
*   But a few special arrows will keep pointing in the **exact same direction**—they just get longer or shorter.
*   That special arrow is the **Eigenvector** (the direction that didn't change).
*   How much it stretched (e.g., it got 2x longer) is the **Eigenvalue**.

**Why do we care in ML? (PCA)**
In Machine Learning (like PCA), we have huge datasets with hundreds of features (columns). We use Eigenvectors to find the "directions" where the data is stretched the most (which means it has the most variance/information). This lets us compress 100 columns down to just the 5 most important ones without losing the core information!

### Singular Value Decomposition (SVD) - The "Summarizer"
SVD is like a summarizing machine for massive data.
Imagine you have a giant spreadsheet (a Matrix) showing how 1,000,000 users rated 10,000 movies. It's massive and mostly empty.
SVD takes that giant matrix and splits it into 3 smaller, concentrated matrices:
1.  **User Profiles ($U$):** Groups users into underlying concepts (e.g., "Action fans", "Rom-Com fans").
2.  **Importance ($\Sigma$):** Tells you which profile is the most dominant/important overall.
3.  **Movie Profiles ($V^T$):** Groups movies into those same underlying concepts.

By only keeping the top profiles (called **Truncated SVD**), you compress gigabytes of messy data into a clean, small summary. This is exactly how early Netflix recommendation systems worked!

---

## 1.5. THE MATHEMATICS (For Completeness)

### Eigenvalue Decomposition

**For square matrix $A$:**
$$A \mathbf{v} = \lambda \mathbf{v}$$
- $\mathbf{v}$ = eigenvector (direction that doesn't change under transformation, only stretches)
- $\lambda$ = eigenvalue (how much it stretches)

$$A = V \Lambda V^{-1}$$
- $V$ = matrix of eigenvectors (columns)
- $\Lambda$ = diagonal matrix of eigenvalues

**In ML:**
- **PCA**: Eigenvectors of covariance matrix = principal components. Eigenvalues = variance explained.
- **PageRank**: Dominant eigenvector of link matrix.

### Singular Value Decomposition (SVD)

**Works for ANY matrix $A$ (m×n):**
$$A = U \Sigma V^T$$

| Component | Size | Meaning |
|---|---|---|
| $U$ | $m \times m$ | Left singular vectors (orthonormal) — row space info |
| $\Sigma$ | $m \times n$ | Diagonal matrix of singular values $\sigma_i$ (≥ 0, sorted descending) |
| $V^T$ | $n \times n$ | Right singular vectors (orthonormal) — column space info |

**Truncated SVD (Rank-$k$ Approximation):**
$$A \approx U_k \Sigma_k V_k^T$$
- Keep only top-$k$ singular values → **best rank-$k$ approximation**
- Used in: **LSA/LSI** (NLP), **image compression**, **recommendation systems** (matrix factorization)

**Relationship Between SVD and Eigen Decomposition:**
- **Singular values = square roots of eigenvalues of $A^T A$**

## 2. KEY MATRIX OPERATIONS FOR ML

| Operation | Notation | Key Property | ML Use Case |
|---|---|---|---|
| **Transpose** | $A^T$ | $(AB)^T = B^T A^T$ | Feature matrix manipulation |
| **Inverse** | $A^{-1}$ | $A A^{-1} = I$ | OLS: $\hat{\beta} = (X^TX)^{-1}X^Ty$ |
| **Determinant** | $\det(A)$ | $=0$ → singular (no inverse) | Multicollinearity check |
| **Trace** | $\text{tr}(A) = \sum \lambda_i$ | Sum of eigenvalues | PCA total variance |
| **Rank** | $\text{rank}(A)$ | # independent rows/columns | Matrix factorization dimensionality |
| **Dot Product** | $\mathbf{a} \cdot \mathbf{b} = \sum a_i b_i$ | Measures similarity | Cosine similarity in NLP |

**Positive Definite Matrix (important for ML):**
- $\mathbf{x}^T A \mathbf{x} > 0$ for all non-zero $\mathbf{x}$
- All eigenvalues > 0
- **Covariance matrices are always positive semi-definite**
- Needed for: kernel methods, Gaussian distributions, optimization (Hessian)

## 3. TOP INTERVIEW QUESTIONS

**Q1:** _What is the geometric interpretation of eigenvalues in PCA?_
> Eigenvalues of the covariance matrix represent the **variance explained** along each principal component (eigenvector). The largest eigenvalue = direction of maximum variance. PCA sorts eigenvectors by eigenvalues and keeps top-k to capture most variance.

**Q2:** _Why use SVD instead of eigendecomposition for PCA?_
> SVD works on the data matrix directly (any shape), is more numerically stable, and avoids computing $X^TX$ (which can amplify numerical errors). Eigendecomposition requires a square matrix.

**Q3:** _How does matrix factorization work in recommendation systems?_
> User-Item rating matrix $R$ (m×n, sparse) ≈ $U \cdot V^T$ where $U$ (m×k) = user latent factors, $V$ (n×k) = item latent factors. Train by minimizing $\|R - UV^T\|^2 + \lambda(\|U\|^2 + \|V\|^2)$. Missing entries are predicted as $\hat{r}_{ui} = \mathbf{u}_i^T \mathbf{v}_j$.

## 4. PRACTICAL CODING

```python
import numpy as np
from sklearn.decomposition import PCA, TruncatedSVD

# --- Eigendecomposition ---
A = np.array([[4, 2], [1, 3]])
eigenvalues, eigenvectors = np.linalg.eig(A)
print(f"Eigenvalues: {eigenvalues}")     # [5, 2]
print(f"Eigenvectors:\n{eigenvectors}")

# --- SVD ---
A = np.random.randn(5, 3)
U, S, Vt = np.linalg.svd(A, full_matrices=False)
# Reconstruct: A_reconstructed = U @ np.diag(S) @ Vt

# --- PCA using Sklearn ---
from sklearn.datasets import load_iris
X = load_iris().data
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X)
print(f"Explained variance ratio: {pca.explained_variance_ratio_}")
# e.g., [0.924, 0.053] → first 2 PCs capture 97.7%

# --- Truncated SVD (for sparse matrices, e.g., NLP) ---
from sklearn.feature_extraction.text import TfidfVectorizer
docs = ["machine learning is great", "deep learning is powerful", "NLP uses embeddings"]
tfidf = TfidfVectorizer()
X_tfidf = tfidf.fit_transform(docs)
svd = TruncatedSVD(n_components=2)
X_lsa = svd.fit_transform(X_tfidf)  # Latent Semantic Analysis
```

**5-Bullet Quick Revision:**
1. **Eigenvectors** = directions preserved under transformation; **eigenvalues** = scaling factors. PCA uses eigenvalues of covariance matrix.
2. **SVD** = $U\Sigma V^T$ works for any matrix. Truncated SVD = best rank-k approximation (lossy compression).
3. $\hat{\beta}_{\text{OLS}} = (X^TX)^{-1}X^Ty$ — fails when $X^TX$ is singular (**multicollinearity** → use regularization or pseudo-inverse).
4. **Singular values** = $\sqrt{\text{eigenvalues of } A^TA}$. They quantify "importance" of each component.
5. **Positive semi-definite** matrices (all eigenvalues ≥ 0) include all covariance matrices and kernel matrices — critical for valid ML models.

---

# PSSP MASTER REVISION TABLE

| Topic | Key Formula | #1 Interview Trap | Quick Fact |
|---|---|---|---|
| Probability | $P(A \mid B) = P(A∩B)/P(B)$ | Independent ≠ Mutually Exclusive | Use complement for "at least one" |
| Distributions | Normal: 68-95-99.7 | Skewness: +ve → Mean > Median | Binomial → Poisson when n→∞, p→0 |
| Sampling | $SE = \sigma/\sqrt{n}$ | Stratified: sample FROM each. Cluster: sample ENTIRE clusters | 4× sample size → halves SE |
| Correlation | Pearson measures LINEAR only | r=0 doesn't mean no relationship | Spearman for ordinal/outliers |
| **Hypothesis Testing** | **t = (x̄ - μ₀)/(s/√n)** | **NEVER say "accept H₀"** | **p < α → Reject H₀** |
| CI | x̄ ± z·(σ/√n) | 95% CI ≠ "95% prob true value is here" | CI and Hypothesis test are dual |
| CLT | x̄ → N(μ, σ²/n) | Applies to means, NOT medians | n ≥ 30 rule of thumb |
| Paired Test | t = d̄/(s_d/√n) | Use when SAME subjects measured twice | Higher power than independent |
| Bayes | Post ∝ Likelihood × Prior | P(A\|B) ≠ P(B\|A) | MAP with Gaussian prior = L2 reg |
| Linear Algebra | SVD: A = UΣVᵀ | SVD works for any matrix, Eigen only square | PCA eigenvalues = variance explained |
