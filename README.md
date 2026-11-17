# Kreg AP 2026 — DBMS files

This repository holds the DBMS deliverables for the Kreg Lite application built in
the AP project. The application code and Prisma schema are in the
[Kreg Lite app repository](https://github.com/Dreamstick9/Kreg_AP_2026).

The files in `dbms/` use that app's four PostgreSQL tables: `users`, `monitors`,
`check_results` and `incidents`.

## Deliverables

- `dbms/01_normalization.md` explains the tables through 1NF, 2NF and 3NF.
- `dbms/02_er_diagram.png` is the ER diagram; the `.drawio` file is its editable source.
- `dbms/03_schema.sql` creates the same enum, tables, indexes and foreign keys as the
  app's Prisma migration.

The CP2 baseline benchmark files belong in the team's DE repository, as listed in the
course checkpoint instructions.
