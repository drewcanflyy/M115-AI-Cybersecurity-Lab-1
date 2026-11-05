# 🛡️ Executive Lab 1 Study & Review Guide (English)
**M115: AI & Intelligent Systems — Master SSIC (Cyber Defense)**  
**ENSIASD Taroudant — Ibn Zohr University**  
**Dataset:** PhiUSIIL Phishing URL Dataset (235,795 URLs, 56 Features)

---

## 🧭 Executive Summary of All Lab Exercises

### 📍 PART 1: Security-Oriented Exploratory Data Analysis (EDA)

#### 🔹 Exercise 1: SOC Target Alignment & Data Quality
* **Concept:** In the raw dataset, `label = 1` was Legitimate and `label = 0` was Phishing.
* **Why invert it?** In a Security Operations Center (SOC), the positive class (`1`) **must represent the threat (Attack)** so that standard metrics (Recall, Precision, True Positives) accurately measure detection capabilities rather than benign traffic detection.
* **Code:** `df['IsPhishing'] = 1 - df['label']`
* **Finding:** The dataset is well-balanced: **42.81% Phishing** (100,945 URLs) and **57.19% Legitimate** (134,850 URLs).

#### 🔹 Exercise 2: Binary Security Indicators & Lexical Distributions
* **`IsHTTPS` Trap:** 100% of benign sites use HTTPS, but **~49.22% of phishing sites also use HTTPS** (via free Let's Encrypt / Cloudflare certificates or Firebase/Vercel hosting). *Conclusion:* A static rule "If HTTPS then Safe" would miss half of all phishing attacks!
* **`HasSocialNet` & `HasCopyrightInfo`:** Strong benign indicators (~80% in legitimate sites, <6% in phishing). Attackers do not maintain social media presence or copyright legalities for throwaway 24h landing pages.
* **`NoOfSelfRef` (Self-References):** Legitimate sites have dozens/hundreds of internal links (`mean ≈ 113`). Over 90% of phishing sites have `NoOfSelfRef = 0` because they are simple, single-page credential harvesting clones.
* **`URLLength`:** Phishing URLs are significantly longer with high character continuation and query token obfuscation.

#### 🔹 Exercise 3: Identifier Cleanup & Pearson Correlations
* **Concept:** Dropping `FILENAME`, `URL`, `Domain`, `TLD`, and `Title`.
* **Why drop domain strings?** If raw domain names are kept, the model suffers from **overfitting through memorization** (acting as a static blacklist). When deployed against zero-day phishing infrastructure with newly registered domains, the model fails to generalize.
* **Top Correlations ($r$):**
  * *Negative ($r < 0$, Benign indicators):* `HasSocialNet` (-0.78), `HasCopyrightInfo` (-0.74), `HasDescription` (-0.69).
  * *Positive ($r > 0$, Phishing indicators):* `SpacialCharRatioInURL` (+0.53), `DegitRatioInURL` (+0.43).

---

### 📍 PART 2: Zero-Data Leakage Preprocessing & Standardization

#### 🔹 Exercise 4: Stratified Split & StandardScaler (SecMLOps Rule)
* **Stratified 75/25 Split:** Guarantees that both training (176,846 samples) and testing (58,949 samples) preserve the exact 42.8% phishing class ratio.
* **Why Standardization? (The Scale Problem):** `LineOfCode` ranges into 10,000+ while `IsHTTPS` is binary (0 or 1). Without scaling, L1/L2 penalties would severely penalize small-scale binary features (forcing their weights to zero) while ignoring large-scale features. `StandardScaler` standardizes each feature to $\mu = 0, \sigma = 1$.
* **Zero-Data Leakage:** We call `.fit_transform()` **only on Train**, and `.transform()` **on Test**. Fitting the scaler on the entire dataset prior to splitting leaks distributional properties of the test set into the training phase.

---

### 📍 PART 3: Parameterized Logistic Regression & Interpretability

#### 🔹 Exercise 5: Comparing 5 Regularization Configurations
* **M1 (Unregularized / None):** Fits data without penalty. Highly prone to weight explosion and instability under adversarial perturbations.
* **M2 (Strong L2 / C=0.001):** Excessive shrinkage penalty; overly conservative, misses 105 phishing attacks (high False Negatives).
* **M3 (Standard L2 / C=1.0):** Optimal balanced model; robust weights, low false alarms (10 FP), excellent F1 (0.9992).
* **M4 (Sparse L1 Lasso / C=0.01) ⭐ Edge Firewall Champion:**
  * L1 penalty forces uninformative weights to **exactly zero ($w_j = 0.0000$)**.
  * **Result:** Eliminates ~24 features (50% reduction) while maintaining F1 = 0.9982.
  * **Firewall Advantage:** Halves packet inspection latency in real-time edge filtering.
* **M5 (SOC Weighted / balanced):** Penalizes missed attacks more heavily, lowering FN from 28 to 25 at the expense of slightly higher FP (13).

#### 🔹 Exercise 6: L1 Regularization Path
* Demonstrates how changing inverse regularization strength $C$ from $0.002$ to $5.0$ controls sparsity:
  * At small $C=0.002$ (severe penalty), only **13 core features** survive.
  * At large $C=5.0$ (relaxed penalty), almost all **49 features** remain active.

#### 🔹 Exercise 7: Log-Odds ($w_j$) and Odds Ratios ($e^{w_j}$)
* **Odds Ratio $e^{w_j}$:** Multiplicative factor by which the odds of phishing multiply per unit increase in the standardized feature.
  * $w_j > 0 \implies e^{w_j} > 1$: Pushes toward Phishing (e.g., `SpacialCharRatioInURL`, `LetterRatioInURL`).
  * $w_j < 0 \implies e^{w_j} < 1$: Pushes toward Legitimate (e.g., `HasSocialNet`, `HasCopyrightInfo`).

---

### 📍 PART 4: Explainable AI (SHAP) & SOC Threshold Analysis

#### 🔹 Exercise 8: Global SHAP Explanations (`shap.LinearExplainer`)
* Calculates Shapley values ($\phi_j$) derived from cooperative game theory.
* **Bar Plot:** Ranks global mean absolute impact $\text{mean}(|\phi_j|)$.
* **Beeswarm Plot:** Shows directional contribution (e.g., high special char ratio points red/right toward phishing; high social links points red/left toward benign).

#### 🔹 Exercise 9: Local SOC Alert Triage (Waterfall Plots)
* Decomposes individual predictions: $f(x) = E[f(X)] + \sum \phi_j(x)$.
* **True Positive (TP):** Phishing site correctly caught; positive SHAP contributions from lack of copyright and high digit ratio.
* **True Negative (TN):** Benign site safely passed; strong negative contributions from social media links and high self-references.
* **False Positive (FP - False Alarm):** Legitimate site wrongly flagged due to long URL parameters and tracking tokens mimicking phishing patterns.

#### 🔹 Exercise 10: Operational Decision Threshold Tuning ($\tau$)
* **$\tau = 0.20$ (Threat Hunting Mode):** Maximizes Recall (catches stealthy attacks, minimizes missed phish, but yields higher false alarms).
* **$\tau = 0.50$ (Default Mode):** Standard balanced threshold.
* **$\tau = 0.85$ (Automated Inline Blocking Mode):** Maximizes Precision and eliminates False Positives (ensures no legitimate business services or client portals are accidentally blocked).

---

## 🛡️ Recommended SOC Dual-Threshold Architecture
1. **$P(\text{Phishing}) \ge 0.85$:** Instant automated inline firewall / DNS block (zero business disruption).
2. **$0.20 \le P(\text{Phishing}) < 0.85$:** Route to human Tier-1 SOC analyst queue with SHAP waterfall explanations for rapid triage.
3. **$P(\text{Phishing}) < 0.20$:** Automatically allow and log.

