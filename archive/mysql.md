# MySQL

## Deploy

```sh
dnf -y install mariadb-devel
cpanm Devel::CheckLib DBI https://github.com/gooddata/DBD-MariaDB/archive/master.tar.gz
```

## Connect

```sh
my $dbh = DBI->connect(
    qq[DBI:MariaDB:database=$db;host=$host;port=$port],
    $username,
    $password,
    {   RaiseError               => 1,
        PrintWarn                => 0,
        ShowErrorStatement       => 1,
        mariadb_auto_reconnect   => 1,
        mariadb_multi_statements => 1,
    }
);
```

## Create table

```sql
CREATE TABLE IF NOT EXISTS `table` (
    `id` VARCHAR(20) NOT NULL,
    PRIMARY KEY (`player_id`)
) DEFAULT CHARSET=utf8mb4;
```
