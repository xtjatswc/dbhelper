# 🍗 dbhelper 🍗

dbhelper is a small php wrapper for mysql/pgsql databases.

## installation

install once with composer:
```
composer require vielhuber/dbhelper
```

then add this to your files:
```php
require __DIR__.'/vendor/autoload.php';
use vielhuber\dbhelper\dbhelper;
$db = new dbhelper();
```

## usage

```php
/* connect to database */
$db->connect('pdo', 'mysql', '127.0.0.1', 'root', 'root', 'database', 3306);

/* disconnect from database */
$db->disconnect();

/* create/update/delete */
$id = $db->insert('tablename', ['col1' => 'foo']);
$db->update('tablename', ['col1' => 'bar'], ['id' => $id]);
$db->delete('tablename', ['id' => $id]);

/* select */
$db->fetch_all('SELECT * FROM tablename WHERE name = ? AND number > ?', 'foo', 42);
$db->fetch_row('SELECT * FROM tablename WHERE ID = ?', 1);
$db->fetch_col('SELECT col FROM tablename WHERE ID > ?', 1);
$db->fetch_var('SELECT col FROM tablename WHERE ID = ?', 1);

/* automatic flattened arguments */
$db->fetch_all('SELECT * FROM tablename WHERE ID = ?', [1], 2, [3], [4,5,6]);
=>
$db->fetch_all('SELECT * FROM tablename WHERE ID = ?', 1, 2, 3, 4, 5, 6);

/* automatic in-expansion */
$db->fetch_all('SELECT * FROM tablename WHERE col1 = ? AND col2 IN (?)', 1, [2,3,4]);

/* support for null values */
$db->query('UPDATE tablename SET col1 = ? WHERE col2 = ? AND col3 != ?', null, null, null);
=>
$db->query('UPDATE tablename SET col1 = NULL WHERE col2 IS NULL AND col3 IS NOT NULL');

/* delete all tables (without dropping the whole database) */
$db->clear('database');

/* raw queries */
$id = $db->query('INSERT INTO tablename(row1, row2) VALUES(?, ?, ?)', 1, 2, 3);
$db->query('UPDATE tablename SET row1 = ? WHERE ID = ?', 1, 2);
$db->query('DELETE FROM tablename WHERE ID = ?', 1);

/* batch functions (they create only one query) */
$db->insert('tablename', [
    ['id' => 1, 'name' => 'foo1'],
    ['id' => 2, 'name' => 'foo2'],
    ['id' => 3, 'name' => 'foo3']
]);
$db->delete('tablename', [
    ['id' => 1],
    ['id' => 7],
    ['id' => 42]
]);
$db->update('tablename', [
    [['col1' => 'var1', 'col2' => 1], ['id' => 1, 'key' => '1']],
    [['col1' => 'var2', 'col2' => 2], ['id' => 2, 'key' => '2']],
    [['col1' => 'var3', 'col2' => 3], ['id' => 3, 'key' => '3']]
]);
/*
this generates the following query:
UPDATE tablename SET
col1 = CASE WHEN (id = 1 AND key = '1') THEN 'var1' WHEN (id = 2 AND key = '2') THEN 'var2' WHEN (id = 3 AND key = '3') THEN 'var3' END,
col2 = CASE WHEN (id = 1 AND key = '1') THEN 1 WHEN (id = 2 AND key = '2') THEN 2 WHEN (id = 3 AND key = '3') THEN 3 END
WHERE id IN (1,2,3) AND key IN ('1','2','3');
*/
```

this also works for wordpress (using wpdb, prepared statements and stripslashes_deep under the hood):
```php
$db->connect('wordpress');
$db->fetch_var('SELECT col FROM tablename WHERE ID = ?', 1);
```

as return values dbhelper usually returns associative arrays. if you use it with wordpress, objects are returned.

zhere is also a static version with static function calls (if you only use a single instance of dbhelper):
```php
require_once($_SERVER['DOCUMENT_ROOT'].'/vendor/vielhuber/dbhelper/src/static.php');
db_fetch_var('SELECT col FROM tablename WHERE ID = ?', 1);
```