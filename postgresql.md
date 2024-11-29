# PostgreSQL

### Create database

```sql
CREATE EXTENSION IF NOT EXISTS softvisio;

SELECT create_database( 'test', 'en_US.UTF8' ) AS password;
```

### User management

```sql
# create user
CREATE USER <username> WITH ENCRYPTED PASSWORD '<password>';

# change password for user
\password <username>

# grant priviledges
/c <dbname>;
GRANT ALL PRIVILEGES ON ALL TABLES IN schema public To <username>;

# to create extensions you need to temporary grant superuser permissions
ALTER USER <username> WITH SUPERUSER;
ALTER USER <username> WITH NOSUPERUSER;

# drop user
REASSIGN OWNED BY <username> TO postgres;
DROP USER <username>;
```

### Unix timestamp with microseconds

```sql
unix_timestamp float NOT NULL DEFAULT extract( EPOCH FROM CURRENT_TIMESTAMP ),
```

### UUID

[`uuid-ossp` extension documentation.](https://www.postgresql.org/docs/current/static/uuid-ossp.html)

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;

# use UUID v4
id uuid PRIMARY KEY NOT NULL DEFAULT gen_random_uuid()
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

### Lock records

```javascript
// listen for free threads
semaphore.on("free", this.runTasks.bind(this));

// wait for mutex
await mutex.down();

// wait for connection
const abortSignal = await this.dbh.getAbortSignal();

// queue is not empty
if (semaphore.waitingThreads) return mutex.up();

const limit = semaphore.totalFreeThreads;

// lock
const tasks = await dbh.select(
    sql`
WITH cte AS (
    SELECT
        id
    FROM
        task
    WHERE
        ( lock >= 0 AND NOT EXISTS ( SELECT FROM pg_active_backend WHERE id = task.lock ) )
    LIMIT ?
    FOR UPDATE
)
UPDATE
    task
SET
    lock = pg_backend_id( ? )
FROM
    cte
WHERE task.id = cte.id
`,
    [limit, abortSignal.pid]
);

// run threads
for (const task of tasks) {
    semaphore.runThread(async task => {
        // check abort signal
        if (abortSignal.aborted) return;

        // process task
        // ...

        // unlock task
        await this.sbh.do(sql`UPDATE task SET lock = NULL WHERE id = ?`, [task.id]);
    });
}

// unlock mutex
this.#mutex.up();
```

```javascript
// unlock all records
UPDATE task SET lock = 0 WHERE lock != 0 AND NOT EXISTS ( SELECT FROM pg_active_backend WHERE id = task.lock );
```

### PostGIS

- Default SRID is `4326`.

- Coordinates order: longitude, latitude.

```sql
CREATE EXTENSION IF NOT EXISTS postgis;

CREATE TABLE IF NOT EXISTS item (
    id serial8 PRIMARY KEY,
    location geography ( 'POINT' ) NOT NULL
);

CREATE INDEX item_location_idx ON item USING GIST ( location );

INSERT INTO item ( location ) VALUES
    ( 'POINT( 0 0 )' ),
    ( 'POINT( 1 1 )' )
;

SELECT id,
    st_distance( location, 'POINT( 0 0 )'::geography, true ) AS distance,
    st_dWithin( location, 'POINT( 0 0 )'::geography, 156899, true ) AS within
FROM
    item
WHERE
    st_dWithin( location::geometry, 'POINT( 0 0 )'::geography::geometry, 156899, true )
ORDER BY
    location <-> 'POINT( 0 0 )'::geography DESC
;
```

#### Update PostGIS extension

```sql
SELECT postgis_full_version();

SELECT * FROM pg_available_extensions WHERE name = 'postgis';

SELECT postgis_extensions_upgrade();

```

### Extensions

```sql
-- get available extensions
ELECT * FROM pg_available_extensions;

-- get updatable extensions
SELECT * FROM pg_available_extensions WHERE installed_version IS NOT NULL AND installed_version != default_version;

-- create extension in the specific schema
CREATE EXTENSION <extension-name> SCHEMA <schema-name>;
```
