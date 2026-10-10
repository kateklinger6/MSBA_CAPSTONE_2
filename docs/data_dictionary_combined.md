# Combined Data Dictionary: PBS Utah Donor Data (Synthetic)

Every variable in the nine course tables, with its profile from the data and how the EDA recommends treating it. Definitions combine the course dictionaries with what the EDA found in the data. Machine-readable copy: `data_dictionary_combined.csv`.

**EDA status key:**
- **Key / link:** an ID used to join tables.
- **Use:** a usable feature or input.
- **Use with care:** usable with the caveat noted.
- **Leaks — exclude:** reflects FY2026 outcomes, so it would leak the answer.
- **Drop:** constant, blank, a placeholder, or redundant.

| Status | Variables |
|---|---:|
| Key / link | 18 |
| Use | 13 |
| Use with care | 17 |
| Leaks — exclude | 6 |
| Drop | 75 |

## How the tables join

```
constituents_w_memberships (Constituent ID)
 ├── unite_payments.donor_id
 ├── soft_credits."Soft Credit Constituent ID"
 ├── team_approach_legacy_payments.constituent_id_1 / _2   (household rows)
 ├── campaign_members.constituent_id ── campaign_codes."Marketing Code" (also unite_payments.marketing_code)
 ├── passport_viewing.constituent_id
 └── cultivation.synthetic_id ── officer_portfolios.portfolio_id
```

**Rows to remove before any analysis:**
- Payment and soft-credit IDs starting `SYN-DEF-`, and legacy accounts starting `SYN-UNMATCHED-`. All are copies of real rows with one field broken.
- 79 exact-duplicate soft credits.

## `constituents_w_memberships.csv`

One row per person or organization; status as of end of FY2026. **196,167 rows, 22 columns.**

| Variable | Type | Missing | Unique | Example | Description | EDA status |
|---|---|---:|---:|---|---|---|
| `Constituent ID` | text | 0% | 196,167 | S00000001 | Unique person/org ID; joins to every other table. Prefixes: SYN-ORG for the 2 organizations | Key / link |
| `Spouse Constituent ID` | text | 91% | 17,360 | S00043054 | Constituent ID of spouse; blank = no spouse on file (91%) | Use with care: household grouping for validation splits |
| `Address City` | text | 26% | 1 | Synthetic City | Placeholder 'Synthetic City' or blank | Drop |
| `Address ZIP` | number | 0% | 16,891 | 97211.0 | ZIP code (synthetic) | Use with care: geography / fairness audit |
| `Address State` | text | 0% | 43 | OR | State; 82% UT | Use with care: geography / fairness audit |
| `Age` | number | 0% | 71 | 71.0 | Age as of FY2026 (snapshot) | Use after rolling back: Age − (2026 − t) |
| `Annual Giving Group` | text | 0% | 5 | ND | Undocumented code; maps exactly onto FY2026 giving (ND = none … CD = \$1,200+) | Leaks — exclude |
| `Constituent Type` | text | 0% | 13 | Friends | Identical to Primary Constituent Type in every row | Drop (duplicate) |
| `UofU Degree Count` | number | 0% | 1 | 0.0 | Always 0 | Drop |
| `UofU Degree Info` | number | 100% | 0 |  | Always blank | Drop |
| `Latest UofU Degree Year` | number | 100% | 0 |  | Always blank | Drop |
| `UofU Employment Status` | number | 100% | 0 |  | Always blank | Drop |
| `Gender` | text | 0% | 3 | Female | Female / Male / Unknown; 'Unknown' converts at a third the rate | Use with care: audit; 'Unknown' reflects record completeness |
| `Has UofU Legacy Gift` | object | 0% | 1 | False | Always False | Drop |
| `Legacy Circle Member PBS Utah` | object | 0% | 1 | False | Always False | Drop |
| `Lifetime UofU Fundraising` | number | 0% | 539 | 3500.0 | Documented as UofU giving; actually tracks PBS giving through FY2026 | Leaks — exclude |
| `Major Donor Class PBS Utah` | text | 100% | 3 | Former | Broadcaster/Director only for FY2026 major donors; 99.7% blank | Leaks — exclude |
| `Marital Status` | text | 91% | 1 | Married | Only 'Married' or blank (blank ≠ single) | Use with care |
| `Previous FY UofU Cash` | number | 0% | 6 | 0.0 | Positive exactly when the person gave to PBS in FY2025 | Leaks — exclude |
| `Primary Constituent Type` | text | 0% | 13 | Friends | Friends, Donors, Alumni, Faculty, … , Organization | Use with care: use to exclude organizations |
| `Prospect Committed Giving Amount UofU` | number | 0% | 46 | 0.0 | Mirrors FY2026 recurring PBS giving | Leaks — exclude |
| `Sustainer PBS Utah` | object | 0% | 2 | False | Currently a monthly sustainer (FY2026 snapshot) | Leaks — exclude; rebuild from payments each year |

## `unite_payments.csv`

One row per payment, FY2021–FY2026. **1,722,698 rows, 13 columns.**

| Variable | Type | Missing | Unique | Example | Description | EDA status |
|---|---|---:|---:|---|---|---|
| `donor_id` | text | 0% | 71,643 | S00000002 | Payer; joins to Constituent ID. SYN-ORPHAN rows are defective copies; SYN-ORG-* are org hard credits | Key / link |
| `designation_detail_id` | text | 0% | 30,312 | Designation 14194 | Fund designation; 2 values cover ~880K rows, not a usable join key | Drop |
| `credit_date` | text | 0% | 2,191 | 2020-07-02 | Payment date → fiscal year (July–June) | Use |
| `tender_type` | text | 0% | 8 | EFT | Payment method; 'Legacy' on 43% of rows is undocumented | Use with care |
| `payment_amount` | integer | 0% | 963 | 8 | Payment amount (\$); sum positives by donor-year | Use |
| `pledge_gift_status` | text | 0% | 1 | Paid | Always 'Paid' | Drop |
| `pledge_gift_type` | text | 0% | 2 | Outright | Recurring vs. Outright | Use: recurring flag per year |
| `opportunity_id` | text | 0% | 1,686,654 | SYN-OPP-000000001 | One per real payment; can't group recurring schedules | Drop |
| `payment_frequency` | text | 0% | 2 | Annual | Monthly (= Recurring) / Annual (= Outright) | Drop (duplicate of pledge_gift_type) |
| `gift_type` | text | 0% | 6 | Renew | New / Renew / Rejoin / Upgrade / Additional / Downgrade | Use: e.g., upgrade-gift flag (9× lift) |
| `campaign_name` | text | 0% | 11,524 | Synthetic Campaign 000680 | Campaign name for marketing_code | Drop (redundant) |
| `payment_id` | text | 0% | 1,722,698 | SYN-PAY-000000001 | Unique row ID; SYN-DEF-* prefix marks defective copies | Key / link; drop SYN-DEF-* rows |
| `marketing_code` | text | 0% | 11,525 | SYN-CODE-000680 | Campaign/source code; joins to campaign_codes | Key / link |

## `soft_credits.csv`

One row per soft credit (org-paid gift credited to a person), FY2021–FY2026. **4,145 rows, 9 columns.**

| Variable | Type | Missing | Unique | Example | Description | EDA status |
|---|---|---:|---:|---|---|---|
| `Credit ID` | text | 0% | 4,066 | SYN-SC-000000029 | Credit ID; 79 exact duplicates; SYN-DEF-* prefix marks defective copies | Key / link; dedupe, drop SYN-DEF-* |
| `Designation Detail ID` | text | 0% | 1,256 | Designation 1 | Designation of the matching org payment | Drop |
| `Soft Credit Constituent ID` | text | 0% | 3,028 | S00000023 | Person credited; joins to Constituent ID | Key / link |
| `Hard Credit Org ID` | text | 0% | 2 | SYN-ORG-SOFT | SYN-ORG-DAF or SYN-ORG-SOFT | Use with care |
| `Hard Credit Org Type` | text | 0% | 2 | Other Sponsor | DAF Sponsor / Other Sponsor | Use: soft-credit/DAF flag (20× lift) |
| `Credit Gift Type` | text | 0% | 1 | Payment | Always 'Payment' | Drop |
| `Credit Amount` | integer | 0% | 777 | 420 | Credited amount (\$); counts as the person's giving | Use |
| `Credit Date` | text | 0% | 1,756 | 2020-08-11 | Credit date → fiscal year | Use |
| `Credit Type` | text | 0% | 1 | Soft | Always 'Soft' | Drop |

## `team_approach_legacy_payments.csv`

One row per old-system household gift, FY1984–FY2020. **1,202,857 rows, 44 columns.**

| Variable | Type | Missing | Unique | Example | Description | EDA status |
|---|---|---:|---:|---|---|---|
| `ta_account_id` | text | 0% | 588,843 | SYN-HH-00000001-1984 | Household account ID (household × FY); SYN-UNMATCHED-* rows are defective copies | Key / link; drop SYN-UNMATCHED-* |
| `gift_kind` | text | 0% | 2 | One-Time | Sustaining / One-Time | Use: history features |
| `gift_type` | text | 0% | 6 | New | Renew / New / Rejoin / Additional / Upgrade / Other | Use: history features |
| `gift_date` | text | 0% | 8,121 | 1984-01-20 | Gift date (FY1984–FY2020) | Use: tenure, past-major flag |
| `activity_type` | text | 0% | 1 | M | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `campaign` | text | 0% | 1 | Q | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `initiative` | integer | 0% | 1 | 0 | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `effort` | integer | 0% | 1 | 0 | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `source_code` | text | 0% | 986,729 | SYN-00000001 | Unique code per row; no link to campaign_codes | Drop |
| `source_description` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `payment_amount` | integer | 0% | 1,595 | 1200 | Household gift amount (\$); history only, never revenue | Use: history only |
| `payment_method` | text | 0% | 1 | Synthetic | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `city` | text | 0% | 1 | Synthetic City | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `state` | text | 0% | 43 | OR | State | Drop (use constituent table) |
| `zip` | integer | 0% | 341 | 97200 | 3-digit-rounded ZIP | Drop (use constituent table) |
| `county` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `program` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `technique_trans` | text | 0% | 1 | Synthetic | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `pledge_time` | text | 0% | 8,121 | 1984-01-20 | Same as gift_date | Drop (duplicate) |
| `premium_1_code` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `premium_1_name` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `premium_2_code` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `premium_2_name` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `premium_3_code` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `premium_3_name` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `premium_4_code` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `premium_4_name` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `premium_5_code` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `premium_5_name` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `premium_6_code` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `premium_6_name` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `program_source` | text | 0% | 1 | SYN | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `source_category` | text | 0% | 1 | Synthetic | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `communication_method` | text | 0% | 1 | Synthetic | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `communication_type` | text | 0% | 1 | Synthetic | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `activity` | text | 0% | 1 | A | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `segment` | integer | 0% | 1 | 0 | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `constituent_id_1` | text | 18% | 108,553 | S00000001 | First linked constituent | Key / link |
| `constituent_id_2` | text | 99% | 2,942 | S00010441 | Second linked constituent (0.8% of rows) | Key / link |
| `constituent_id_3` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `constituent_id_4` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `constituent_id_5` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `constituent_id_6` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |
| `constituent_id_7` | number | 100% | 0 |  | Placeholder ('Synthetic', 'SYN', 0) or always blank | Drop |

## `campaign_codes.csv`

One row per marketing code. **29,249 rows, 17 columns.**

| Variable | Type | Missing | Unique | Example | Description | EDA status |
|---|---|---:|---:|---|---|---|
| `Marketing Code` | text | 0% | 29,249 | SYN-CODE-000001 | Unique code; joins to payments and campaign_members | Key / link |
| `Source Category` | text | 0% | 1 | Synthetic | Placeholder, duplicate of Marketing Code, or constant in this release | Drop |
| `Campaign Name` | text | 0% | 29,249 | Synthetic Campaign 000001 | Placeholder, duplicate of Marketing Code, or constant in this release | Drop |
| `Campaign ID` | text | 0% | 29,249 | SYN-CODE-000001 | Placeholder, duplicate of Marketing Code, or constant in this release | Drop |
| `Activity` | text | 0% | 1 | A | Placeholder, duplicate of Marketing Code, or constant in this release | Drop |
| `Activity Type` | text | 0% | 1 | M | Placeholder, duplicate of Marketing Code, or constant in this release | Drop |
| `Campaign Type` | text | 0% | 1 | Q | Placeholder, duplicate of Marketing Code, or constant in this release | Drop |
| `Initiative Year` | integer | 0% | 1 | 0 | Placeholder, duplicate of Marketing Code, or constant in this release | Drop |
| `Initiative Month` | integer | 0% | 1 | 0 | Placeholder, duplicate of Marketing Code, or constant in this release | Drop |
| `Effort` | integer | 0% | 1 | 0 | Placeholder, duplicate of Marketing Code, or constant in this release | Drop |
| `Segment` | integer | 0% | 1 | 0 | Placeholder, duplicate of Marketing Code, or constant in this release | Drop |
| `Communication Method` | text | 0% | 8 | Digital | Legacy / Miscellaneous / Digital / Mail / Email / Phone / In-Person / Third Party | Use with care: only varying field |
| `Communication Type` | text | 0% | 1 | Synthetic | Placeholder, duplicate of Marketing Code, or constant in this release | Drop |
| `Active` | boolean | 0% | 1 | True | Placeholder, duplicate of Marketing Code, or constant in this release | Drop |
| `Campaign Description` | number | 100% | 0 |  | Placeholder, duplicate of Marketing Code, or constant in this release | Drop |
| `Gift Type` | text | 0% | 1 | Synthetic | Placeholder, duplicate of Marketing Code, or constant in this release | Drop |
| `Solicitation` | boolean | 0% | 1 | True | Placeholder, duplicate of Marketing Code, or constant in this release | Drop |

## `campaign_members.csv`

One row per solicitation, FY2021–FY2026. **5,233,404 rows, 8 columns.**

| Variable | Type | Missing | Unique | Example | Description | EDA status |
|---|---|---:|---:|---|---|---|
| `constituent_id` | text | 0% | 181,680 | S00000002 | Solicited person; joins to Constituent ID | Key / link |
| `marketing_code` | text | 0% | 28,385 | SYN-CODE-028679 | Solicitation code; joins to campaign_codes | Key / link |
| `campaign_name` | text | 0% | 28,385 | Synthetic Campaign 028679 | Campaign name | Drop (redundant) |
| `responded` | boolean | 0% | 2 | False | Paid on the same code in the same FY | Use with care: no value once giving is known |
| `response_type` | text | 99% | 1 | Payment | 'Payment' if responded, else blank | Drop (redundant) |
| `indirect_response` | boolean | 0% | 1 | False | Always False | Drop |
| `responded_date` | text | 99% | 2,191 | 2021-03-26 | Earliest responding payment date | Drop (use payment dates) |
| `campaign_start_date` | text | 0% | 6 | 2020-07-01 | First day of the solicitation's FY (a year marker, not a contact date) | Use: → fiscal year |

## `passport_viewing.csv`

One row per person × fiscal year × genre with viewing. **444,741 rows, 5 columns.**

| Variable | Type | Missing | Unique | Example | Description | EDA status |
|---|---|---:|---:|---|---|---|
| `constituent_id` | text | 0% | 70,515 | S00000005 | Viewer; joins to Constituent ID | Key / link |
| `fiscal_year` | integer | 0% | 6 | 2021 | FY of viewing; no row = no account, no consent, or no viewing | Key / link |
| `genre` | text | 0% | 10 | Arts and Music | 10 genres (uncommon ones grouped as Other) | Use with care |
| `titles_viewed` | integer | 0% | 91 | 2 | Distinct titles viewed in genre that year | Use with care: no signal once giving is known |
| `mean_percent_watched` | number | 0% | 284,470 | 1.0 | Average share of each title watched (0–1) | Use with care |

## `cultivation.csv`

One row per donor selected for officer contact in a year, FY2022–FY2025. **4,000 rows, 7 columns.**

| Variable | Type | Missing | Unique | Example | Description | EDA status |
|---|---|---:|---:|---|---|---|
| `synthetic_id` | text | 0% | 2,398 | S00104067 | Selected donor; joins to Constituent ID | Key / link |
| `fiscal_year` | integer | 0% | 4 | 2022 | FY of selection (FY2022–25) | Key / link |
| `portfolio_id` | text | 0% | 8 | P01 | Officer portfolio; joins to officer_portfolios | Key / link |
| `selected` | boolean | 0% | 1 | True | Always True (table only lists selected donors) | Drop |
| `contacted` | boolean | 0% | 2 | True | Assigned for officer contact | Use with care: treatment — exclude from baseline model |
| `holdout` | boolean | 0% | 2 | False | Randomly withheld from contact (= not contacted) | Use with care: treatment-effect analysis only |
| `program_adopted` | boolean | 0% | 2 | False | Portfolio had adopted the cultivation program that year | Use with care: treatment-effect analysis only |

## `officer_portfolios.csv`

One row per officer portfolio. **8 rows, 4 columns.**

| Variable | Type | Missing | Unique | Example | Description | EDA status |
|---|---|---:|---:|---|---|---|
| `portfolio_id` | text | 0% | 8 | P01 | Portfolio ID | Key / link |
| `officer_id` | text | 0% | 8 | OFFICER-01 | Officer ID (one per portfolio) | Drop (redundant) |
| `capacity` | integer | 0% | 1 | 125 | Donors selectable per year; always 125 | Drop |
| `rollout_fy` | number | 12% | 4 | 2023.0 | FY the portfolio adopted the program; blank = never (P08) | Use with care: treatment-effect analysis only |

## Derived variables used in the EDA

Built in `notebooks/EDA_Individual_Kate_Klinger.qmd`, using only information available by the end of FY *t*.

| Variable | Role | Definition |
|---|---|---|
| `fy` | All transaction tables | Fiscal year: year of date, +1 if month ≥ July (FY2026 = Jul 2025–Jun 2026) |
| `giving (amount)` | Payments + soft credits | Donor fiscal-year giving: positive personal payments + deduplicated positive soft credits; org payments and defective copies excluded |
| `eligible` | Donor-year panel | Ever gave (Unite or legacy) through FY t and gave under \$1,200 in FY t; includes lapsed donors |
| `status` | Feature / segment | Active (gave in FY t), Lapsed 1–2 yrs, Lapsed 3–5 yrs, Lapsed 6+ yrs |
| `years_since_last_gift` | Feature | FY t minus the last fiscal year with any gift (0 = active) |
| `last_year_amount` | Feature | Giving in the most recent fiscal year with a gift |
| `conv_12m` | Target | Giving ≥ \$1,200 in FY t+1 (0.046% of eligible donor-years; 0.17% for active donors) |
| `conv_3yr / conv_5yr` | Target | ≥ \$1,200 in any of FY t+1..t+3 / t+1..t+5 (snapshots FY2021–23 / FY2021 only) |
| `amount_t, amount_3yr` | Feature | Giving in FY t; total giving FY t-2..t (legacy household credit before FY2021) |
| `max_gift` | Feature | Largest single payment or soft credit in FY t |
| `former_major` | Feature | Any earlier year with giving ≥ \$1,200 (Unite or legacy) |
| `years_gave_last5` | Feature | Number of FY t-4..t with any giving |
| `tenure_years` | Feature | FY t minus first fiscal year with any gift (legacy included) |
| `any_recurring` | Feature | Any recurring payment in FY t (replaces the leaking sustainer flag) |
| `any_upgrade_gift` | Feature | Any payment with gift_type 'Upgrade' in FY t |
| `soft_credit_t / soft_credit_ever` | Feature | Any soft credit in FY t / up to FY t |
| `age_at_t` | Feature | Age − (2026 − t) |
| `n_solicited, n_responded` | Candidate feature | Solicitations and responses in FY t |
| `titles`, `genres`, `pct_watched`, `viewed` | Candidate feature | Passport titles, genres, and share watched in FY t; `viewed` flags any record (blank = no record, not zero) |
