# Lecture 12 — Databases and PostgreSQL for Backend Engineers
**Series:** Backend Engineering from First Principles  
**Focus:** Database fundamentals, PostgreSQL schema design, migrations, API queries, indexes and triggers  

> These notes follow the supplied lecture transcript. They organize the concepts and add explanatory examples where needed for study; they are not a substitute for practicing SQL and PostgreSQL directly.

## Learning outcomes
- Explain persistence, databases and DBMS responsibilities.
- Compare relational and document databases and understand the lecture's PostgreSQL choice.
- Select common PostgreSQL data types.
- Model entities and relationships with keys and constraints.
- Use migrations to version schema changes.
- Build safe API queries with joins, filtering, sorting and pagination.
- Understand indexes, their trade-offs and database triggers.

## 1. Why backend systems need databases

### Persistence
A database provides persistence: data survives after the program that created it stops. A to-do app should show the same tasks and completion state when a user closes and reopens it.

### Database — broad meaning
At a high level, any structured persistent storage that supports creating, reading, updating and deleting data can be called a database. Examples mentioned in the lecture include phone contacts, browser local storage, session storage, cookies and even a text file of notes.

### Backend-engineering meaning
In typical backend discussions, “database” usually means a disk-based database system: data is stored on secondary storage such as HDDs or SSDs, with software providing efficient operations over it.

## 2. RAM, disk and caching

### RAM / primary memory
RAM is fast to read and write, but relatively expensive and limited in capacity. It is commonly used by caches such as Redis or in-process memory caches.

### Disk / secondary storage
HDDs and SSDs provide much larger capacity at lower cost per unit of storage, but access is slower than RAM. Traditional relational and non-relational databases persist their data on disk.

### Backend takeaway
Use persistent databases for durable, larger-scale data. Use caching to keep frequently accessed data in faster memory when the performance trade-off is worthwhile. A cache is not automatically a replacement for durable storage.

## 3. What a DBMS does

### Definition
A Database Management System (DBMS) is software that stores data and exposes efficient operations for creating, reading, updating and deleting it (CRUD).

### Core responsibilities
Data organization; access through query or client operations; integrity (validity, accuracy and consistency); and security against unauthorized access. DBMS products also provide capabilities for scaling, availability and managing concurrent clients.

### Integrity example
If an order payment amount is a numeric field, the database should reject an attempt to store arbitrary text in that field.

## 4. Why plain text files are not enough

### Parsing cost and fragility
To find or update a customer in a text file, application code must parse the file, split records and compare fields. This is inefficient at scale and creates opportunities for bugs or incorrect data.

### No enforced structure
A plain text file does not inherently enforce column types, required fields or relationships. Different records can drift into inconsistent formats.

### Concurrency problem
Two clients can read the same old value, independently calculate updates and overwrite each other. For example, both read 40; one writes 60 after adding 20, while another writes 20 after subtracting 20. A naive file workflow can lose one update.

### Why DBMS
A DBMS supplies structured data handling and concurrency mechanisms so simultaneous operations can be coordinated more safely.

## 5. Relational vs non-relational databases

### Relational databases
Organize data into tables, rows and columns. Tables have predefined schemas and relationships are represented using keys, commonly foreign keys. SQL is the standard language used to query relational databases.

### Non-relational / NoSQL databases
Use models such as documents and collections. In the MongoDB example from the lecture, a collection contains documents; documents in the same collection may have different shapes.

### Trade-off
Relational systems enforce a stronger schema and constraints at the database layer, helping preserve integrity. Document systems offer flexible structures and can make evolving or heterogeneous content convenient, but application code may need to enforce more consistency.

### Use-case examples from the lecture
A CRM with customers, contacts and sales opportunities benefits from relational structure and integrity. A CMS storing articles with varied content blocks, images, code snippets and embedded videos can benefit from a flexible document model.

### Important nuance
Flexible schema does not mean “no schema” or “no validation.” It means the database may enforce fewer structural rules by default; application-level validation and thoughtful document design still matter.

## 6. Why the lecture chooses PostgreSQL

### Open source
PostgreSQL is open source and can be self-hosted.

### SQL support
It supports standard SQL concepts, which can make moving between relational systems easier, although database-specific features and syntax can still require migration work.

### Extensibility and capability
It has a broad feature set and an extension ecosystem.

### Reliability and scalability
The lecture highlights PostgreSQL's reputation for reliability and its ability to scale for many application workloads.

### JSON support
PostgreSQL supports JSON and JSONB, including indexing and querying JSON fields. This can cover many dynamic-content use cases without automatically requiring a separate NoSQL database.

### Practical recommendation
For many typical projects, PostgreSQL is a strong default. Do not prematurely choose a database based on theoretical performance differences; first identify a real workload or bottleneck.

## 7. SQL, PostgreSQL and the tooling boundary

### SQL
Structured Query Language: the language used to define, query and modify relational data.

### PostgreSQL
The database management system that executes SQL statements and stores the data.

### Drivers
Backend applications commonly interact with a database through a language-specific driver rather than having users manually run SQL in a graphical tool.

### TablePlus
The lecture uses TablePlus as a graphical interface for inspecting tables and running queries. A GUI is useful for exploration, but production schema changes should be tracked through migrations.

## 8. PostgreSQL data types covered

### Integer family
SMALLINT, INTEGER and BIGINT are integer types with increasing ranges. SERIAL and BIGSERIAL are auto-incrementing integer-style column declarations often used for generated identifiers. A primary key uniquely identifies a row.

### Exact numeric
NUMERIC and DECIMAL are exact numeric types. Use them for values where decimal accuracy matters, such as money or prices. A declaration such as NUMERIC(10,2) allows precision 10 with scale 2: up to 10 total digits, 2 after the decimal point.

### Floating point
REAL and DOUBLE PRECISION use approximate floating-point representation. They are useful for scientific or measurement values where tiny representation differences are acceptable and floating-point performance/range is useful. Do not use them for currency calculations that require exact decimal arithmetic.

### Text types
CHAR(n) is fixed-width and may pad values with spaces. VARCHAR(n) stores variable-length text up to a limit. TEXT stores variable-length text without an application-chosen length limit (subject to PostgreSQL's system limit). The lecture recommends TEXT for ordinary text fields and enforcing product-specific length rules in application validation when appropriate.

### Boolean and temporal
BOOLEAN stores true/false. DATE stores a calendar date; TIME stores time of day; TIMESTAMP stores date and time; TIMESTAMPTZ represents a timestamp with time-zone-aware semantics; INTERVAL stores a duration such as one week or ten days.

### UUID
UUID is a native identifier type. It is commonly used for primary keys when globally unique identifiers are useful.

### JSON and JSONB
JSON stores JSON text with validation. JSONB stores a decomposed binary representation that is generally better suited to querying and indexing. The lecture recommends JSONB for most PostgreSQL JSON use cases, while noting it is PostgreSQL-specific.

### Arrays and specialized types
PostgreSQL also supports arrays and specialized types such as network addresses, MAC addresses, geometric values and XML. Choose them only when the data model benefits from their semantics.

## 9. Database migrations

### Definition
A migration is a versioned file containing database schema changes, commonly SQL such as CREATE TABLE, ALTER TABLE, CREATE INDEX or DROP TABLE.

### Why migrations
They make schema evolution reproducible, reviewable and version-controlled. Migration files live with the application code and are committed to Git.

### Migration tool workflow
A command-line tool reads migration files in a deterministic order, applies unapplied changes to the configured database and records the current schema version in a migration metadata table.

### Up migration
Applies a change, for example creating a table, enum type, index or trigger.

### Down migration
Attempts to reverse the corresponding change so a prior schema state can be restored. Not every modern migration workflow relies on down migrations; rollback strategy must be planned carefully.

### Tools named in the lecture
The transcript mentions dbmate and golang-migrate. The demo uses dbmate, a database URL supplied through an environment variable, and a DB/migrations directory.

### Production caution
A down migration that drops data may be destructive. A successful rollback of schema objects does not necessarily restore deleted or transformed data. Backups and deployment/rollback planning remain important.

## 10. Modeling a project-management database

### Core entities
The lecture builds a project-management platform around users, user profiles, projects, tasks and project membership. The schema is designed to support APIs created in the preceding REST API lesson.

### Enums
PostgreSQL ENUM types restrict a column to a predefined set of values. Examples include project status (active, completed, archived), task status (pending, in progress, completed, cancelled) and member role (owner, admin, member).

### Why enums
They move allowed-value enforcement into the database, improve integrity and make the domain vocabulary visible in the schema. They also act as lightweight documentation. Changing enum values later requires a deliberate migration.

### Primary keys
A primary key uniquely identifies each row. The lecture discusses generated integer identifiers and UUIDs as common choices.

### Foreign keys
A foreign key references a key in another table and prevents references to nonexistent rows. For example, a task's assigned user must exist in the users table.

### One-to-one profile
A user profile can use user_id as both its primary key and foreign key to users.id. This models at most one profile per user. Optional profile fields such as image URL, bio and phone number can be nullable.

### One-to-many
A user can own multiple projects; a project can contain multiple tasks. A foreign key on the “many” side represents the relationship.

### Many-to-many
Users can belong to multiple projects and projects can have multiple users. A join table such as project_members stores one row per user-project membership, with a composite key or other uniqueness rule preventing duplicate memberships.

### Nullability and defaults
Use NOT NULL for required fields. Use nullable columns for genuinely optional data. Defaults can provide values such as created_at timestamps or initial status, but defaults should reflect domain rules.

## 11. Referential integrity and delete behavior

### Foreign-key integrity
A foreign key ensures that a referenced parent row exists. A syntactically valid UUID is not enough: the UUID must identify an actual row in the referenced table.

### ON DELETE RESTRICT
Prevents deleting a parent row while dependent rows still reference it. The lecture uses this to prevent deleting a user who still owns associated projects.

### Other common actions
PostgreSQL also supports actions such as CASCADE, SET NULL and SET DEFAULT. Choose them based on lifecycle semantics, not convenience.

### Design question
If a user is deleted, should their projects be deleted, reassigned, anonymized or should deletion be blocked? The correct constraint follows the product's data-retention and ownership rules.

## 12. Seeding and migration version tracking

### Seeding
Seeding inserts initial or development/test data so the application can be exercised against realistic records.

### Version tracking
Migration tools maintain a schema-version record so they know which migrations have already run. This prevents blindly re-running the same schema changes.

### Environment discipline
Use the intended database URL for each environment. Keep credentials out of source control and ensure migrations are applied consistently in development, staging and production.

## 13. Building API queries from the schema

### List users
A basic list endpoint can select user rows, often joining the profile table when the API response needs profile information.

### Joins
A JOIN combines related rows using a condition such as users.id = user_profiles.user_id. Use explicit join conditions and select only the fields required by the response.

### Nested JSON responses
PostgreSQL JSON functions such as json_build_object / jsonb_build_object and row-to-JSON helpers can shape related database rows into nested JSON objects. The lecture demonstrates returning a profile object alongside user data.

### Single-resource lookup
For a route such as GET /users/:id, use the route parameter as a query parameter to find the corresponding user. Handle the not-found case in the application layer.

### Parameterized queries
Pass user-supplied values as bound parameters through the driver. Do not concatenate raw user input into SQL strings. Parameterization prevents values from being interpreted as SQL syntax and is a key defense against SQL injection.

### Dynamic sorting
Column names and sort directions generally cannot be treated like ordinary value parameters. Map requested sort keys to a strict allowlist of known SQL identifiers, and allow only ASC or DESC. Never interpolate arbitrary client-provided column names.

### Dynamic filtering
Build optional WHERE conditions from validated filters. Each filter value should be bound as a parameter. Keep a clear allowlist of supported filters and define their semantics.

### Pagination
Use LIMIT and OFFSET (or a cursor/keyset strategy for large or rapidly changing datasets). The lecture demonstrates page and limit parameters; the usual offset formula for one-based page numbers is (page - 1) × limit.

### Stable ordering
Pagination should use a deterministic ORDER BY, ideally including a unique tie-breaker. Without stable ordering, records can appear on multiple pages or be skipped when values tie or data changes.

### API-to-query workflow
Inspect the endpoint's route parameters, query parameters and body; validate and normalize them; select the required fields and joins; bind values safely; apply allowlisted filters/sorts and pagination; execute; map the result into the API response.

## 14. Indexes and query performance

### What an index does
An index is an auxiliary data structure that helps PostgreSQL locate matching rows without scanning the entire table for every query. It can accelerate filters, joins and ordering when it matches the workload.

### Index candidates
Frequently filtered columns, join keys and columns used for common ORDER BY patterns are candidates. The lecture adds indexes for common user-list sorting/filtering and for a task foreign key used in joins.

### Foreign keys are not automatically indexed
PostgreSQL automatically indexes primary keys and unique constraints, but it does not automatically create an index on every referencing foreign-key column. Add one when query patterns justify it.

### Composite indexes
A multi-column index can support queries that filter or sort on a combination of columns. Column order matters; design it around the actual WHERE and ORDER BY patterns.

### Costs and trade-offs
Indexes consume storage and add work to INSERT, UPDATE and DELETE because index entries must be maintained. Too many indexes can slow writes and increase operational cost.

### Measure, don't guess
Use representative data and query plans (for example, EXPLAIN / EXPLAIN ANALYZE) to evaluate whether an index helps. The lecture's key principle is to decide whether read-performance gains justify index-maintenance overhead.

## 15. Triggers and automatic updated_at timestamps

### Trigger
A database trigger runs a specified function automatically when a configured database event occurs, such as INSERT, UPDATE or DELETE.

### updated_at example
The lecture creates a trigger function that sets NEW.updated_at to the current timestamp, then attaches an UPDATE trigger to relevant tables. Each update automatically refreshes the timestamp for the changed row.

### Why use it
It centralizes timestamp maintenance in the database, so every update path gets the same behavior, including updates that do not pass through a particular application code path.

### Down migration
The reverse migration removes the triggers and trigger function, along with indexes created by the migration.

### Caution
Triggers are powerful but can make side effects less visible to application developers. Name and document them clearly, keep them focused, and test their behavior.

## 16. Practical backend database checklist

### Before modeling
Identify entities, ownership, relationships, lifecycle rules, required fields, optional fields and query patterns implied by the APIs.

### While defining schema
Choose suitable types; add primary keys, foreign keys, unique constraints, NOT NULL rules, defaults and enums where justified.

### While implementing endpoints
Validate input; use parameterized values; allowlist dynamic identifiers; select only needed fields; join on indexed relationship columns when justified; paginate lists.

### Before shipping
Create versioned migrations; review both apply and rollback behavior; seed test data; verify constraints; inspect query plans; add only useful indexes; keep secrets outside Git.

### Operational thinking
Monitor slow queries and database load. Revisit indexes and query shape when real usage reveals bottlenecks rather than optimizing based only on guesses.

## 17. Interview questions and concise answers

### Why do we need a database instead of a text file?
A DBMS provides structured storage, efficient querying, integrity constraints, concurrency control and access management that are difficult to implement reliably with plain files.

### What is the difference between SQL and PostgreSQL?
SQL is a query language; PostgreSQL is a DBMS that stores data and executes SQL.

### Relational vs NoSQL?
Relational databases use structured tables and relationships with strong database-level constraints. Document databases support flexible document structures, with more integrity rules often handled by the application.

### Why PostgreSQL?
It is open source, feature-rich, extensible, reliable, scalable for many workloads and has strong JSON/JSONB support.

### NUMERIC vs FLOAT?
NUMERIC/DECIMAL provides exact decimal arithmetic; floating-point types are approximate and are better suited to scientific/measurement workloads where small representation errors are acceptable.

### CHAR vs VARCHAR vs TEXT?
CHAR is fixed-width; VARCHAR(n) is variable-width with a maximum; TEXT is variable-width without a chosen column length limit. PostgreSQL generally has no meaningful performance reason to choose VARCHAR(n) over TEXT for ordinary text.

### What is a migration?
A version-controlled, ordered change to the database schema or related database objects, applied by a migration tool.

### Why use foreign keys?
They ensure referenced rows exist and preserve referential integrity.

### What is a join table?
A table that represents a many-to-many relationship by storing pairs of foreign keys, often with a composite uniqueness constraint.

### Why parameterized queries?
They keep user-provided values separate from SQL syntax and prevent SQL injection through value concatenation.

### Can sort column names be parameterized like values?
Usually not as ordinary bind values. Use an allowlist that maps public sort keys to trusted SQL identifiers.

### Why index a foreign key?
If it is frequently used in joins or filters, an index can speed up finding referencing rows. PostgreSQL does not automatically index every foreign-key column.

### What is the cost of an index?
Extra storage and additional write overhead because index structures must be maintained on data changes.

### What is a trigger?
A database mechanism that automatically runs a function when a configured event occurs, such as an update.

### Why maintain updated_at in a trigger?
It ensures the timestamp is refreshed consistently for updates from any code path.

## 18. Quick revision sheet

### Persistence
Data survives process restarts.

### DBMS
Stores, organizes, protects and exposes CRUD operations over data.

### RAM vs disk
RAM: faster, smaller/costlier; disk: larger/cheaper, slower.

### Relational
Tables + rows + columns + schema + keys + SQL.

### Document NoSQL
Collections + documents + flexible shapes.

### PostgreSQL
Strong general-purpose relational default; JSONB supports semi-structured fields.

### Money
Prefer NUMERIC/DECIMAL, not floating point.

### General text
TEXT is a sensible PostgreSQL default.

### Migration
Versioned schema change; up applies, down attempts reversal.

### Integrity
PK, FK, UNIQUE, NOT NULL, CHECK and ENUM constraints.

### Many-to-many
Join table with uniqueness across the relationship pair.

### Safe SQL
Bind values; allowlist dynamic identifiers.

### Pagination
LIMIT + OFFSET; use stable ordering.

### Index
Speeds selected reads/joins/orderings; costs storage and writes.

### Trigger
Automatically runs database logic on configured events.

## SQL patterns to remember

```sql
-- A parameterized lookup (placeholder syntax depends on the driver)
SELECT id, email, full_name
FROM users
WHERE id = $1;

-- Join a one-to-one profile
SELECT u.id, u.email, p.bio
FROM users AS u
LEFT JOIN user_profiles AS p
  ON p.user_id = u.id;

-- Stable pagination
SELECT id, email, created_at
FROM users
ORDER BY created_at DESC, id DESC
LIMIT $1 OFFSET $2;

-- Example index for a frequently used filter/order pattern
CREATE INDEX idx_users_created_at
ON users (created_at DESC);

-- Example trigger function pattern for updated_at
CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS trigger AS $$
BEGIN
  NEW.updated_at = CURRENT_TIMESTAMP;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

**Security reminder:** Never build SQL by concatenating raw user input. Bind values through the database driver, and map dynamic sort/filter identifiers through server-side allowlists.

## Suggested practice
1. Create users, user_profiles, projects, tasks and project_members tables.
2. Add primary keys, foreign keys, unique constraints, NOT NULL rules and status enums.
3. Write a migration that creates the schema and a corresponding rollback plan.
4. Insert sample users, projects, tasks and memberships.
5. Write list and detail queries with joins, filtering, sorting and pagination.
6. Add indexes based on the queries you actually run; compare query plans before and after.
7. Add an updated_at trigger and verify it fires on updates.

## Source boundary and accuracy notes
- The lecture is an application-oriented introduction, not a complete PostgreSQL reference.
- The transcript's broad comparison of relational and non-relational databases is simplified; real systems can combine strict validation with flexible JSON/document fields.
- PostgreSQL's JSONB is not a universal replacement for every NoSQL workload; choose based on access patterns, consistency needs and operational requirements.
- Migration rollback is not always safely reversible, especially after destructive data changes.
- Exact SQL placeholder syntax and migration-file conventions depend on the driver and migration tool.