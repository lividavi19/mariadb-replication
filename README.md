# MARIADB MASTER-SLAVE REPLIACTION

## MASTER SERVER

### Installing MariaDB server
```
sudo apt update
sudo apt install mariadb-server
```

Validate MariaDB Installtion
```
mariadb --version
```

Or, check MariaDB Service Status;
```
sudo systemctl status mariadb
```

### Configure MariaDB
Open the MariaDB configuration file for editing
```
sudo vi /etc/mysql/mariadb.conf.d/50-server.cnf
```

Add the following lines to the [mysqld] section:
```
[mysqld]
server-id = 1
log_bin = /var/log/mysql/mariadb-bin
log_bin_index = /var/log/mysql/mariadb-bin.index
```

Save and exit editor.
```
:wq
```

Then restart MariaDB server.
```
sudo systemctl restart mariadb
```

### Create Replication User
Log in to the MariaDB shell:
```
mysql -u root -p
```

Then run the following SQL commands;
```
GRANT REPLICATION SLAVE ON *.* TO 'replication_user'@'%' IDENTIFIED BY 'your_password';
FLUSH PRIVILEGES;
```
Replace values for `replication_user` and `your_password` with your actual details.

### Check Master Status
In the MariaDB shell, run
```
SHOW MASTER STATUS;
```
Note down the values for `File` and `Position`.


## SLAVE SERVER

### Install MariaDB
```
sudo apt update
sudo apt install mariadb-server
```

### Configure MariaDB
Open the MariaDB configuration file for editing:
```
sudo vi /etc/mysql/mariadb.conf.d/50-server.cnf
```

Add the following line to the [mysqld] section:
```
[mysqld]
server-id = 2
```

Save and exit editor.
```
:wq
```

Then restart MariaDB server.
```
sudo systemctl restart mariadb
```

### Import Data Snapshot (if available)
If you have a data snapshot from the master, import it into the slave database:
```
mysql -u root -p < snapshot.sql
```

### Configure Replication on Slave
Log in to the MariaDB shell:
```
mysql -u root -p
```

Run the following SQL commands, replacing placeholder values:
```
CHANGE MASTER TO
  MASTER_HOST='master_ip',
  MASTER_USER='replication_user',
  MASTER_PASSWORD='your_password',
  MASTER_LOG_FILE='mariadb-bin.000001',  -- Use actual binlog file from master
  MASTER_LOG_POS=123;  -- Use actual position from master
```

### Start Replication on Slave
In the MariaDB shell:
```
START SLAVE;
```

### Check Slave Status
To monitor the replication status, use:
```
SHOW SLAVE STATUS\G;
```
Verify that `Slave_IO_Running` and `Slave_SQL_Running` are both `Yes`.
