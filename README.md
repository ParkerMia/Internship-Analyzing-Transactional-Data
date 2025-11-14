# Transaction Risk Classification

**Language:** Python  
**Libraries:** `pandas`, `numpy`, `matplotlib`

---

## Data Preparation
### Loading & Cleaning Data
- Imported two datasets: an alerts export and a transactions export.
- Cleaned missing columns, removed rows with incomplete state information, and filtered alerts older than 30 days.
- Extracted structured features (`days_old`, `time`) from the alert age field.
- Expanded multi-transaction alert IDs into individual rows for accurate merging.
- Merged cleaned alert and transaction data to create a unified working dataset.

### Feature Filtering
- Converted `tx_date_time` into a datetime object and extracted the day of week.
- Cleaned and normalized sender names, then assigned each business a unique ID.
- Created a filtered dataset of **Not Suspicious + Soft Stop** transactions to serve as baseline behavioral data.

---

## Behavioral Modeling
Two baseline summary tables were built:

### Step 1 (Sender-Based Patterns)
For each sender:
- Count of transactions to each counterparty  
- Median transfer amount  
- Median absolute deviation (MAD)  
- Most common day of week for transactions  

Only counterparties with more than two historical interactions were included.

### Step 2 (Receiver-Based Patterns)
Aggregated across senders:
- Count  
- Median amount  
- MAD  
- Mode day of week  

Also limited to counterparties with sufficient historical activity.

---

## Risk Classification
### Transaction-Level Risk Groups
Each **In Review + Soft Stop** transaction is evaluated using a multi-step rule function:

- **Group 1:** Matches sender-level behavioral patterns (amount within median+MAD and day matches mode)
- **Group 2:** Fails sender check but matches receiver-level patterns
- **Group 3:** Does not match any known patterns or appears anomalous

Classified transactions are stored in DataFrames `df1`, `df2`, and `df3`, then combined into a single dataset called `all_tx`.

---

## Alert-Level Classification
Alerts are evaluated based on their underlying transactions:

- **Alert Group 1:** All transactions are Group 1  
- **Alert Group 3:** At least one transaction is Group 3  
- **Alert Group 2:** A mix of Group 1 and Group 2  

Three final lists (`alert_group_1`, `alert_group_2`, `alert_group_3`) store the grouped alert IDs.

---
