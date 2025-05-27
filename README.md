# CPO-dashboard

# 📊 RDMP CPO Field Completion Tracker Dashboard

## 📌 Purpose
This Splunk dashboard provides a centralized view to **track the completeness** of required CPO fields across RDMP initiatives. It highlights which RDMP entries are **pending vs complete**, which PMs are responsible, and what specific fields are missing.

---

## 📁 Data Source

- **Primary Lookup File:** `rdmp_cpo_malini.csv`
- **Key Fields Used:**
  - `Key`: Unique RDMP identifier
  - `Updated`: Timestamp of last update
  - Multiple CPO-related fields (e.g., `Target Customer Segmentation`, `Partner Value`, etc.)
  - `PM`, `Product_Area`, `Issue Type`, `Status`, `ddate`

---

## 🔍 Base Search Logic

**Search ID:** `completion_base`

### Logic Overview:
- Filters to most recent records per `Key` using the `Updated` timestamp
- Trims and flattens multi-value fields (`Initiative_Status`, `ddate_history`)
- Filters initiatives where:
  - `ddate > 2025-05-01`
  - `Status` is `"New"`, `"In Progress"`, or `"Backlog"`
- Fills nulls in `Launch_Phase` and `Launch_Tier` with `"None"`
- Filters using user-selected filters via tokens:
  - `$pm_filter$`, `$product_area_filter$`, `$completion_status_filter$`, `$launch_tier_filter$`
- Assigns:
  - `Completion_Status = "Pending"` if **any** required CPO field is null/empty
  - `Completion_Status = "Complete"` otherwise
- Builds a list of `Missing_Fields` using `mvappend`

### Key Derived Fields:
- `Completion_Status`
- `Missing_Fields`
- Cleaned/trimmed `PM`, `Product_Area`

---

## 🎛 Filters

| Filter             | Token                   | Description                                   |
|--------------------|--------------------------|-----------------------------------------------|
| Product Manager    | `$pm_filter$`            | Multi-select of unique PM values              |
| Product Area       | `$product_area_filter$`  | Multi-select of Product Area values           |
| Completion Status  | `$completion_status_filter$` | Pending / Complete                           |
| Launch Tier        | `$launch_tier_filter$`   | Tier 1 or Tier 2 (hidden via `depends`)       |

> ℹ️ **Note**: Issue Type filter is present in earlier versions but currently not included in the base search. Can be re-added as needed.

---

## 📈 Panels Overview

### 📊 Completion Status (Pie Chart)
- Visual representation of `Pending` vs `Complete` RDMPs
- Percent breakdown shown on chart

### 📋 Completion Status Breakdown (Table)
- Table view of counts and percentages by `Completion_Status`

### 👤 PM Completion Summary (Table)
- Shows each PM’s:
  - `Completed` count and RDMP list
  - `Pending` count and RDMP list

### ⚠️ Pending RDMPs with Missing Fields
- Displays RDMPs with `Completion_Status = "Pending"`
- Lists:
  - `Key`
  - `Missing_Fields`
  - `url` (clickable link to Jira)

---

## 🔗 Drilldown Behavior

- **Panel:** Pending RDMPs with Missing Fields
- **Action:** Clicking the `url` field opens the Jira ticket in a new tab.
- **Link format:** `$row.url|n$`

---

## 🧠 Additional Notes

- Completion logic checks both `isnull(...)` and `=""` to capture blanks
- Deduplication and recent timestamps ensure accuracy
- Multivalue cleanup ensures readable values in dropdowns
- Can be extended to include additional fields or dimensions (e.g., `Issue Type`)

---

## 🛠 Maintenance Tips

- **To Add More Completion Fields:**
  - Update the `eval Completion_Status=case(...)` and `eval Missing_Fields=mvappend(...)` blocks

- **To Add a New Filter:**
  - Add `<input>` in the `<fieldset>` and add `| search $token$` in the base query

- **To Replace the Source File:**
  - Change `inputlookup rdmp_cpo_malini.csv` to the new CSV

---

## 📎 Related Resources

- Jira Dashboard: [Splunk CPO Dashboard](https://splunk.atlassian.net/jira/dashboards/50791)

---

