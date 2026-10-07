---
name: qa-db-verification
description: "Writes precise, read-only SQL that a tester can run to verify test results in a database (PostgreSQL, SQL Server, MySQL): finds the right tables and column names from the code or schema first, filters to the exact test records, compares a working and a failing case side by side, counts impact, and explains how to read the result. Also writes cleanup queries only on request, with a preview first. Use when the user says \"give me a query\", \"simple query\", \"check it in the DB\", \"how many records are affected\", \"find a record that has no link\", or pastes query results to interpret."
---

# QA DB Verification

Queries are evidence. A wrong column name wastes the user's time and a careless
query can change data. Be exact and safe.

## Before writing a query

1. **Find real table and column names** from the code (ORM entities, mappings,
   migrations, repository SQL) or the schema. Note naming conventions
   (snake_case vs PascalCase, schemas like `v1.`). Never guess a column; if you
   cannot confirm it, use `SELECT *` with a tight filter.
2. Know which **database and engine** the user is on. Syntax differs:
   `ILIKE` and `::` casts are PostgreSQL; `TOP`, `GETDATE()`, `DATEADD` are SQL
   Server; identifiers may need quoting.
3. Know the **tenant/organisation key** so results are not mixed across tenants.

## Writing it

- **Read-only by default.** `SELECT` only.
- Filter tightly: exact ids, test-name prefix, created in the last N minutes.
- Return the columns that prove the point, plus audit columns
  (`created_on`, `created_by`, `modified_by`, `deleted_on`) when relevant.
- Include soft-delete filters where the model uses them.
- For "has no X" questions use `NOT EXISTS`, not a `LEFT JOIN` that people misread.
- For impact, add a grouped `COUNT(*)` query.
- **Side-by-side control:** one query returning the working and the failing
  record together makes the difference obvious.
- If a script has two statements and one fails to compile, the engine may run
  neither; say "run both again" after fixing.

## Templates

```sql
-- Find test records
SELECT id, name, created_on FROM <schema.table>
WHERE name LIKE 'QaTest%' ORDER BY created_on DESC;

-- Records missing a link
SELECT p.id, p.name FROM <parents> p
WHERE NOT EXISTS (SELECT 1 FROM <links> l
                  WHERE l.parent_id = p.id AND l.type = '<type>' AND l.deleted_on IS NULL);

-- Format/impact breakdown
SELECT type, CASE WHEN key LIKE '%:%' THEN 'composite' ELSE 'plain' END AS format, COUNT(*)
FROM <links> GROUP BY 1, 2 ORDER BY 1, 2;
```

## Reading results back

- Say what each row means in one line, then the verdict (pass, bug, data issue).
- Watch for display quirks: empty fixed-width columns show as spaces, `NULL`
  vs empty string, time zone of timestamps.
- If the result contradicts an earlier conclusion, say so and correct it.

## Cleanup (only when asked)

- First a `SELECT` preview of exactly what will be removed, with a count.
- Then the `DELETE`/`UPDATE` in a transaction, scoped by ids from the preview.
- Never on production. Prefer asking the dev or using the app's own delete.
- Get explicit confirmation before giving the destructive statement.
