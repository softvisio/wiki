# PostgreSQL

## Create database

```
CREATE EXTENSION IF NOT EXISTS "softvisio";

SELECT create_database('test') AS "password";
```

## User management

```
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

## Unix timestamp with microseconds

```
"unix_timestamp" FLOAT NOT NULL DEFAULT EXTRACT(EPOCH FROM CURRENT_TIMESTAMP),
```

## UUID

[`uuid-ossp` extension documentation.](https://www.postgresql.org/docs/current/static/uuid-ossp.html)

```
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

# use UUID v1 - based on MAC addr.
"id" uuid PRIMARY KEY NOT NULL DEFAULT uuid_generate_v1()

CREATE EXTENSION IF NOT EXISTS "pgcrypto";

# use UUID v4 - random
"id" uuid PRIMARY KEY NOT NULL DEFAULT gen_random_uuid()
```

## Notifications

```
PERFORM pg_notify( 'event-name', payload );
```
