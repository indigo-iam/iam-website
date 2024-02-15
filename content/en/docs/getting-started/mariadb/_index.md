---
title: "Database configuration"
linkTitle: "Database configuration"
weight: 2
---

IAM keeps state in a MariaDB/MySQL database, and will need its own schema and a
user that has read/write/schema change privileges on such database.

IAM has been tested successfully against:

- MySQL v. 5.5.5
- MariaDB v. 10.1.22

For more instructions on how to create a database schema and a user with
administrator privileges on it, see the [MariaDB documentation][maria-db-doc].

Assuming the organization that IAM will manage is called `test`, the following
commands can be used  to create a database and a user for the IAM application:

```sql
CREATE DATABASE iam_test_db CHARACTER SET latin1 COLLATE latin1_swedish_ci;
CREATE USER 'iam_test'@'%' identified by 'some_super_secure_password';
GRANT ALL PRIVILEGES on iam_test_db.* to 'iam_test'@'%';
```

You may want to restrict the set of hosts from which a mysql client can connect
to the database with the above credentials according to your deployment needs.

[maria-db-doc]:https://mariadb.com/kb/en/

