# 📐 Formulas Breakdown

Detailed breakdown of every formula used across all sheets in the model. All formulas reference the `Assumptions` sheet — no hardcoded values in calculations.

---

## Assumptions Sheet

This is the only sheet where you manually enter values. Every other sheet pulls from here using absolute references like `Assumptions!$B$3`.

| Row | Parameter | Cell | Value |
|---|---|---|---|
| 2 | Initial Customers | B2 | 100 |
| 4 | Monthly Growth Rate | B4 | 6% |
| 5 | Price per Customer | B5 | ₹500 |
| 7 | Variable Cost per Customer | B7 | ₹150 |
| 9 | Fixed Monthly Cost | B9 | ₹20,000 |
| 11 | Marketing Cost | B11 | ₹8,000 |
| 13 | Tax Rate | B13 | 25% |
| 17 | Forecast Period | B17 | 36 Months |
| 19 | Monthly Churn Rate | B19 | 3% |
| 21 | Starting Cash | B21 | ₹5,00,000 |

---

## Customer_Model Sheet

Tracks the customer base month by month applying both growth and churn.

| Column | Header | Formula (Row 3 = Month 1) |
|---|---|---|
| A | Month | `=Revenue_Model!A3` |
| B | Customers Start | `=Assumptions!$B$2` (Month 1 only), then `=E[prev row]` |
| C | New Customers | `=B3 * Assumptions!$B$4` |
| D | Churned Customers | `=B3 * Assumptions!$B$19` |
| E | Customers End | `=B3 + C3 - D3` |

**Row 4 onwards (Month 2+):**
```
B4 = E3                          ← carries forward previous month's end count
C4 = B4 * Assumptions!$B$4      ← new customers = start × growth rate
D4 = B4 * Assumptions!$B$19     ← churn = start × churn rate
E4 = B4 + C4 - D4               ← net customers
```

**Key logic:** Growth and churn are both applied to the **starting** customer count each month, not the ending count. This avoids double-counting within the same period.

---

## Revenue_Model Sheet

Links to Customer_Model for customer counts and multiplies by price.

| Column | Header | Formula |
|---|---|---|
| A | Month | Auto-increments |
| B | Customers | `=Customer_Model!E[prev row]` — pulls ending customers from Customer_Model |
| C | Revenue | `=B3 * Assumptions!$B$5` — customers × price per customer |

**Example:**
```
Month 1:  100 customers × ₹500 = ₹50,000
Month 36: 281 customers × ₹500 = ₹1,40,500
```

---

## Cost_Model Sheet

Breaks costs into three components: variable, fixed, and marketing.

| Column | Header | Formula |
|---|---|---|
| A | Month | `=Revenue_Model!A3` |
| B | Customers | `=Revenue_Model!B3` — mirrors Revenue_Model customer count |
| C | Variable Cost | `=B3 * Assumptions!$B$7` — customers × variable cost per customer |
| D | Fixed Cost | `=Assumptions!$B$9` — constant ₹20,000 every month |
| E | Marketing Cost | `=Assumptions!$B$11` — constant ₹8,000 every month |
| F | Total Cost | `=C3 + D3 + E3` |

**Note:** Fixed and Marketing costs do not scale with customers — they are flat monthly charges.

---

## Model_Calculations Sheet

Core profitability calculations including break-even analysis.

| Column | Header | Formula |
|---|---|---|
| A | Month | Sequence 1–36 |
| B | Revenue | `=Revenue_Model!C3` |
| C | Total Cost | `=Cost_Model!F3` |
| D | Gross Profit | `=B3 - C3` |
| E | Contribution Margin | `=Assumptions!$B$5 - Assumptions!$B$7` → ₹500 − ₹150 = **₹350** (static) |
| F | Break-Even Customers | `=(Assumptions!$B$9 + Assumptions!$B$11) / E3` → ₹28,000 / ₹350 = **~57 customers** |
| G | Cumulative Profit | `=G2 + D3` — running total of gross profit |

**Break-Even logic:**
```
Contribution Margin  = Price − Variable Cost = ₹500 − ₹150 = ₹350
Break-Even Customers = (Fixed Cost + Marketing) / Contribution Margin
                     = ₹28,000 / ₹350
                     = 57.14 customers
```
Since Month 1 starts at 100 customers, the business is **profitable from Day 1**.

---

## Unit_Economics Sheet

Calculates the three most important SaaS/startup metrics.

| Row | Metric | Formula | Result |
|---|---|---|---|
| 3 | Marketing Spend | `=Assumptions!$B$11` | ₹8,000 |
| 4 | Avg New Customers | `=AVERAGE(Customer_Model!C3:C38)` | ~11 |
| 5 | CAC | `=C3 / C4` | ~₹727 |
| 6 | LTV | `=Assumptions!$B$5 / Assumptions!$B$19` | ₹16,667 |
| 7 | LTV/CAC Ratio | `=C6 / C5` | ~22.9x |

**Formulas explained:**
```
CAC   = Monthly Marketing Spend / Avg New Customers per Month
      = ₹8,000 / 11 = ₹727

LTV   = Price per Customer / Monthly Churn Rate
      = ₹500 / 0.03 = ₹16,667

LTV/CAC = ₹16,667 / ₹727 = 22.9x
```

**Benchmark:** LTV/CAC above 3x is generally considered healthy. At 22.9x this model shows strong unit economics.

---

## Profit_Loss Sheet

Monthly P&L applying tax to arrive at net profit.

| Column | Header | Formula |
|---|---|---|
| A | Month | Sequence 1–36 |
| B | Revenue | `=Revenue_Model!C3` |
| C | Total Cost | `=Cost_Model!F3` |
| D | Profit Before Tax | `=B3 - C3` |
| E | Tax | `=D3 * Assumptions!$B$13` — 25% of pre-tax profit |
| F | Net Profit | `=D3 - E3` |

**Note:** Tax is calculated monthly. In practice, tax is assessed annually — but this approach gives a conservative monthly view.

---

## Cash_Flow Sheet

Tracks actual cash position starting from the initial capital.

| Column | Header | Formula |
|---|---|---|
| A | Month | Sequence 1–36 |
| B | Revenue | `=Revenue_Model!C3` |
| C | Total Cost | `=Cost_Model!F3` |
| D | Burn Rate (Net) | `=B3 - C3` — positive = cash inflow, negative = burn |
| E | Cash Balance | Month 1: `=Assumptions!$B$21 + D3` / Month 2+: `=E3 + D4` |

**Critical formula — Cash Balance accumulation:**
```
Month 1:  Cash Balance = Starting Cash + Month 1 Net Cash Flow
          E3 = Assumptions!B21 + D3 = ₹5,00,000 + ₹7,000 = ₹5,07,000

Month 2+: Cash Balance = Prior Month Balance + Current Net Cash Flow
          E4 = E3 + D4 = ₹5,07,000 + ₹9,100 = ₹5,16,100
```

---

## Sensitivity_Analysis Sheet

Tests how Month 36 Net Profit changes when Price or Growth Rate varies.

**Price sensitivity (column A–B):**
```
For each price point → recalculate Revenue → recalculate Net Profit at Month 36
```

| Price (₹) | Month 36 Net Profit |
|---|---|
| 400 | ₹1,23,114 |
| 450 | ₹1,51,937 |
| **500** | **₹1,80,760** ← base case |
| 550 | ₹2,09,583 |
| 600 | ₹2,38,405 |

**Growth Rate sensitivity (column D–E):**

| Growth Rate | Month 36 Net Profit |
|---|---|
| 3% | ₹52,864 |
| 5% | ₹1,23,795 |
| **6%** | **₹1,80,760** ← base case |
| 8% | ₹3,67,115 |
| 10% | ₹7,16,689 |
| 12% | ₹13,64,990 |

---

## Dashboard Sheet

Pulls summary KPIs from across the model.

| Metric | Formula |
|---|---|
| Total Revenue (Year 3) | `=SUM(Revenue_Model!C2:C37)` |
| Final Net Profit | `=SUM(Profit_Loss!F2:F37)` |
| Total Customers (Month 36) | `=Customer_Model!E38` |
| Cash Balance (Month 36) | Pulled from Cash_Flow sheet |