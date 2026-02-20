SQL Injection simply allows an attacker to interfere with the queries being executed in the backend, that an application makes to its interfaces.

## Impact
- Passwords
- Credit Card Details
- Personal user Information

## How to detect SQLi vulnerabilities?
- A single `'` quote character, that prints some errors or anomalies
- Boolean conditions such as `1=1` or `1=2` that changes application behavior
- OAST payloads designed to trigger an out-of-band network interaction when executed within a SQL query, and monitor any reaction

## Where does SQL Injection occur?
- In `UPDATE` statements, within the updated values or the `WHERE` clause.
- In `INSERT` statements, within the inserted values.
- In `SELECT` statements, within the table or column name.
- In `SELECT` statements, within the `ORDER BY` clause.

