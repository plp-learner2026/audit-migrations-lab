\# Week 7 Lab: Audit Logging, Category Trees, Migrations and Security (PostgreSQL 18, Windows)



\## Step 1: Audit log

\- Created `audit\_log`, the `audit()` trigger function and trigger `trg\_audit` on `students`.



\## Step 2: Audit trail in action

\- `UPDATE students SET name = 'Kofi M.' WHERE id = 1;`

\- `DELETE FROM students WHERE id = 3;`

\- `audit\_log` recorded each change as JSON: UPDATE rows showed old and new names, and the DELETE row had a null new value.



\## Step 3: Category tree

\- Created a self-referencing `categories` table with 4 rows.

\- The recursive CTE returned all 4 rows by depth: Electronics, then Computers and Phones, then Laptops.



\## Step 4: Flyway migrations

\- Flyway 10.17.0 applied `V1\_\_core\_tables.sql`, `V2\_\_audit\_log.sql` and `V3\_\_categories.sql` to a fresh database `bootcamp\_flyway`.

\- Output: `Successfully applied 3 migrations ... now at version v3`.

\- `flyway info` showed V1, V2 and V3 all as `Success`.

\- Command used:

&#x20; `flyway -url=jdbc:postgresql://localhost:5432/bootcamp\_flyway -user=postgres -password=<password> -locations=filesystem:C:\\flyway-lab\\migrations migrate`

\- The migration files are in the `migrations/` folder.



\## Step 5: Least-privilege security

\- Created roles `app\_read` (SELECT only) and `app\_write` (SELECT, INSERT, UPDATE, DELETE).

\- Granted CONNECT on the database and USAGE on schema `public`.

\- Created login user `api` as a member of `app\_write`.

\- Verified in `pg\_roles`: `api` can log in; `app\_read` and `app\_write` cannot.



\## Conclusion

Change tracking, hierarchical data modelling, versioned migrations and least-privilege access control were all implemented and checked.



\## Fix after feedback: sequence permissions (V4)

\- Feedback: `app\_write` could not insert into tables with SERIAL columns, because it had no permission on the underlying sequences.

\- Fix: added a new versioned migration, `V4\_\_sequence\_grants.sql`, instead of editing V1 to V3:

&#x20; - `GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO app\_write;`

&#x20; - default privileges so future tables and sequences are covered too.

\- Applied with Flyway: schema is now at version 4 (`flyway info` shows V1 to V4 as Success).

\- Verified: `SET ROLE api; INSERT INTO categories (name, parent\_id) VALUES ('Tablets', 1);` succeeded.

