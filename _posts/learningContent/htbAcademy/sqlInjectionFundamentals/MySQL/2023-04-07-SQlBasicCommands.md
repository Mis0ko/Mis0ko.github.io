---
title:  "Commandes de bases de SQL"
category: "MySQL"
tag: "Principes de base de l'injection SQL"
---

### Connection à un serveur MySQL

```console
Misoko@htb[/htb]$ mysql -u root -p <password> -P 3306 -h docker.hackthebox.remote.host.eu 
Misoko@htb[/htb]$ mysql -u root -h 138.68.169.126 -P 32006 -ppassword
mysql> 
```

### Affichage de bases de données

```console
mysql> SHOW DATABASES;
```

# TABLES

### Création de Tables

```sql
CREATE TABLE logins (
    id INT,
    username VARCHAR(100),
    password VARCHAR(100),
    date_of_joining DATETIME
    );

```

### Affichage de tables

```console
mysql> SHOW TABLES;

+-----------------+
| Tables_in_users |
+-----------------+
| logins          |
+-----------------+
1 row in set (0.00 sec)
```

### Inspecter une table

```console
mysql> DESCRIBE logins;

+-----------------+--------------+
| Field           | Type         |
+-----------------+--------------+
| id              | int          |
| username        | varchar(100) |
| password        | varchar(100) |
| date_of_joining | date         |
+-----------------+--------------+
4 rows in set (0.00 sec)
```

### Insertion 

```console
mysql> INSERT INTO logins VALUES(1, 'admin', 'p@ssw0rd', '2020-07-02');
```

### SELECT 

```console
mysql> SELECT * FROM logins;
```

### DROP 

```console
mysql> DROP TABLE logins;
```

### ALTER 

```console
mysql> ALTER TABLE logins ADD newColumn INT;
mysql> ALTER TABLE logins RENAME COLUMN newColumn TO oldColumn;
mysql> ALTER TABLE logins MODIFY oldColumn DATE;
mysql> ALTER TABLE logins DROP oldColumn;
```

### ORDER BY 

```console
mysql> SELECT * FROM logins ORDER BY password DESC, id ASC;
```


### LIMIT 

```console
mysql> mysql> SELECT * FROM logins LIMIT 2;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
+----+---------------+------------+---------------------+
2 rows in set (0.00 sec)
```


### WHERE 

```console
SELECT * FROM table_name WHERE <condition>;
```


### LIKE 

```console
mysql> SELECT * FROM logins WHERE username like '___';  # Exactement 3 caractères
mysql> SELECT * FROM logins WHERE username LIKE 'admin%';

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  4 | administrator | adm1n_p@ss | 2020-07-02 15:19:02 |
+----+---------------+------------+---------------------+
2 rows in set (0.00 sec)
```



