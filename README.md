# Risk vs Pricing Analysis: Does the Interest Rate Pay for the Credit Risk? (SQL)

## 📊 Project Overview
This project analyses over 2 million peer-to-peer loans from the Lending Club dataset (2007–2018) to test whether the interest rate charged on each grade actually compensated the lender for the credit risk taken. Instead of comparing an annual interest rate with a lifetime default rate (two numbers in different units), it measures what lenders really received in cash, annualised on the balance outstanding, and compares that with the rate promised, the cost of funds and the losses of a bad year. The analysis follows the actuarial pricing logic in Coleman's *A Practical Guide to Risk Management* and the credit triangle from the CQF credit risk module, implemented entirely in SQL.

## 🎯 Objective
To evaluate pricing adequacy the way a lender's credit risk and finance teams would: checking that the pricing grid rises with risk, measuring how much of the promised yield is lost to defaults, testing profitability after funding and servicing costs, assessing whether margins survive a stressed year, identifying mispricing inside grades, and comparing the default rate implied by the price with the default rate actually observed, presented as a portfolio-ready case study.

## 🧰 Tools Used
- SQL (SQLite via DB Browser for SQLite): CTEs, window functions (`LAG`), helper and assumption tables, conditional aggregation
- GitHub (project documentation)
- Dataset: [Lending Club Loan Data 2007–2018](https://www.kaggle.com/datasets/wordsforthewise/lending-club) via Kaggle


## ⚠️ Why "Interest Rate − Default Rate" Is the Wrong Test
A common shortcut subtracts the default rate from the interest rate. It fails for three reasons:
1. **Different units:** the interest rate is per year, while the default rate is cumulative over the loan's life
2. **Loss severity is ignored:** when a loan defaults, the lender has usually already received some principal and later recovers part of the balance
3. **Costs are ignored:** the lender pays for its funding and for servicing the loans

This analysis instead compares like with like: **cash actually received versus cash lent, per year of money outstanding**, which is in the same units as the interest rate.

## 📌 Key Business Questions
1. Does the interest rate rise in step with risk across the 35 sub-grades?
2. What return did each grade actually deliver, compared with the rate it promised?
3. How much of the interest income was consumed by credit losses?
4. Is each grade profitable after funding and servicing costs, and how far could losses rise before it breaks even?
5. Does the margin survive a bad year, not just an average one?
6. Within a grade, does the price still respond to risk, or is there mispricing and adverse selection?
7. Did pricing keep pace with risk through the credit cycle?
8. How much default does the price imply, compared with the default actually observed?

## 🔍 Analysis Structure
**File:** `queries/06_risk_vs_pricing.sql`

| Query | Business Question | Technique |
|-------|-------------------|-----------|
| 0 | Setup: cash flows, exposure-time and cost assumptions | Helper table `pricing_base`; editable assumption table `pricing_params` |
| 1 | Does price rise with risk? | Pricing curve by sub-grade; `LAG()` to flag inversions and steep jumps |
| 2 | Promised rate vs realised return | Balance-weighted rate, annualised realised return, yield erosion |
| 3 | Where did the return go? | Return decomposition: interest + fees − credit loss + recoveries |
| 4 | Profitability and break-even headroom | Net margin after costs, loss cover multiple, loss headroom |
| 5 | Does the margin survive a bad year? | Worst-vintage loss, stress buffer, RAROC-style return on stress |
| 6 | Mispricing within grades | Grade × DTI band: rate, default and realised return |
| 7 | Pricing through the cycle | Realised return and margin by vintage |
| 8 | Price-implied vs realised default | Credit triangle inverted; realised default intensity; premium multiple |

## 🧠 Methodology Notes
- **Complete loans only:** a loan is included only if its full term (36 or 60 months) has elapsed and its outcome is final (fully paid or charged off). Comparing pricing with unfinished loans would understate losses
- **Annualised returns:** realised return = (cash received − amount lent) ÷ principal-years, where principal-years = average outstanding balance × years on book. Average balance assumes roughly straight-line repayment, so it is an approximation
- **Cash received** = principal + interest + late fees + recoveries − collection fees
- **Net credit loss** = exposure at default (funded amount − principal repaid) − net recoveries
- **Reconciliation:** the realised return in Query 2 equals the sum of components in Query 3, which serves as an internal consistency check
- **Assumptions:** funding cost (3% p.a.) and servicing cost (1% p.a.) are **illustrative assumptions**, stored in one table so they can be changed to test sensitivity
- **Stress proxy:** the stress buffer in Query 5 is the worst vintage's loss rate minus the average, a simple proxy for unexpected loss rather than a full 99.9% VaR
- **Realised default intensity:** estimated as defaults ÷ loan-years at risk, the standard estimator of a constant default intensity


## 💡 CFA / CQF Concepts Applied
- **Promised vs expected income:** income net of expected loss and the cost of funds (Coleman, *A Practical Guide to Risk Management*)
- **Provisions, reserves and economic capital:** expected loss is priced and provisioned; unexpected loss in a bad year must be covered by capital (Coleman)
- **Actuarial pricing:** a loan is held if the spread earned sufficiently compensates for its risk, using historical (physical) probabilities (Coleman, citing McNeil, Frey and Embrechts)
- **Physical vs risk-neutral probabilities:** price-implied default compared with realised default (Coleman)
- **Credit triangle:** `spread ≈ λ × (1 − RR)`, used forwards for break-even and backwards to infer default from price (CQF M5S8)
- **Constant default intensity:** λ estimated as defaults ÷ time at risk 
- **Expected Loss (PD × LGD × EAD)** and **risk-adjusted return (RAROC-style)**
- **Adverse selection** in flat, grade-level pricing

## ✅ How to Reproduce
1. Download the Lending Club dataset from [Kaggle](https://www.kaggle.com/datasets/wordsforthewise/lending-club)
2. Open **DB Browser for SQLite** → New Database → name it `credit_portfolio.db`
3. Import the CSV: **File → Import → Table from CSV file** (tick "Column names in first line"). The table is named `accepted_2007_to_2018Q4` automatically
4. In the **Execute SQL** tab, run **Query 0** once to build `pricing_base` and `pricing_params`, then click **Write Changes**
5. Highlight one query at a time and press **F5**
6. Optional: change the cost assumptions with `UPDATE pricing_params SET funding_cost_pct = 5.0;` and re-run Queries 4, 5, 7 and 8 to test sensitivity

## 💼 Author
**Hannah (Huong) Tran** |
Financial Analysis |
MSc Banking & Finance with Distinction |
CFA Levels I & II cleared

[LinkedIn](https://www.linkedin.com/in/huong-hannah/) | [GitHub](https://github.com/Hannah-Tran)
