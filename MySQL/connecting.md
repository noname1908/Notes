# Connecting to the MySQL Server

The examples here use the **mysql** client program, but the principles apply to other clients such as **mysqldump**, **mysqladmin**, or **mysqlshow**.

This command invokes **mysql** without specifying any connection parameters explicitly:
```sh
mysql
```

Because there are no parameter options, the default values apply:
- The default host name is localhost. On Unix, this has a special meaning, as described later.
- The default user name is ODBC on Windows or your Unix login name on Unix.
- No password is sent if neither -p nor --password is given.
- For mysql, the first nonoption argument is taken as the name of the default database. If there is no such option, mysql does not select a default database.

To specify the host name and user name explicitly, as well as a password, supply appropriate options on the command line:
```sh
mysql --host=localhost --user=myname --password=password mydb
# or
mysql -h localhost -u myname -p[password] mydb
```

You can also specify the connection protocol explicitly, even for localhost, by using the --protocol=TCP option:
```sh
mysql --host=127.0.0.1
mysql --host=remote.example.com

mysql --protocol=TCP
mysql --port=13306
```