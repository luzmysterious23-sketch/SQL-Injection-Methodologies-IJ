# SQL Injection Methodologies

**Author:** Isabelle Jean  
**Status:** All three tasks documented; supporting result screenshots pending  
**Focus:** Web application security and SQL injection

## Project Overview

This guided cybersecurity lab explores SQL injection in Damn Vulnerable Web Application (DVWA) and password-hash cracking with John the Ripper. The planned workflow covers checking that the training application is running, conducting the lab's SQL injection exercise, and testing captured password hashes.

## Objective

Develop hands-on skills in identifying and exploiting SQL injection within the provided training environment, then examine how exposed password hashes can create additional account-security risks.

## Exercise 1 — Database Attack Types and Exploits

| Stage | Learning objective |
| --- | --- |
| Application verification | Verify that DVWA is running |
| SQL injection | Conduct the SQL injection exploit described in the lab |
| Password-hash analysis | Crack captured lab password hashes using John the Ripper |

[Read the full Exercise 1 portfolio write-up](exercises/exercise-01-database-attacks.md), covering all three tasks, corrected commands, evidence captions, and security lessons.

The supplied XAMPP screenshot confirms service startup. Later outcomes are documented from the lab instructions; additional execution screenshots are pending.

## Tools and Environment

- **DVWA:** The intentionally vulnerable web application used for the exercise.
- **John the Ripper:** The password-cracking tool specified for analyzing captured lab hashes.

The lab uses PLABDC01 (Windows Server 2019 domain controller), PLABDM01 (Windows Server 2019 application host), and PLABKALI (Kali Linux workstation). XAMPP provides the web and database services.

## Exam Alignment

**Web application hacking — SQL injection**

## Scope

All testing described in this repository is limited to the authorized practice-lab environment and its provided applications and data.
