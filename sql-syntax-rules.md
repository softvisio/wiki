# Example

```sql
CREATE TABLE "test" (
	"id" uuid PRIMARY KEY NOT NULL DEFAULT get_random_uuid(),
	"title" text NOT NULL
);

CREATE UNIQUE INDEX IF NOT EXISTS "test_title_idx" ON "test" ("title");

SELECT count(*) AS "total", COALESCE("name", 'no name') FROM "user" WHERE "id" IN (100, 101) GROUP BY "group";
```

# Rules

-   All keywords (SELECT, INSER, etc.) must be in upper case;

-   All identifiers (table, column, indexes, etc. names) must be double-quoted lower camel case strings.

    -   Reserver row identifiers `NEW`, `OLD`, `EXCLUDED` must be unquoted upper case.

-   Subquery expressions: `EXISTS`, `IN`, `NOT IN`, `ANY`, `SOME`, `ALL` must be in upper case.

-   Conditional expressions `CASE`, `COALECSE`, `NULLIF`, `CREATEST`, `LEAST` must be in upper case;

-   All type names must be in lower case.

-   All functions names must be in lower camel case.

## Indexes

-   Index name template `<table_name>_<column1_name>_<column2_name>_idx`.

    Index names examples

    ```sql
    CREATE INDEX "user_id_idx" ON "user" ("id");

    CREATE INDEX "user_id_title_idx" ON "user" ("id", "title");
    ```

## Sequences

-   Sequence name template `<table_name>_<column_name>_seq`.

    Sequence names examples

    ```sql
    CREATE SEQUENCE "user_id_seq" AS int8;
    ```

## Triggers

-   Trigger name template `<table_name>_<triiger_event>_trigger`.

-   Trigger functions names must have same name as trigger or must have `_trigger` suffix.

    Trigger names examples

    ```sql
    CREATE FUNCTION user_before_insert_trigger() RETURNS TRIGGER

    CREATE TRIGGER "user_before_insert_trigger" BEFORE INSERT ON "user" FOR EACH ROW EXECUTE PROCEDURE user_before_insert_trigger();
    ```
