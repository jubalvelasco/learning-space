# Install Database

## Install MariaDB in Ubuntu Linux

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install software-properties-common
sudo apt-get install mariadb-server mariadb-client
```

## Install MariaDB in Arch Linux

```bash
sudo pacman -Syu
sudo pacman -S mariadb
```

## Check if MariaDB is running

```bash
sudo systemctl status mariadb
```

## Run MariaDB if not running

```bash
sudo systemctl start mariadb
```

## Stop MariaDB if running

```bash
sudo systemctl stop mariadb
```

## Status MariaDB

```bash
sudo systemctl status mariadb
```

## Enable MariaDB to start on system start-up

```bash
sudo systemctl enable mariadb
```

## Secure your MariaDB

```bash
sudo mysql_secure_installation
```

## Login to root user

- In **ubuntu** linux the database root user uses `unix_socket` authentication by default, so running `sudo mariadb` logs you in as root without a database password (the password `sudo` asks for is just your Linux user's password, not a database password).
- But if you don't want to use `sudo` to open the root user, you can set a password for the **root user** in one of the steps in `sudo mysql_secure_installation`
- In **windows** the root password is the one you set during installation, make sure to remember that.

```bash
sudo mariadb -u root -p
```

## Create a new admin user then, grant all privileges to it

```sql
CREATE USER 'admin_user'@'localhost' IDENTIFIED BY 'secret_password';
GRANT ALL PRIVILEGES ON *.* TO 'admin_user'@'localhost' WITH GRANT OPTION;
-- FLUSH PRIVILEGES; is NOT required after CREATE USER / GRANT (it's only needed if you edit the grant tables directly)
```

## Login to admin_user

```bash
mariadb -u admin_user -p
```

## Exporting a Database
```bash
mysqldump -u username -p database_name > data-dump.sql
```

## Importing a Database

```bash
mysql -u username -p new_empty_database < data-dump.sql
```

## Create A User For A Server

1. **Create A User Using Root User On The Server**

    ```sql
    CREATE USER '<db_user_name>'@'<user_address>' IDENTIFIED BY '<db_user_password>';
    GRANT ALL PRIVILEGES ON `<database_name>`.* TO '<db_user_name>'@'<user_address>';
    ```
    If you want to give access to only one table use **`<database_name>`.`<table_name>`** instead of `` `<database_name>`.* ``

    - The `<user_address>` is the IP address or hostname the client connects **from** (`localhost` for local only, `%` for any host). Note that `'user'@'localhost'` and `'user'@'%'` are two different accounts.

2. **Remove The User If You Want/Need To**
    ```sql
    DROP USER IF EXISTS <db_user_name>@<user_address>;
    ```

3. **Access Database From Client**

    ```bash
    mariadb -u <db_user_name> -p<db_user_password> -h <server_address> <database_name>
    ```

    - The `<server_address>` could be an IP address or hostname where the database resides.
    - Writing the password right after `-p` (no space) works, but it will be saved in your shell history; prefer just `-p` and type it when prompted.
