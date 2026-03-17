# 🏥 US Healthcare Payment Integrity Analysis
**Medicare Inpatient Billing Analysis | Python · MySQL · Matplotlib**

---

## 📌 Project Overview

End-to-end analysis of US Medicare inpatient billing data to identify
payment integrity risks across 3,182 hospitals and $9.82 billion in
billing gap.

Completed as part of the EXL Service Analytics Case Study —
Assistant Manager role — Payment Integrity division.

---

## 🔑 Key Findings

| Finding | Number | Action |
|---|---|---|
| Hospitals above 10x charge ratio | 200 hospitals (6.3%) | Immediate audit flag |
| For-profit vs Govt charge ratio | 8.50x vs 4.16x | Ownership-based risk scoring |
| Low-rated hospitals overbill most | 2.15M patients affected | Quality-linked audit priority |
| Heart Failure total exposure | $11.3B gap | Volume-weighted audit protocol |
| State billing gap | Maryland 1.22x vs Nevada 9.03x | Policy investigation |

---

## 📂 Datasets Used

| Dataset | Source | Records |
|---|---|---|
| T1 — Inpatient Payments | CMS data.cms.gov | 196,086 rows |
| T2 — DRG Details | CMS data.cms.gov | 543 rows |
| T3 — Provider Info | CMS data.cms.gov | 5,426 rows |
| T4 — HCAHPS Survey (External) | CMS data.cms.gov | 325,652 rows |

> ⚠️ Raw datasets not included due to size.
> Download from: https://data.cms.gov

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| Python (pandas, numpy) | Data cleaning · Feature engineering · ETL |
| MySQL | 3-table JOIN analysis · GROUP BY · Subqueries |
| Matplotlib | 7 analysis charts · PNG exports |
| PowerPoint / WPS | 13-slide presentation |

---

## 🔍 Key SQL Queries

### Billing Gap and Charge Ratio — Metric Creation
```sql
SELECT provider_name, state,
  AVG(avg_charged - avg_medicare_payment) AS billing_gap,
  AVG(avg_charged / avg_medicare_payment) AS charge_ratio
FROM inpatient_payments
GROUP BY provider_name, state
ORDER BY charge_ratio DESC
LIMIT 10
```

### Ownership vs Billing Behaviour — JOIN + Aggregation
```sql
SELECT p.hospital_ownership,
  COUNT(DISTINCT i.provider_id) AS hospitals,
  ROUND(AVG(i.avg_charged / i.avg_medicare_payment), 2)
  AS avg_charge_ratio
FROM inpatient_payments i
JOIN provider_info p ON i.provider_id = p.facility_id
GROUP BY p.hospital_ownership
ORDER BY avg_charge_ratio DESC
```

### Star Rating vs Charge Ratio — Hospital Level Average
```sql
SELECT rating,
  COUNT(*) AS hospitals,
  ROUND(AVG(avg_ratio), 2) AS avg_charge_ratio,
  SUM(total_patients) AS total_patients
FROM (
  SELECT i.provider_id,
    p.overall_rating AS rating,
    AVG(i.charge_ratio) AS avg_ratio,
    SUM(i.total_discharges) AS total_patients
  FROM inpatient_payments i
  JOIN provider_info p ON i.provider_id = p.facility_id
  WHERE p.overall_rating NOT IN ('Unknown', 'Not Available', '0')
  GROUP BY i.provider_id, p.overall_rating
) hospital_level
GROUP BY rating
ORDER BY rating
```

### External Dataset JOIN — HCAHPS Nurse Communication
```sql
SELECT b.provider_name,
  b.avg_charge_ratio,
  h.nurse_always_pct
FROM inpatient_payments b
JOIN hcahps h ON b.provider_id = h.facility_id
WHERE h.HCAHPS_Measure_ID = 'H_COMP_1_A_P'
ORDER BY avg_charge_ratio DESC
```


---

## 💡 Highlights

- Joined 4 datasets using hospital CCN as common key
- Fixed CCN zero-padding mismatch between T1 (integer) and T3 (string)
- Created 3 engineered features — billing_gap · charge_ratio · mdc
- Linked external CMS HCAHPS survey dataset across 2,812 hospitals
- Found hospitals with poor nurse communication charge **44% more**
- Maryland All-Payer Model proves regulation controls overbilling —
  1.22x vs Nevada 9.03x for identical treatments

---

## 👤 Author
**Shasti Alagan R**
