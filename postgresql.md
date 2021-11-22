# PostgreSQL

### Create database

```sql
CREATE EXTENSION IF NOT EXISTS "softvisio";

SELECT create_database('test') AS "password";
```

### User management

```sql
# create user
CREATE USER "<username>" WITH ENCRYPTED PASSWORD '<password>';

# change password for user
\password "<username>"

# grant priviledges
/c <dbname>;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA "public" TO "<username>";

# to create extensions you need to temporary grant superuser permissions
ALTER USER "<username>" WITH SUPERUSER;
ALTER USER "<username>" WITH NOSUPERUSER;

# drop user
REASSIGN OWNED BY "<username>" TO "postgres";
DROP USER "<username>";
```

### Unix timestamp with microseconds

```sql
"unix_timestamp" FLOAT NOT NULL DEFAULT EXTRACT(EPOCH FROM CURRENT_TIMESTAMP),
```

### UUID

[`uuid-ossp` extension documentation.](https://www.postgresql.org/docs/current/static/uuid-ossp.html)

```sql
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

# use UUID v4
"id" uuid PRIMARY KEY NOT NULL DEFAULT gen_random_uuid()
```

### Notifications

String payload:

```sql
PERFORM pg_notify( 'event-name', payload );
```

JSON payload:

```sql
PERFORM pg_notify( 'event-name', json_build_object( 'key1', 'value1', 'key2', 'value2' )::text );
```

### INSERT ON CONFLICT UPDATE RETURNING

All inserted `key` fields must be unique, otherwise you will get error `Can not affect row a second time`. In the example below if `column` field is unique in the table - you must pre-filter data and throw away rows with the duplicate `column`.

```sql
INSERT INTO table ( column ) VALUES ( ? )
ON CONFLICT ( key ) DO UPDATE SET column = EXCLUDED.column
RETURNING *;
```
