# Healthcare HR Attrition Analysis

A SQL-based analysis identifying where employee turnover risk is concentrated in a 350-employee healthcare organization, and what factors correlate with it.

**Tools:** SQL (SQLite) · Google Sheets (cross-validation) · Python/Faker (synthetic data generation)

---

## The Question

Where is attrition concentrated, and what's actually driving it — is it overtime, tenure, recruitment channel, shift, or something else entirely?

## Key Findings

- **Overall attrition rate: 28.9%**
- **Housekeeping is the highest-risk department (46.2%)** — nearly 2.5x Administration's rate (17.4%), and *not* explained by overtime (Housekeeping actually has the **lowest** overtime of any department)
- **New hires (<2 years) attrite at 36.0%**, vs. ~25% for both established and veteran staff — risk is concentrated early in tenure
- **Referral hires retain ~2x better than agency/contract staff** (21.1% vs. 38.0% attrition)
- **Night shift has the highest turnover by shift** (32.6% vs. 24.0% for evening)
- **Satisfaction scores are a real early-warning signal** — leavers average 2.68/5 vs. 3.12/5 for retained staff
- **High overtime (>15 hrs/month) correlates with higher attrition company-wide** (34.4% vs. 27.6%) — but doesn't explain Housekeeping's problem specifically, pointing to a different root cause there (likely pay or job conditions)

Full findings, tables, and business recommendations: [`Healthcare_HR_Analytics_Project.md`](./Healthcare_HR_Analytics_Project.md)

## Methodology

- Synthetic 350-employee dataset generated with Python/Faker, with realistic patterns deliberately embedded (higher attrition for night shift, high overtime, low satisfaction, agency-sourced hires)
- SQL techniques: conditional aggregation (`CASE WHEN` inside `COUNT`/`AVG`/`SUM`), `JOIN`s, `GROUP BY`/`HAVING`, date-based tenure bucketing (`julianday()`), and CTEs for comparing individual rows against group-level aggregates
- **Cross-validated in Google Sheets** — 3 of 8 findings independently rebuilt via pivot tables and IF-based helper columns, matching the SQL results to within rounding

## Files

| File | Contents |
|---|---|
| [`Healthcare_HR_Analytics_Project.md`](./Healthcare_HR_Analytics_Project.md) | Full write-up: all 8 findings, methodology, every SQL query used, and business recommendations |
| [`healthcare_hr_data.csv`](./healthcare_hr_data.csv) | The underlying synthetic dataset |

## Sample Query

Attrition rate by department, using conditional aggregation:

```sql
SELECT department,
    ROUND(COUNT(CASE WHEN attrition = 'Yes' THEN 1 END) * 100.0 / COUNT(employee_id), 1) AS attrition_percentage
FROM healthcare_hr_data
GROUP BY department;
```

## Recommendations

1. Investigate Housekeeping specifically — its attrition isn't explained by overtime, so the driver is likely pay, shift conditions, or department-specific management practices
2. Strengthen first-2-year onboarding — new hires leave at nearly 1.5x the rate of tenured staff
3. Shift recruitment spend toward referral programs over agency/contract staffing
4. Monitor and cap overtime where feasible, outside Housekeeping
5. Use satisfaction pulse surveys as an early warning signal for at-risk employees

---

*Self-directed learning project built to demonstrate SQL proficiency (joins, aggregation, CASE logic, CTEs) applied to a realistic HR analytics use case.*
