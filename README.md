# 💬 Loan Risk Chatbot — Natural Language to SQL

**Ask questions in plain English. Get answers from 1.34M loan records instantly.**

This chatbot converts natural language questions into SQL queries and runs them against a scored loan database — no SQL knowledge required.

> "Show high risk customers in Texas" → SQL → Results + Insight

Companion project to [loan-risk-pipeline](https://github.com/Pari4113/loan-risk-pipeline) which built and scored the underlying data.

---

## 🎯 What It Does

```
You type a question in plain English
          ↓
LLM (Databricks ai_query) reads the database schema
          ↓
Converts your question to Spark SQL automatically
          ↓
Runs SQL against 1.34M scored loan records
          ↓
Returns results as a table
          ↓
Generates plain English insight from results
```

---

## 💡 Example Questions

| Question | What it returns |
|---|---|
| "Show high risk customers in Texas" | All High/Very High risk loans in TX |
| "Which states have the highest default rates?" | Ranked list of states by default % |
| "What is the average default probability by grade?" | Grade A→G with avg default score |
| "Compare interest rates between defaulted and paid loans" | Side by side avg interest rates |
| "How many loans are in each risk category?" | Count by Low/Medium/High/Very High |
| "Show top 10 loans with highest default probability" | Riskiest loans with all details |

---

## 🏗️ How It Works

### 1. Schema Discovery
The chatbot automatically reads the Gold Delta table schema — all 29 columns, data types, and row count. This tells the LLM exactly what data is available.

### 2. Prompt Engineering
Builds a system prompt with:
- Full table schema
- Column explanations (what each column means)
- SQL rules (always filter clean states, use correct table name etc)
- Business context (what risk_label means, what default means)

### 3. LLM — Databricks ai_query
Uses Databricks built-in `ai_query()` function with `databricks-meta-llama-3-3-70b-instruct` model:
- No API key needed
- No external services
- Runs entirely inside Databricks
- Free on Community Edition

### 4. Query Execution
Generated SQL runs directly against the Delta table using Spark SQL — handles 1.34M rows instantly.

### 5. Insight Generation
Same LLM generates a 2-sentence plain English summary of the results — written for a business audience, not a technical one.

---

## 🗄️ Database

Queries the Gold Delta table from [loan-risk-pipeline](https://github.com/Pari4113/loan-risk-pipeline):

| Table | Rows | Description |
|---|---|---|
| `loan_risk.gold_loans_scored` | 1,345,308 | Cleaned loans with ML risk scores |

**Key columns available to query:**

| Column | Type | Description |
|---|---|---|
| `default_probability` | double | ML model score 0.0-1.0 |
| `risk_label` | string | Low / Medium / High / Very High |
| `grade` | string | A (best) to G (worst) |
| `addr_state` | string | 2-letter US state code |
| `int_rate` | double | Interest rate % |
| `dti` | double | Debt-to-income ratio |
| `loan_amnt` | double | Loan amount in $ |
| `purpose` | string | Why they borrowed |
| `default` | int | 1=defaulted, 0=fully paid |
| `risk_score` | int | Composite risk score 3-13 |

---

## 🔧 Tech Stack

| Component | Technology |
|---|---|
| Platform | Databricks Community Edition |
| LLM | Databricks ai_query (Meta LLaMA 3.3 70B) |
| Data Storage | Delta Lake |
| Query Engine | Spark SQL |
| Language | Python |
| UI | Databricks notebook widget |

---

## 🚀 How to Run

### Prerequisites
- Databricks Community Edition account
- [loan-risk-pipeline](https://github.com/Pari4113/loan-risk-pipeline) completed (creates the Gold table)

### Setup

**1. Clone this repo as a Git folder in Databricks:**
- Go to **Repos → Add Repo**
- Paste: `https://github.com/Pari4113/loan-risk-chatbot`

**2. Make sure Gold table exists:**
```sql
-- Run this in Databricks to verify
SHOW TABLES IN loan_risk;
-- Should show: gold_loans_scored
```

**3. Open `Notebook_4_Chatbot` and click Run All**

**4. Type your question in the widget box at the top and click Run**

---

## 📊 Sample Results

**Question:** "Which states have the highest default rates?"
```
MS  → 26.1%
NE  → 25.2%
AR  → 24.1%
AL  → 23.6%
OK  → 23.5%
```

**Question:** "Average default probability by grade?"
```
G → 87.5%
F → 82.1%
E → 73.4%
D → 61.8%
C → 49.8%
B → 37.2%
A → 25.7%
```

**Question:** "Compare interest rates between defaulted and paid loans?"
```
Defaulted loans  → 15.7% avg interest rate
Fully paid loans → 12.6% avg interest rate
```

---

## 🎯 Key Design Decisions

**Why Databricks ai_query instead of external LLM?**
- No API key or cost
- Data never leaves Databricks — important for financial data privacy
- Runs in the same environment as the data
- Built-in feature worth knowing for Databricks certification

**Why filter `LENGTH(addr_state) = 2`?**
Raw Lending Club data has dirty values in the addr_state column — some rows contain loan descriptions instead of state codes. This filter ensures only valid 2-letter state codes are returned.

**Why auto-discover schema?**
Instead of hardcoding table structure, the chatbot reads the live schema every time it starts. This means if the underlying table changes, the chatbot automatically adapts.

---

## 🔗 Related Projects

- **[loan-risk-pipeline](https://github.com/Pari4113/loan-risk-pipeline)** — ETL pipeline that creates the data this chatbot queries
  - Bronze → Silver → Gold Medallion Architecture
  - 1.34M loans cleaned and ML-scored
  - Default risk insights by state, grade, and purpose

---

## 📝 Notes

This project demonstrates:
- **Prompt engineering** — building effective LLM prompts with schema context
- **Text-to-SQL** — converting natural language to executable queries
- **Delta Lake integration** — querying production-grade Delta tables
- **Databricks ai_query** — using built-in LLM capabilities
- **Data pipeline thinking** — chatbot is built ON TOP of a clean data pipeline

---

**Status:** ✅ Complete
