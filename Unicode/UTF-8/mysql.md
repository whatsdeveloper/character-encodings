# Character encodings in MySQL

## Modifications to your `my.cnf` file

On the MySQL/UTF-8 side of things, modifications to the `my.cnf` file are required as follows:

- Set the following config parameters after each corresponding tag:

```ini
[client]
default-character-set = UTF-8

[mysql]
default-character-set = UTF-8

[mysqld]
character-set-client-handshake = false
character-set-server = UTF-8
collation-server = UTF-8_general_ci

[mysqld_safe]
default-character-set = UTF-8
```

> Force encoding to uft8: `character-set-client-handshake = false`

- After making the above changes to your `my.cnf` file, restart your MySQL daemon.

- To verify that everything has properly been set to use the UTF-8 encoding, execute the following query:

```sh
mysql> show variables like 'char%';
```

The output should look something like:

| Variable_name            | Value                                |
| ------------------------ | ------------------------------------ |
| character_set_client     | UTF-8                                |
| character_set_connection | UTF-8                                |
| character_set_database   | UTF-8                                |
| character_set_filesystem | binary                               |
| character_set_results    | UTF-8                                |
| character_set_server     | UTF-8                                |
| character_set_system     | UTF-8                                |
| character_sets_dir       | /path/to/mysql/share/mysql/charsets/ |

If you instead see `latin1` listed for any of these, double-check your configuration and make sure you’ve properly restarted your mysql daemon.

## Other things to consider

- **MySQL UTF-8 is actually a partial implementation of the full UTF-8 character set.** Specifically, MySQL UTF-8 encoding uses a maximum of 3 bytes, whereas 4 bytes are required for encoding the full UTF-8 character set. This is fine for all language characters, but if you need to support astral symbols (whose code points range from U+010000 to U+10FFFF), those require a four byte encoding which is not supported in MySQL UTF-8. **In MySQL 5.5.3, this was addressed with the addition of support for the [utf8mb4](https://dev.mysql.com/doc/refman/5.5/en/charset-unicode-utf8mb4.html) character set** which uses a maximum of four bytes per character and thereby supports the full UTF-8 character set. So if you’re using MySQL 5.5.3 or later, use `utf8mb4` instead of UTF-8 as your database/table/row character set. More info is available [here](https://mathiasbynens.be/notes/mysql-utf8mb4).

Make sure to set the client and server character set as well.

```ini
[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4

[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
```

You can easily confirm these settings work correctly:

```sh
mysql> SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';
```

| Variable_name            | Value              |
| ------------------------ | ------------------ |
| character_set_client     | utf8mb4            |
| character_set_connection | utf8mb4            |
| character_set_database   | utf8mb4            |
| character_set_filesystem | binary             |
| character_set_results    | utf8mb4            |
| character_set_server     | utf8mb4            |
| character_set_system     | utf8               |
| collation_connection     | utf8mb4_general_ci |
| collation_database       | utf8mb4_general_ci |
| collation_server         | utf8mb4_general_ci |

- If the connecting client has no way to specify the encoding for its communication with MySQL, after the connection is established you may have to run the following command/query:

```sql
set names UTF-8;
```

- When determining the size of varchar fields when modeling the database, don’t forget that UTF-8 characters may require as many as 4 bytes per character.

## If you use Sphinx

- In your Sphinx configuration file (i.e., `sphinx.conf`):
  - Set your index definition to have: `charset_type = utf-8`
  - Add the following to your source definition:

```ini
sql_query_pre = SET CHARACTER_SET_RESULTS=UTF-8
sql_query_pre = SET NAMES UTF-8
```

- Restart the engine and remake all indices.

- If you want to configure sphinx so that letters like C c Ć ć Ĉ ĉ Ċ ċ Č č are all treated as equivalent for search purposes, you will need to configure a `charset_table` (a.k.a. character folding) which is essentially an equivalency mapping between characters. More information is available [here](http://sphinxsearch.com/docs/current.html#conf-charset-table).

## Summary

Never use `utf8` in MySQL — always use `utf8mb4` instead.
