# 📖 Model Explanation

A plain-English walkthrough of how the Startup Financial Planning Model works, why it's structured the way it is, and how all the sheets connect together.

---

## What This Model Does

This is a **bottom-up financial model** that projects a startup's financial performance over 36 months (3 years). Instead of guessing a revenue number and working backwards, it starts from a single customer count and builds up revenue, costs, profit, and cash position organically month by month.

The entire model is driven by **10 assumptions** stored in one place. Change any assumption and every sheet recalculates instantly.

---

## How the Sheets Connect

The model flows in one direction — data moves forward, never backwards:

```
Assumptions
     │
     ▼
Customer_Model  ──────────────────────────────┐
     │                                         │
     ▼                                         │
Revenue_Model                             Unit_Economics
     │
     ├──────────────────┐
     ▼                  ▼
Cost_Model       Model_Calculations
     │                  │
     └────────┬──────────┘
              ▼
         Profit_Loss
              │
              ▼
          Cash_Flow
              │
              ▼
    Sensitivity_Analysis
              │
              ▼
          Dashboard
```

**Single source of truth:** Customer counts flow from `Customer_Model` → `Revenue_Model` → `Cost_Model`. This ensures the same customer number is used everywhere and there are no inconsistencies across sheets.

---

## Sheet-by-Sheet Explanation

### 1. Assumptions
The control panel of the entire model. Every input that drives the business logic sits here — growth rate, pricing, costs, churn, starting cash. Nothing is hardcoded in the calculation sheets. If you want to model a different scenario, this is the only sheet you need to touch.

---

### 2. Customer_Model
The foundation of the model. This sheet answers: *"How many customers do we have each month?"*

It uses a **net customer growth formula** that applies both growth and churn simultaneously:

```
End Customers = Start Customers + New Customers − Churned Customers
```

This is more realistic than simple compounding growth because it acknowledges that some customers leave every month. With a 6% growth rate and 3% churn, the **net monthly growth is approximately 3%**, which is why customer count grows from 100 to ~281 over 36 months rather than the ~769 you'd get ignoring churn.

**Why this matters:** Ignoring churn would have overstated Month 36 revenue by nearly 3x.

---

### 3. Revenue_Model
Answers: *"How much money are we making each month?"*

Revenue is straightforward — it takes the ending customer count from `Customer_Model` and multiplies by the monthly price per customer. This sheet does not independently calculate customers; it always defers to `Customer_Model` to ensure consistency.

```
Revenue = Customers × Price per Customer
```

---

### 4. Cost_Model
Answers: *"How much are we spending each month?"*

Costs are split into three buckets:

- **Variable Cost** — scales with customers (e.g. hosting, support, fulfilment)
- **Fixed Cost** — flat every month regardless of customers (e.g. rent, salaries)
- **Marketing Cost** — flat monthly spend to acquire new customers

This separation is intentional. It lets you see which costs are under control (fixed) and which grow with the business (variable). As customer count grows, variable cost grows proportionally — but fixed and marketing costs stay flat, which is why margins improve over time.

---

### 5. Model_Calculations
Answers: *"Are we profitable, and when do we break even?"*

This sheet brings revenue and costs together to calculate:

- **Gross Profit** = Revenue − Total Cost
- **Contribution Margin** = Price − Variable Cost per Customer (₹350 per customer)
- **Break-Even Point** = Fixed + Marketing Costs ÷ Contribution Margin (~57 customers)
- **Cumulative Profit** = Running total of gross profit since Month 1

Since the model starts with 100 customers and break-even is at ~57, **the business is profitable from Month 1** in the base case.

---

### 6. Unit_Economics
Answers: *"Is the business efficient at acquiring and retaining customers?"*

The three metrics here are the most important for evaluating startup health:

**CAC (Customer Acquisition Cost)**
How much does it cost to acquire one new customer?
```
CAC = Monthly Marketing Spend ÷ Average Monthly New Customers
    = ₹8,000 ÷ 11 = ~₹727
```

**LTV (Lifetime Value)**
How much revenue does a customer generate before churning?
```
LTV = Price per Customer ÷ Monthly Churn Rate
    = ₹500 ÷ 0.03 = ₹16,667
```
This assumes a customer stays for an average of 1/churn rate months (33 months at 3% churn).

**LTV/CAC Ratio**
The ultimate efficiency metric — how many times over does a customer pay back their acquisition cost?
```
LTV/CAC = ₹16,667 ÷ ₹727 = ~22.9x
```
Industry benchmark: anything above 3x is healthy. At 22.9x, this model shows very strong unit economics.

---

### 7. Profit_Loss
Answers: *"What is the business's net profit after tax each month?"*

A standard monthly P&L:
```
Revenue
− Total Cost
= Profit Before Tax
− Tax (25%)
= Net Profit
```

Tax is applied monthly at 25% of pre-tax profit. This is a simplification — in reality, corporate tax is calculated annually. The monthly application gives a more conservative and consistent view of profitability.

---

### 8. Cash_Flow
Answers: *"How much cash do we actually have in the bank?"*

This sheet tracks the **real cash position** of the business starting from the ₹5,00,000 initial capital:

```
Month 1 Cash Balance = Starting Cash + Month 1 Net Cash Flow
Month N Cash Balance = Prior Month Balance + Month N Net Cash Flow
```

The key difference from the P&L is the **starting cash** — the business begins with ₹5,00,000 of capital, so even in early months where profits are small, the cash balance is healthy.

**Burn Rate column** shows monthly net cash flow (positive = generating cash, negative = burning cash). In this model the business generates positive cash from Month 1, so there is no burn period.

---

### 9. Sensitivity_Analysis
Answers: *"How sensitive are our results to key assumptions?"*

Tests two variables against Month 36 Net Profit:

- **Price sensitivity** — what happens to profit if we charge ₹400 vs ₹600?
- **Growth Rate sensitivity** — what happens if we grow at 3% vs 12% per month?

Key insight from the growth rate analysis: the relationship is highly non-linear. Going from 6% to 12% monthly growth doesn't double profit — it increases it by **~7.5x** due to compounding over 36 months. This highlights how critical early growth rate is to long-term outcomes.

---

### 10. Dashboard
A one-page summary of the most important outputs. Useful for sharing with stakeholders who don't need to see the full model detail.

| Metric | Value |
|---|---|
| Total Revenue (36 months) | ₹30,23,104 |
| Total Net Profit (36 months) | ₹21,90,163 |
| Customers at Month 36 | ~281 |
| Final Cash Balance | ~₹24,10,130 |

---

## Design Decisions & Tradeoffs

### Why separate Customer_Model from Revenue_Model?
Keeping customer counts in their own sheet makes it easy to swap in a more complex customer model later (e.g. adding multiple customer segments, cohort tracking, or seasonality) without touching the revenue logic.

### Why are Fixed and Marketing costs not scaling?
This is a simplification that works for early-stage modelling. In reality, marketing spend and fixed costs would increase as the business grows. You can extend the model by making these cells reference a growth formula rather than a flat assumption.

### Why apply tax monthly?
Consistency. Applying tax annually would create large swings in the monthly P&L. The monthly approach smooths this out and gives a more conservative view of profitability that is easier to reason about month to month.

### Why use Average New Customers for CAC?
Month 1 new customers (6) is an artificially low base because the business is just starting. Using the 36-month average (~11) gives a more representative CAC figure that accounts for the growing acquisition engine over time.

---

## How to Stress Test the Model

1. **Bear case** — reduce growth rate to 3%, increase churn to 5%
2. **Bull case** — increase growth rate to 10%, reduce churn to 1%
3. **Price test** — use Sensitivity Analysis sheet to see profit impact of ±₹100 on price
4. **Cost shock** — double Fixed Monthly Cost to simulate a hiring round
5. **Capital test** — reduce Starting Cash to ₹1,00,000 to see if cash balance stays positive

All changes should be made only in the **Assumptions sheet**.