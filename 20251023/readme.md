# Training MySQL Oktober 2025
## 1. Replication GTID for Disaster Recovery

enable GTID_MODE on config file server DC & DRC 
```
nano /etc/my.cnf

--- ADD ---
enforce_gtid_consistency=ON
gtid_mode=ON

```

Restart Service MySQL
```
systemctl restart mysqld

```

On Slave
```
mysql> show variables like '%gtid_mode%';show variables like '%enforce_gtid_consistency%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| gtid_mode     | ON    |
+---------------+-------+
1 row in set (0.01 sec)

+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| enforce_gtid_consistency | ON    |
+--------------------------+-------+
1 row in set (0.00 sec)

mysql> start slave;
Query OK, 0 rows affected, 2 warnings (0.00 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 192.168.170.134
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog-innodbcluster2.000001
          Read_Master_Log_Pos: 504
               Relay_Log_File: innodbcluster3-relay-bin.000002
                Relay_Log_Pos: 744
        Relay_Master_Log_File: binlog-innodbcluster2.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 504
              Relay_Log_Space: 963
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 134
                  Master_UUID: 97f0dde3-ab69-11f0-b830-0050562e94ff
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set: 97f0dde3-ab69-11f0-b830-0050562e94ff:1
            Executed_Gtid_Set: 97f0dde3-ab69-11f0-b830-0050562e94ff:1
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
       Master_public_key_path:
        Get_master_public_key: 0
            Network_Namespace:
1 row in set, 1 warning (0.01 sec)

```

On Master
```
mysql> show master status\G
*************************** 1. row ***************************
             File: binlog-innodbcluster2.000001
         Position: 504
     Binlog_Do_DB:
 Binlog_Ignore_DB:
Executed_Gtid_Set: 97f0dde3-ab69-11f0-b830-0050562e94ff:1
1 row in set (0.00 sec)

mysql> CREATE TABLE employees (
    employee_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    hourly_pay DECIMAL(10, 2),
    hire_date DATE
);
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO db_test2.employees (first_name, last_name, email, hire_date, salary) VALUES ('orang1', 'oke1', 'orang1.oke1@example.com', '2023-01-15', 60000.00);
Query OK, 1 row affected (0.00 sec)

mysql> select * from db_test2.employees;
+-------------+------------+-----------+--------------------------+------------+----------+
| employee_id | first_name | last_name | email                    | hire_date  | salary   |
+-------------+------------+-----------+--------------------------+------------+----------+
|           1 | John       | Doe       | john.doe@example.com     | 2023-01-15 | 60000.00 |
|           3 | Jane       | Smith     | jane.smith@example.com   | 2022-05-20 | 75000.00 |
|           4 | Peter      | Jones     | peter.jones@example.com  | 2024-03-10 | 55000.00 |
|           6 | John       | Doel      | john.doel@example.com    | 2023-01-15 | 60000.00 |
|           7 | Johny      | yDoe      | john.ydoe@example.com    | 2023-01-15 | 60000.00 |
|           8 | Johnyy     | yDoel     | john.ydoel@example.com   | 2023-01-15 | 60000.00 |
|           9 | Johnt      | Doet      | john.doet@example.com    | 2023-01-15 | 60000.00 |
|          10 | Johnll     | Doelll    | john.doelll@example.com  | 2023-01-15 | 60000.00 |
|          11 | Johnll1    | Doelll    | john1.doelll@example.com | 2023-01-15 | 60000.00 |
|          12 | orang1     | oke1      | orang1.oke1@example.com  | 2023-01-15 | 60000.00 |
+-------------+------------+-----------+--------------------------+------------+----------+
10 rows in set (0.00 sec)

mysql> show master status\G
*************************** 1. row ***************************
             File: binlog-innodbcluster2.000001
         Position: 847
     Binlog_Do_DB:
 Binlog_Ignore_DB:
Executed_Gtid_Set: 97f0dde3-ab69-11f0-b830-0050562e94ff:1-2
1 row in set (0.00 sec)

```

On Slave
```
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 192.168.170.134
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog-innodbcluster2.000001
          Read_Master_Log_Pos: 847
               Relay_Log_File: innodbcluster3-relay-bin.000002
                Relay_Log_Pos: 1087
        Relay_Master_Log_File: binlog-innodbcluster2.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 847
              Relay_Log_Space: 1306
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 134
                  Master_UUID: 97f0dde3-ab69-11f0-b830-0050562e94ff
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set: 97f0dde3-ab69-11f0-b830-0050562e94ff:1-2
            Executed_Gtid_Set: 97f0dde3-ab69-11f0-b830-0050562e94ff:1-2
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
       Master_public_key_path:
        Get_master_public_key: 0
            Network_Namespace:
1 row in set, 1 warning (0.01 sec)

mysql> select * from db_test2.employees;
+-------------+------------+-----------+--------------------------+------------+----------+
| employee_id | first_name | last_name | email                    | hire_date  | salary   |
+-------------+------------+-----------+--------------------------+------------+----------+
|           1 | John       | Doe       | john.doe@example.com     | 2023-01-15 | 60000.00 |
|           3 | Jane       | Smith     | jane.smith@example.com   | 2022-05-20 | 75000.00 |
|           4 | Peter      | Jones     | peter.jones@example.com  | 2024-03-10 | 55000.00 |
|           6 | John       | Doel      | john.doel@example.com    | 2023-01-15 | 60000.00 |
|           7 | Johny      | yDoe      | john.ydoe@example.com    | 2023-01-15 | 60000.00 |
|           8 | Johnyy     | yDoel     | john.ydoel@example.com   | 2023-01-15 | 60000.00 |
|           9 | Johnt      | Doet      | john.doet@example.com    | 2023-01-15 | 60000.00 |
|          10 | Johnll     | Doelll    | john.doelll@example.com  | 2023-01-15 | 60000.00 |
|          11 | Johnll1    | Doelll    | john1.doelll@example.com | 2023-01-15 | 60000.00 |
|          12 | orang1     | oke1      | orang1.oke1@example.com  | 2023-01-15 | 60000.00 |
+-------------+------------+-----------+--------------------------+------------+----------+
10 rows in set (0.00 sec)

```

## Using Replication Filter for specific schema to switchover / drill

scenario : we have 2 schema database (db_test1 & db_test2) but we just want db_test1 schema switchover to master, so we can use replica filter for this scenario
```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| db_test1           |
| db_test2           |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
6 rows in set (0.00 sec)

mysql> use db_test1; show tables; select * from users;
Database changed
+--------------------+
| Tables_in_db_test1 |
+--------------------+
| users              |
+--------------------+
1 row in set (0.00 sec)

+----+----------+----------------------+-------------------+
| id | name     | email                | registration_date |
+----+----------+----------------------+-------------------+
|  1 | John Doe | john.doe@example.com | 2025-10-18        |
+----+----------+----------------------+-------------------+
1 row in set (0.00 sec)

mysql> use db_test2; show tables; select * from employees;
Database changed
+--------------------+
| Tables_in_db_test2 |
+--------------------+
| employees          |
+--------------------+
1 row in set (0.01 sec)

+-------------+------------+-----------+--------------------------+------------+----------+
| employee_id | first_name | last_name | email                    | hire_date  | salary   |
+-------------+------------+-----------+--------------------------+------------+----------+
|           1 | John       | Doe       | john.doe@example.com     | 2023-01-15 | 60000.00 |
|           3 | Jane       | Smith     | jane.smith@example.com   | 2022-05-20 | 75000.00 |
|           4 | Peter      | Jones     | peter.jones@example.com  | 2024-03-10 | 55000.00 |
|           6 | John       | Doel      | john.doel@example.com    | 2023-01-15 | 60000.00 |
|           7 | Johny      | yDoe      | john.ydoe@example.com    | 2023-01-15 | 60000.00 |
|           8 | Johnyy     | yDoel     | john.ydoel@example.com   | 2023-01-15 | 60000.00 |
|           9 | Johnt      | Doet      | john.doet@example.com    | 2023-01-15 | 60000.00 |
|          10 | Johnll     | Doelll    | john.doelll@example.com  | 2023-01-15 | 60000.00 |
|          11 | Johnll1    | Doelll    | john1.doelll@example.com | 2023-01-15 | 60000.00 |
|          12 | orang1     | oke1      | orang1.oke1@example.com  | 2023-01-15 | 60000.00 |
+-------------+------------+-----------+--------------------------+------------+----------+
10 rows in set (0.00 sec)


```

On Slave : Disable super_read_only & read_only and Verify Slave Status Running
```
mysql> show variables like '%read_only%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_read_only      | OFF   |
| read_only             | ON    |
| super_read_only       | ON    |
| transaction_read_only | OFF   |
+-----------------------+-------+
4 rows in set (0.01 sec)

mysql> set global super_read_only=0;set global read_only=0;
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like '%read_only%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_read_only      | OFF   |
| read_only             | OFF   |
| super_read_only       | OFF   |
| transaction_read_only | OFF   |
+-----------------------+-------+
4 rows in set (0.01 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 192.168.170.134
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog-innodbcluster2.000001
          Read_Master_Log_Pos: 2304
               Relay_Log_File: innodbcluster3-relay-bin.000002
                Relay_Log_Pos: 2544
        Relay_Master_Log_File: binlog-innodbcluster2.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 2304
              Relay_Log_Space: 2763
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 134
                  Master_UUID: 97f0dde3-ab69-11f0-b830-0050562e94ff
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set: 97f0dde3-ab69-11f0-b830-0050562e94ff:1-8
            Executed_Gtid_Set: 97f0dde3-ab69-11f0-b830-0050562e94ff:1-8
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
       Master_public_key_path:
        Get_master_public_key: 0
            Network_Namespace:
1 row in set, 1 warning (0.01 sec)

```

On Master : Verify No Slave Status and Set Slave for db_test1 schema with Replication Filter
```
mysql> show slave status\G
Empty set, 1 warning (0.01 sec)

mysql> show variables like '%read_only%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_read_only      | OFF   |
| read_only             | OFF   |
| super_read_only       | OFF   |
| transaction_read_only | OFF   |
+-----------------------+-------+
4 rows in set (0.00 sec)

mysql> CHANGE MASTER TO MASTER_HOST='192.168.170.135', MASTER_USER='repl', MASTER_PASSWORD='Tes250299!!!', MASTER_PORT=3306, MASTER_AUTO_POSITION = 1;
Query OK, 0 rows affected, 8 warnings (0.03 sec)

mysql> CHANGE REPLICATION FILTER REPLICATE_DO_DB = (db_test1);
Query OK, 0 rows affected (0.00 sec)

mysql> start slave;
Query OK, 0 rows affected, 1 warning (0.02 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 192.168.170.135
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog-innodbcluster3.000001
          Read_Master_Log_Pos: 2361
               Relay_Log_File: innodbcluster2-relay-bin.000002
                Relay_Log_Pos: 456
        Relay_Master_Log_File: binlog-innodbcluster3.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: db_test1
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 2361
              Relay_Log_Space: 675
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 135
                  Master_UUID: 37cf40f4-ab68-11f0-9657-0050563c05df
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set: 97f0dde3-ab69-11f0-b830-0050562e94ff:1-8
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
       Master_public_key_path:
        Get_master_public_key: 0
            Network_Namespace:
1 row in set, 1 warning (0.00 sec)

```

On Slave : Insert Data on Users Table
On Master : Check whether the data in the Users Table has been replicated  
```
mysql> INSERT INTO db_test1.users VALUES (NULL, 'Andy', 'Andy@example.com', '2025-10-18');
Query OK, 1 row affected (0.00 sec)

mysql> select * from db_test1.users;
+----+----------+----------------------+-------------------+
| id | name     | email                | registration_date |
+----+----------+----------------------+-------------------+
|  1 | John Doe | john.doe@example.com | 2025-10-18        |
|  2 | Andy     | Andy@example.com     | 2025-10-18        |
+----+----------+----------------------+-------------------+
2 rows in set (0.00 sec)

```

## 2. Pengenalan MySQL 9 sebagai inovasi rilis dalam pengembangan AI