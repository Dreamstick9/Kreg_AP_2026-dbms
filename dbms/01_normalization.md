# Kreg Lite database normalization

Kreg Lite lets users save websites as monitors. The app checks each monitor and stores
the result. After repeated failures, it can also store an incident. The app's schema is
in the [Kreg Lite repository](https://github.com/Dreamstick9/Kreg_AP_2026).

The design below follows the same four things that the app stores: users, monitors,
check results and incidents.

## Starting point: unnormalized data

Imagine one row per user, with a list of that user's monitors. Each monitor also has a
list of checks and a list of incidents. These lists are repeating groups, so values are
not stored in separate, atomic rows. Finding or updating one check would require working
through a list.

## First normal form (1NF)

1NF means each cell has one value and each row has a key. Make one row for each check
and one row for each incident. Keep the two event lists separate.

For this step, the relations are:

- `CHECK_LOG(monitor_id, checked_at, is_up, status_code, response_time_ms, error, user_id, user_name, email, password_hash, user_created_at, monitor_name, url, interval_sec, is_active, is_public, status, consecutive_failures, last_checked_at, monitor_created_at)`
- `INCIDENT_LOG(monitor_id, started_at, reason, resolved_at, user_id, user_name, email, password_hash, user_created_at, monitor_name, url, interval_sec, is_active, is_public, status, consecutive_failures, last_checked_at, monitor_created_at)`

For this explanation, assume a monitor cannot have two events with the same timestamp.
The example keys are `(monitor_id, checked_at)` for a check and
`(monitor_id, started_at)` for an incident. The final app schema uses generated `id`
keys for these event tables and does not declare the timestamp pairs as unique.

Functional dependencies:

- `(monitor_id, checked_at) -> is_up, status_code, response_time_ms, error`
- `(monitor_id, started_at) -> reason, resolved_at`
- `monitor_id -> user_id, monitor details`
- `user_id -> user details`

Monitor details repeat on every event row. They depend on only part of each event key;
user details are reached through `user_id`.

## Second normal form (2NF)

2NF means 1NF, with no non-key field depending on only part of a composite key. Move
monitor details out of the check and incident rows:

- `MONITORS_2NF(monitor_id PK, user_id, user_name, email, password_hash, user_created_at, name, url, interval_sec, is_active, is_public, status, consecutive_failures, last_checked_at, created_at)`
- `CHECK_RESULTS_2NF(monitor_id, checked_at, is_up, status_code, response_time_ms, error)`; key `(monitor_id, checked_at)`
- `INCIDENTS_2NF(monitor_id, started_at, reason, resolved_at)`; key `(monitor_id, started_at)`

The event tables now contain only their own event data. In `MONITORS_2NF`, the key
determines the monitor and its `user_id`, while `user_id` determines the user's name,
email, password hash and creation time. That is still a transitive dependency.

Functional dependencies:

- `monitor_id -> user_id, monitor details`
- `user_id -> user_name, email, password_hash, user_created_at`
- `(monitor_id, checked_at) -> is_up, status_code, response_time_ms, error`
- `(monitor_id, started_at) -> reason, resolved_at`

## Third normal form (3NF)

3NF means 2NF, with no non-key field depending on another non-key field. Move user
details into their own table. The resulting tables are:

| Table | Columns and key |
|---|---|
| `users` | `id` (PK), `name`, `email` (unique), `password_hash`, `created_at` |
| `monitors` | `id` (PK), `user_id` (FK), `name`, `url`, `interval_sec`, `is_active`, `is_public`, `status`, `consecutive_failures`, `last_checked_at`, `created_at` |
| `check_results` | `id` (PK), `monitor_id` (FK), `is_up`, `status_code`, `response_time_ms`, `error`, `checked_at` |
| `incidents` | `id` (PK), `monitor_id` (FK), `reason`, `started_at`, `resolved_at` |

Functional dependencies in the final tables:

- `users.id -> name, email, password_hash, created_at`; `email` is also unique.
- `monitors.id -> user_id, name, url, interval_sec, is_active, is_public, status, consecutive_failures, last_checked_at, created_at`
- `check_results.id -> monitor_id, is_up, status_code, response_time_ms, error, checked_at`
- `incidents.id -> monitor_id, reason, started_at, resolved_at`

Each table now describes one type of record. The foreign keys connect a user to their
monitors, and a monitor to its check results and incidents. The app keeps the monitor's
current status, failure count and last-check time for quick reads; the checker updates
them along with each result.

The final keys and column types are defined in `03_schema.sql`. The app currently uses
generated `id` primary keys for check results and incidents, as shown in its Prisma
schema.
