# PostgreSQL

### Create database

```sql
create extension if not exists softvisio;

select create_database( 'test' ) AS password;
```

### User management

```sql
# create user
create user <username> with encrypted password '<password>';

# change password for user
\password <username>

# grant priviledges
/c <dbname>;
grant all privileges on all tables in schema public to <username>;

# to create extensions you need to temporary grant superuser permissions
alter user <username> with superuser;
alter user <username> with nosuperuser;

# drop user
reassign owned by <username> to postgres;
drop user <username>;
```

### Unix timestamp with microseconds

```sql
unix_timestamp float not null default extract( epoch from current_timestamp ),
```

### UUID

[`uuid-ossp` extension documentation.](https://www.postgresql.org/docs/current/static/uuid-ossp.html)

```sql
create extension if not exists pgcrypto;

# use UUID v4
id uuid primary key not null default gen_random_uuid()
```

### Notifications

String payload:

```sql
perform pg_notify( 'event-name', payload );
```

JSON payload:

```sql
perform pg_notify( 'event-name', json_build_object( 'key1', 'value1', 'key2', 'value2' )::text );
```

### INSERT ON CONFLICT UPDATE RETURNING

All inserted `key` fields must be unique, otherwise you will get error `Can not affect row a second time`. In the example below if `column` field is unique in the table - you must pre-filter data and throw away rows with the duplicate `column`.

```sql
insert into table ( column ) values ( ? )
on conflict ( key ) do update set column = excluded.column
returning *;
```

### Lock records

`locked` column values:

-   `-1` action done;
-   `0` action pending;
-   `>0` action locked, if pid exists;

```javascript
const res = await this.dbh.lock(dbh => {
    // lock
    const tasks = await dbh.select(sql`
with cte as (
    select
		id
	from task
	where
		( locked >= 0 and not exists ( select from pg_stat_activity where pid = task.locked and datname = current_database() ) )
	limit ?
	for update
)
update
	task
set
	locked = pg_backend_pid()
from
	cte
where task.id = cte.id
`);

	// process tasks
	// ...

	// unlock
	update task set locked = -1 where locked = pg_backend_pid();
});
```

```javascript
// unlock all records
update task set locked = 0 where locked > 0 and not exists ( select from pg_stat_activity where pid = task.locked and datname = current_database() );
```
