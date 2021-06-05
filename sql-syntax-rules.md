# SQL Syntax rules

## Example

```sql
CREATE TABLE test (
    id uuid PRIMARY KEY NOT NULL DEFAULT get_random_uuid(),
    title text NOT NULL
);

CREATE UNIQUE INDEX IF NOT EXISTS test_title_key ON test (title);

SELECT count(*) AS total, COALESCE(name, 'no name') FROM user WHERE id IN (100, 101) GROUP BY group;
```

## Rules

- All keywords (SELECT, INSER, etc.) must be in upper case;

- All identifiers (table, column, indexes, etc. names) must be in the snake_case.
    - Reserved row identifiers `NEW`, `OLD`, `EXCLUDED` must be unquoted upper case.

- Subquery expressions: `EXISTS`, `IN`, `NOT IN`, `ANY`, `SOME`, `ALL` must be in upper case.

- Conditional expressions `CASE`, `COALECSE`, `NULLIF`, `CREATEST`, `LEAST` must be in upper case;

- All type names must be in the `snake_case`.

- All functions names must be in the `snake_case`.

### Indexes

Indexes must be unique across the schema.

Index name must be in the `snake_case`.

Index name template `<table_name>_<column_name>..._<suffix>`, where the suffix is one of the following:

- 'pkey' for a Primary Key constraint;
- `key` for a Unique constraint;
- `excl` for an Exclusion constraint;
- `idx` for any other kind of index;
- `fkey` for a Foreign key;
- `check` for a Check constraint;

Examples

```sql
CREATE INDEX user_id_idx ON "user" ( id );

CREATE INDEX user_id_title_idx ON "user" ( id, title );
```

### Sequences

Sequence name template `<table_name>_<column_name>_seq`.

Examples

```sql
CREATE SEQUENCE user_id_seq AS int8;
```

### Triggers

`PostgreSQL` triggers are unique across the table, so trigger name can be: `[<column_name>...]_<triger_event>`, eg: `before_update`, `some_column_after_insert`;

`SQLite` triggers are global, so name must be fully specified, template `<table_name>_[<column_name>...]_<triger_event>_trigger`.

Trigger functions names must have same name as trigger and must have `_trigger` suffix.

Examples:

```sql
CREATE FUNCTION user_before_insert_trigger() RETURNS TRIGGER

-- SQLite
CREATE TRIGGER user_before_insert_trigger BEFORE INSERT ON user FOR EACH ROW EXECUTE PROCEDURE user_before_insert_trigger();

-- PostgreSQL
CREATE TRIGGER before_insert BEFORE INSERT ON user FOR EACH ROW EXECUTE PROCEDURE user_before_insert_trigger();
```
