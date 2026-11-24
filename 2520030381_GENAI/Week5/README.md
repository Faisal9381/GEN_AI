# Week 5: Token Economics & Feature Budgeting

Welcome to **Week 5**! 👋 In this lab, we tackle the financial and infrastructure engineering side of Generative AI: **Token Economics and Capacity Planning**.

Building an AI feature is not just about writing good prompts — before deploying to production, you must answer:
- *How much will this feature cost per day and per month?*
- *Will our peak traffic exceed the model's rate limits (RPM / TPM)?*
- *Do we have enough safety margin for traffic spikes?*

This week's code provides a clean, automated **Feature Budget Calculator** that answers these questions.

---

## 📖 Table of Contents
- [What You Will Learn](#-what-you-will-learn)
- [Project Structure](#-project-structure)
- [Formula Reference: How Costs & Limits are Calculated](#-formula-reference-how-costs--limits-are-calculated)
- [Step-by-Step Setup](#-step-by-step-setup)
- [How to Run](#-how-to-run)
- [Running Unit Tests](#-running-unit-tests)
- [Key Metrics Explained](#-key-metrics-explained)
- [Configuration Reference (.env)](#-configuration-reference-env)

---

## 🎯 What You Will Learn
1. **Token Cost Modeling**: Calculating API expenses from input tokens, output tokens, and per-million token pricing.
2. **Rate Limits & Capacity Planning**:
   - **RPM (Requests Per Minute)**: How many queries your app sends per minute.
   - **TPM (Tokens Per Minute)**: Total volume of input + output tokens sent/received per minute.
3. **Safety Margins**: Accounting for real-world traffic variance and prompt length fluctuations (e.g., +20% buffer).
4. **Automated Budget Guardrails**: Programmatically verifying if your design is within cost and provider quota limits before building.

---

## 📂 Project Structure

```text
Week5/
├── budget.py           # FeatureBudget dataclass & financial/capacity calculation logic
├── test_budget.py      # Automated unit tests checking calculations and pass/fail thresholds
├── requirements.txt    # Python dependencies (python-dotenv)
├── .env                # Configurable assumptions (traffic, token sizes, pricing, quotas)
└── README.md           # This guide
```

---

## 📐 Formula Reference: How Costs & Limits are Calculated

### 1. Daily Token Consumption
$$\text{Total Daily Input Tokens} = \text{Requests/Day} \times \text{Avg Input Tokens}$$
$$\text{Total Daily Output Tokens} = \text{Requests/Day} \times \text{Avg Output Tokens}$$

### 2. Daily & Monthly Cost (USD)
$$\text{Daily Cost} = \frac{(\text{Daily Input Tokens} \times \text{Price}_{\text{in}}) + (\text{Daily Output Tokens} \times \text{Price}_{\text{out}})}{1,000,000}$$
$$\text{Monthly Cost} = \text{Daily Cost} \times 30$$

### 3. Peak Capacity with Safety Margin
$$\text{Margin Multiplier} = 1 + \frac{\text{Safety Margin \%}}{100}$$
$$\text{Peak RPM} = \text{Peak Requests/Min} \times \text{Margin Multiplier}$$
$$\text{Peak TPM} = \text{Peak Requests/Min} \times (\text{Avg Input Tokens} + \text{Avg Output Tokens}) \times \text{Margin Multiplier}$$

---

## 🚀 Step-by-Step Setup

### Step 1: Open Terminal in Week 5 Folder
Navigate into the `Week5` directory:
```bash
cd "2520030381_GENAI/Week5"
```

### Step 2: Create and Activate a Virtual Environment
- **Windows (PowerShell):**
  ```powershell
  python -m venv .venv
  .venv\Scripts\Activate.ps1
  ```
- **macOS / Linux:**
  ```bash
  python3 -m venv .venv
  source .venv/bin/activate
  ```

### Step 3: Install Required Packages
```bash
pip install -r requirements.txt
```

### Step 4: Configure the `.env` File
Create or update your `.env` file with your project's traffic and pricing assumptions:

```ini
# Daily Usage Assumptions
REQUESTS_PER_DAY=1000
AVG_INPUT_TOKENS=800
AVG_OUTPUT_TOKENS=300
PEAK_REQUESTS_PER_MINUTE=25

# Model Pricing per 1 Million Tokens (USD)
INPUT_PRICE_PER_MILLION=0.15
OUTPUT_PRICE_PER_MILLION=0.60

# Provider Quotas & Rate Limits
RPM_LIMIT=60
TPM_LIMIT=100000

# Budget Constraints (USD)
DAILY_BUDGET_USD=5.00
MONTHLY_BUDGET_USD=100.00

# Safety Margin Buffer (in percentage)
SAFETY_MARGIN_PERCENT=20
```

---

## ▶️ How to Run

Run the budget calculation script:
```bash
python budget.py
```

### Example Terminal Output:
```text
TOKEN ECONOMICS REPORT
-----------------------------------
Input tokens/day:  800,000
Output tokens/day: 300,000
Total tokens/day:  1,100,000
Estimated daily cost:   $0.3000
Estimated monthly cost: $9.00
Peak RPM with margin: 30.0
Peak TPM with margin: 39,600

CHECKS
RPM capacity: PASS
TPM capacity: PASS
Daily budget: PASS
Monthly budget: PASS

Recommendation: WITHIN configured planning limits.
```
*If any check exceeds your budget or provider quota, the script flags it as `REVIEW` and warns you before development.*

---

## 🧪 Running Unit Tests

Run the test suite to verify calculation accuracy and guardrail checks:

```bash
python test_budget.py
```
*or using unittest runner:*
```bash
python -m unittest test_budget.py -v
```

Expected output:
```text
test_capacity_passes (test_budget.TestFeatureBudget) ... ok
test_cost_estimate (test_budget.TestFeatureBudget) ... ok
test_monthly_budget_fails (test_budget.TestFeatureBudget) ... ok
test_rpm_limit_fails (test_budget.TestFeatureBudget) ... ok

----------------------------------------------------------------------
Ran 4 tests in 0.001s

OK
```

---

## 💡 Key Metrics Explained

| Term | What it Means | Why it Matters |
| :--- | :--- | :--- |
| **Input Tokens** | Words sent to the model (system prompt + user input + history). | Usually cheaper per token than output tokens. |
| **Output Tokens** | Words generated by the LLM in response. | More computationally expensive; usually priced higher. |
| **RPM (Requests Per Min)** | How many API calls hit the server within 60 seconds. | Cloud providers throttle or block you if you exceed your RPM tier. |
| **TPM (Tokens Per Min)** | Total volume of tokens processed in 60 seconds. | Prevents sending huge bulk prompts all at once. |
| **Safety Margin** | A buffer percentage (e.g. 20%) added to peak traffic. | Protects your production app from crashing during unexpected traffic surges. |
