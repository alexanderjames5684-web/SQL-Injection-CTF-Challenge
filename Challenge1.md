🔐 SQL Injection Walkthrough: VoltEdge Tracker (CTF Challenge 1) 📌 Challenge Overview

The VoltEdge Tracker asset search application appears simple at first glance, but hints suggest hidden database content. The objective was to determine whether sensitive information could be extracted through SQL injection.

🔗 Target: https://ctf.kd8bfn.org/sqli/search

🧠 Initial Observations

After exploring the application, the About page provided a key clue:

The system had been migrated from Access DB → SQLite

This immediately influenced the attack strategy since SQLite uses different schema tables (e.g., sqlite_master instead of information_schema).

🚨 Identifying the Vulnerability

Testing basic input in the search bar:

'--

Result:

The application returned all records This confirmed a SQL injection vulnerability 🔢 Determining Column Count

By analyzing the table output, it was determined that the query returns 6 columns, which is necessary for constructing valid UNION SELECT statements.

🧪 Crafting the Injection Step 1: Match Column Count

To align with the query structure:

' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL FROM system_config--

This verifies that UNION SELECT is working correctly.

Step 2: Discover Table Structure

Since SQLite is being used, schema information is stored in sqlite_master.

' UNION SELECT sql,NULL,NULL,NULL,NULL,NULL FROM sqlite_master WHERE name='system_config'--

This reveals the table structure:

CREATE TABLE system_config ( id INTEGER PRIMARY KEY, config_key TEXT, config_value TEXT ) Step 3: Extract Sensitive Data

With column names identified, the next step is to retrieve the contents:

' UNION SELECT config_key,config_value,NULL,NULL,NULL,NULL FROM system_config-- 🎯 Results

This query exposed several internal configuration values, including:

License key Backup schedule Maintenance window Application version 🚩 CTF Flag 🏁 Final Flag CYBR3200{union_select_the_unseen_a7c3} 🧠 Key Takeaways Always check metadata pages (like "About") for hints about backend technologies SQLite requires different enumeration techniques (sqlite_master) Matching column count is critical for successful UNION attacks Misconfigured input validation can expose sensitive internal data Even simple search fields can be high-risk attack vectors 🔐 Security Lessons

This challenge highlights the importance of:

Using parameterized queries / prepared statements Proper input sanitization Limiting database permissions Avoiding exposure of internal system details in public-facing pages 💭 Final Thoughts

This was a great example of how small clues + methodical testing can lead to full database compromise. Once the database type was identified, the rest became a structured process of enumeration and extraction.
