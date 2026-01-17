## 🚀 USING MYSQL SIMPLE

🎅 **User & Permission**

```md
--Tạo user:

CREATE USER '$USER'@'%' IDENTIFIED BY '$PASS';

--Cap quyen:

GRANT ALL PRIVILEGES ON $USER.* TO '$USER'@'%';
FLUSH PRIVILEGES;

--Xem quyen:

SHOW GRANTS FOR '$USER'@'%';

--Update user:

ALTER USER '$USER'@'%' IDENTIFIED BY '$PASS';

--List User in System Database:

SELECT host, user FROM mysql.user;
```
➡️ **Check DataBase**

```md
--Xem danh sách database:

SHOW DATABASES;

--Tạo database:

CREATE DATABASE mydb;

--Xóa database:

DROP DATABASE mydb;

--Chọn database:

USE mydb;

--Xem danh sách table:

SHOW TABLES;

--Tạo table:

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100)
);

--Xóa table:

DROP TABLE $TABLE_NAME;

--Xem cau truc table:

DESCRIBE $TABLE_NAME;
```

👉 **Backup & Restore**

Backup 1 database

```bash
mysqldump -u root -p euet > euet.sql
```
Backup all Database

```bash
mysqldump -u root -p --all-databases > all_db.sql
````

---

Restore 1 database

```js
Step 1: SQL new phai tao DB_NAME can restore truoc!

Step 2: mysql -u root -p $DB_NAME < euet.sql
```
Restore all Database

```bash
mysql -u root -p < all_db.sql
```

