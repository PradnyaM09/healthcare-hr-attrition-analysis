# Healthcare HR Attrition Analysis
**A SQL-based analysis of employee turnover using a synthetic healthcare organization dataset (350 employees)**

---

## Summary

This project analyzes attrition patterns across a simulated 350-employee healthcare organization, using SQL to identify where turnover risk is concentrated and what factors correlate with it. The dataset includes department, shift, tenure, recruitment source, satisfaction scores, and overtime hours for each employee.

**Overall attrition rate: 28.9%**

---

## Key Findings

### 1. Attrition is heavily concentrated in Housekeeping
| Department | Attrition Rate |
|---|---|
| Housekeeping | 46.2% |
| Pharmacy | 39.3% |
| Physical Therapy | 37.0% |
| IT/Facilities | 34.6% |
| Radiology | 33.3% |
| Billing/Records | 29.2% |
| Emergency | 27.3% |
| Lab/Pathology | 22.6% |
| Nursing | 20.9% |
| Administration | 17.4% |

**Insight:** Housekeeping's attrition rate is nearly 2.5x that of Administration, and well above the company average — a clear priority area for retention intervention.

### 2. Night shift has the highest turnover
| Shift | Attrition Rate |
|---|---|
| Night | 32.6% |
| Day | 29.9% |
| Evening | 24.0% |

**Insight:** This matches a well-documented pattern in healthcare — night-shift staff experience higher burnout and turnover than day/evening staff.

### 3. Referral hires retain far better than agency/contract hires
| Recruitment Source | Attrition Rate |
|---|---|
| Agency/Contract Staffing | 38.0% |
| Campus Recruiting | 33.3% |
| Internal Transfer | 31.6% |
| Job Board | 28.2% |
| Nursing School Partnership | 27.5% |
| Referral | 21.1% |

**Insight:** Referral-sourced employees retain nearly 2x better than agency/contract staff, suggesting recruitment budget reallocation toward referral incentive programs could meaningfully reduce turnover.

### 4. New hires (under 2 years) are the highest-risk group
| Tenure Group | Attrition Rate |
|---|---|
| New (<2 yrs) | 36.0% |
| Established (2-5 yrs) | 25.4% |
| Old/Veteran (5+ yrs) | 25.5% |

**Insight:** Attrition risk is concentrated in the first two years of employment. Once employees pass that threshold, retention stabilizes significantly — pointing to onboarding and early-career support as a high-leverage intervention point.

### 5. Departing employees report meaningfully lower satisfaction
| Status | Avg. Satisfaction (1-5 scale) |
|---|---|
| Left | 2.68 |
| Stayed | 3.12 |

**Insight:** A clear, expected gap — validating that engagement/satisfaction initiatives are a legitimate lever for reducing turnover.

### 6. Lab/Pathology carries the heaviest overtime load
| Department | Avg. Overtime Hours/Month |
|---|---|
| Lab/Pathology | 12.35 |
| Pharmacy | 11.0 |
| Emergency | 9.8 |
| Nursing | 9.54 |
| IT/Facilities | 9.65 |
| Administration | 8.96 |
| Billing/Records | 8.96 |
| Housekeeping | 8.42 |
| Radiology | 7.57 |
| Physical Therapy | 7.52 |

**Notable cross-check:** Housekeeping has the *lowest* overtime but the *highest* attrition — meaning overtime alone does not explain Housekeeping's turnover problem. This suggests a different root cause (likely pay, shift structure, or job conditions) is driving that department's specific issue.

### 7. High overtime correlates with higher attrition, company-wide
| Overtime Level | Attrition Rate |
|---|---|
| High (>15 hrs/month) | 34.38% |
| Low (≤15 hrs/month) | 27.62% |

**Insight:** While overtime doesn't explain Housekeeping specifically (finding #6), it is a real company-wide attrition driver — worth monitoring and capping where possible.

### 8. Top 5 highest-paid currently-employed staff
*(Query built; specific names pending final case-sensitivity fix — see Methodology Notes)*

---

## Methodology

- **Tool:** SQLite (via DB Browser for SQLite)
- **Dataset:** Synthetic 350-employee healthcare dataset, generated with realistic embedded patterns (higher attrition for night shift, Emergency dept, high overtime, low satisfaction, and agency-sourced hires)
- **Techniques used:** conditional aggregation (`CASE WHEN` inside `COUNT`/`AVG`/`SUM`), `JOIN`s, `GROUP BY`/`HAVING`, date-based tenure calculation (`julianday()`), and CTEs for comparing individual values against group-level aggregates
- **Cross-validation:** Findings #1, #3, and #4 were independently rebuilt in Google Sheets (pivot tables + IF-based helper columns) and matched the SQL results to within rounding — confirming the analysis is correct across two independent tools

---

## Core SQL Queries Used

**Overall attrition rate:**
```sql
SELECT ROUND(COUNT(CASE WHEN attrition = 'Yes' THEN 1 END) * 100.0 / COUNT(employee_id), 1) AS attrition_rate_pct
FROM healthcare_hr_data;
```

**Attrition rate by department:**
```sql
SELECT department,
ROUND(COUNT(CASE WHEN attrition = 'Yes' THEN 1 END) * 100.0 / COUNT(employee_id), 1) AS attrition_percentage
FROM healthcare_hr_data
GROUP BY department;
```

**Attrition rate by shift:**
```sql
SELECT shift,
ROUND(COUNT(CASE WHEN attrition = 'Yes' THEN 1 END) * 100.0 / COUNT(employee_id), 1) AS attrition_percentage
FROM healthcare_hr_data
GROUP BY shift;
```

**Attrition rate by recruitment source:**
```sql
SELECT recruitment_source,
ROUND(COUNT(CASE WHEN attrition = 'Yes' THEN 1 END) * 100.0 / COUNT(employee_id), 1) AS attrition_percentage
FROM healthcare_hr_data
GROUP BY recruitment_source
ORDER BY attrition_percentage ASC;
```

**Attrition rate by tenure group (date-based bucketing):**
```sql
SELECT
CASE WHEN (julianday('now') - julianday(hire_date)) / 365 < 2 THEN 'New'
     WHEN (julianday('now') - julianday(hire_date)) / 365 < 5 THEN 'Established'
     ELSE 'Old'
END AS tenure_group,
ROUND(COUNT(CASE WHEN attrition = 'Yes' THEN 1 END) * 100.0 / COUNT(employee_id), 1) AS attrition_percentage
FROM healthcare_hr_data
GROUP BY tenure_group;
```

**Satisfaction score comparison (left vs. stayed):**
```sql
SELECT
CASE WHEN attrition = 'Yes' THEN 'left_employees' ELSE 'present_employees' END AS attrition_status,
ROUND(AVG(satisfaction_score), 2) AS avg_satisfaction
FROM healthcare_hr_data
GROUP BY attrition_status;
```

**Average overtime by department:**
```sql
SELECT department, ROUND(AVG(overtime_hours_per_month), 2) AS avg_overtime
FROM healthcare_hr_data
GROUP BY department
ORDER BY avg_overtime DESC;
```

**Overtime-attrition link:**
```sql
SELECT
CASE WHEN overtime_hours_per_month > 15 THEN 'High' ELSE 'Low' END AS overtime_group,
ROUND(COUNT(CASE WHEN attrition = 'Yes' THEN 1 END) * 100.0 / COUNT(employee_id), 2) AS attrition_rate
FROM healthcare_hr_data
GROUP BY overtime_group;
```

**Top 5 highest-paid retained employees:**
```sql
SELECT name, salary
FROM healthcare_hr_data
WHERE attrition = 'No'
ORDER BY salary DESC
LIMIT 5;
```

---

## Recommendations (business-facing summary)

1. **Investigate Housekeeping specifically** — its attrition is far above every other department and is not explained by overtime, so likely drivers are pay, shift conditions, or management practices unique to that department.
2. **Strengthen the first-2-year onboarding experience** — new hires leave at nearly 1.5x the rate of tenured staff; targeted early-career support/mentorship could meaningfully reduce this.
3. **Shift recruitment spend toward referral programs** — referral hires retain substantially better than agency/contract staff.
4. **Monitor and cap overtime where feasible**, particularly outside Housekeeping, given its measurable link to attrition.
5. **Use satisfaction scores as an early warning signal** — the clear gap between leavers and stayers suggests regular pulse surveys could help flag at-risk employees before they leave.

---

*Dataset and analysis built as a self-directed learning project to demonstrate SQL proficiency (joins, aggregation, CASE logic, CTEs) applied to a realistic HR analytics use case.*
