# HR Attrition Analytics Dashboard

A Power BI dashboard analyzing employee attrition drivers across 1,470 employees, using IBM's classic HR analytics sample dataset. Built to demonstrate DAX-heavy, categorical people-analytics — a distinct domain from transactional/retail analysis.

## Business Questions Answered

1. What is the overall attrition rate, and how does it break down by Department and Job Role?
2. Does OverTime correlate with higher attrition?
3. How does attrition vary by tenure (New Hires vs. Veterans)?
4. What's the relationship between Job Level, income, and attrition?
5. Do job/environment satisfaction scores vary meaningfully across departments?

## Data Source

[IBM HR Analytics Employee Attrition \& Performance – Kaggle](https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset) — 1,470 employees, 35 original fields.

## Tools Used

Power BI Desktop · Power Query · DAX

## Data Cleaning

Unlike a scraped retail dataset, this is a well-known, genuinely clean sample dataset — verified via Power Query's Column Quality profiling (entire dataset, not a sample):

* **Zero nulls, zero duplicate rows** across all 1,470 records
* `EmployeeNumber` confirmed as a true unique identifier
* **Removed 3 constant columns** providing zero analytical value: `EmployeeCount` (always 1), `Over18` (always "Y"), `StandardHours` (always 80) — leftover artifacts of IBM's original data export, not analytically useful

## Data Model

Single flat table (`HR`, 1,470 rows, 32 columns after cleanup). No relationships required. Built a **Department → Job Role hierarchy** to support drill-down/expand analysis, revealing role-level detail hidden by department-level averages alone.

## Key DAX Measures

```dax
Total Employees = COUNTROWS(HR)

Attrited Employees = CALCULATE(COUNTROWS(HR), HR\\\\\\\[Attrition] = "Yes")

Attrition Rate = DIVIDE(\\\\\\\[Attrited Employees], \\\\\\\[Total Employees])

Tenure Bucket =
SWITCH(
    TRUE(),
    HR\\\\\\\[YearsAtCompany] <= 2, "New Hire (0-2 yrs)",
    HR\\\\\\\[YearsAtCompany] <= 5, "Establishing (3-5 yrs)",
    HR\\\\\\\[YearsAtCompany] <= 10, "Experienced (6-10 yrs)",
    "Veteran (10+ yrs)"
)

Average Monthly Income = AVERAGE(HR\\\\\\\[MonthlyIncome])
```

`Attrition Rate` is the backbone measure of the entire project — every breakdown below reuses this single formula, recalculated automatically for each filter context (department, role, tenure bucket, etc.) rather than requiring separate formulas per question.

## Findings

**Overall attrition rate: 16.1%** (237 of 1,470 employees) — the baseline every other figure below is compared against.

### Department \& Job Role

* Drilling into Job Role reveals department averages hide large internal gaps: **Sales Representative attrites at 39.8%** — nearly double Sales' own department average — while **Sales Managers sit at just 5.4%**
* **Manager-level roles are consistently the most stable position across every department** (Sales 5.4%, R\&D 5.6%, HR 0.0%), suggesting role/seniority predicts attrition risk more reliably than department alone
* Other high-churn roles: Laboratory Technician (R\&D) at 23.9%, and the "Human Resources" role itself (within HR department) at 23.1% — HR's entire attrition problem is concentrated in this one non-managerial role

### OverTime

* Employees working overtime attrite at **30.5%**, vs. **10.4%** for those who don't — nearly **3x higher risk**
* The strongest single attrition indicator found in this dataset

### Tenure

* **New Hires (0-2 yrs): 29.8%** attrition — by far the highest
* Establishing (3-5 yrs): 13.8% · Experienced (6-10 yrs): 12.3% · **Veterans (10+ yrs): 8.1%**
* New Hires leave at **\~3.7x the rate of Veterans** — risk drops sharply after the first two years

### Income \& Job Level

* Attrition generally **decreases as Job Level (and income) rises**: Level 1 (\~₹2,787/mo avg) has 26.3% attrition, dropping to 9.7% at Level 2
* **Anomaly: Level 3 breaks the trend at 14.7%**, higher than both Level 2 and Level 4 — a possible mid-career retention gap worth further investigation
* Levels 4 (n=106) and 5 (n=69) have small sample sizes; their rates (4.7%, 7.2%) shouldn't be over-interpreted

### Satisfaction

* **Minimal variation across departments** — Job Satisfaction ranges only 2.60-2.75, Environment Satisfaction only 2.68-2.74 (both on a 1-4 ordinal scale)
* Department is not a meaningful driver of satisfaction differences in this dataset — a useful "ruled out" finding, not a failed analysis

## Recommendations 

1. **Prioritize retention efforts by role, not department** — Sales Representatives and Lab Technicians are the actual high-risk groups, not their departments broadly
2. **Investigate overtime policy** — the \~3x attrition gap associated with overtime is the strongest signal in the data
3. **Focus onboarding/early retention programs on the 0-2 year tenure window**, where attrition is highest
4. **Examine Level 3 specifically** for promotion/compensation gaps, given its anomalous attrition spike relative to neighboring levels

## 

