# 🌐 E-Commerce Distributed Deployment Lab – Web and Database on Separate Servers

In this lab, I deployed the **KodeKloud E-Commerce (LAMP)** application in a **distributed architecture**, separating the **database** and **web server** into two hosts — `db` and `web`.
This setup mirrors real-world production environments where the database and web tiers run independently for better scalability and isolation.

---

## 📋 Lab Overview

**Goal:**

* Install and configure **MariaDB** on the `db` server
* Allow remote connections from the web server
* Load product inventory data from SQL script
* Install and configure **Apache + PHP** on the `web` server
* Connect the web application to the remote database (`db`)
* Verify that the site loads correctly on port 80

**Learning Outcomes:**

* Use SSH to manage multiple servers in a distributed setup
* Create a MariaDB user that allows remote access using `%`
* Load `.sql` data files via MySQL redirection
* Modify Apache configuration to prioritize PHP pages
* Update application code to reference the remote DB host

---

## 🛠 Step-by-Step Journey

### Step 1 – SSH into the Database Server

```bash
ssh db
```

✅ Connected to the target `db` host.

---

### Step 2 – Install and Start MariaDB

```bash
sudo yum install -y mariadb-server
sudo systemctl start mariadb
sudo systemctl status mariadb
sudo systemctl enable mariadb
```

✅ Database installed, started, and enabled for reboot persistence.

---

### Step 3 – Configure Remote Database Access

```bash
sudo mysql
```

Inside the MySQL shell:

```sql
CREATE DATABASE ecomdb;
CREATE USER 'ecomuser'@'%' IDENTIFIED BY 'ecompassword';
GRANT ALL PRIVILEGES ON *.* TO 'ecomuser'@'%';
FLUSH PRIVILEGES;
EXIT;
```

✅ Allows `ecomuser` to connect from **any host** using `%`.

---

### Step 4 – Load the Sample Data

```bash
sudo mysql < /opt/db-load-script.sql
```

✅ Imports the product inventory data into `ecomdb`.

---

### Step 5 – Switch to Web Server and Install Packages

```bash
exit    # return to control node
ssh web # connect to the web server
sudo yum install -y httpd php php-mysqlnd
```

✅ Web server and PHP modules installed.

---

### Step 6 – Set PHP as the Default Apache Page

```bash
sudo sed -i 's/index.html/index.php/g' /etc/httpd/conf/httpd.conf
```

✅ Ensures PHP homepage (`index.php`) loads first.

---

### Step 7 – Start and Enable Apache

```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```

✅ Apache running and enabled on boot.

---

### Step 8 – Clone the Application Code

```bash
sudo git clone https://github.com/kodekloudhub/learning-app-ecommerce.git /var/www/html/
```

✅ Application source code downloaded into Apache’s web directory.

---

### Step 9 – Update Database Host in App Configuration

Replace old IP (`172.20.1.101`) with `db`:

```bash
sudo sed -i 's/172.20.1.101/db/g' /var/www/html/index.php
```

✅ Points the web server to use the **remote database server**.

---

### Step 10 – Verify the Application

* Click the **E-Commerce Application** tab in the terminal interface (port **80**)
* ✅ The site loads successfully and displays product data from the remote `db` server.

---

## 🏁 End of Lab

### ✅ Key Commands Summary

| Task                   | Command                                                                                    |
| ---------------------- | ------------------------------------------------------------------------------------------ |
| Connect to DB server   | `ssh db`                                                                                   |
| Install MariaDB        | `sudo yum install -y mariadb-server`                                                       |
| Start & enable MariaDB | `sudo systemctl start mariadb && sudo systemctl enable mariadb`                            |
| Enter MySQL shell      | `sudo mysql`                                                                               |
| Load SQL data          | `sudo mysql < /opt/db-load-script.sql`                                                     |
| Connect to Web server  | `ssh web`                                                                                  |
| Install Apache + PHP   | `sudo yum install -y httpd php php-mysqlnd`                                                |
| Set PHP default page   | `sudo sed -i 's/index.html/index.php/g' /etc/httpd/conf/httpd.conf`                        |
| Start & enable Apache  | `sudo systemctl start httpd && sudo systemctl enable httpd`                                |
| Clone application      | `sudo git clone https://github.com/kodekloudhub/learning-app-ecommerce.git /var/www/html/` |
| Update DB host         | `sudo sed -i 's/172.20.1.101/db/g' /var/www/html/index.php`                                |

---

### 💡 Notes / Tips

* `%` in `'ecomuser'@'%'` allows **remote connections from any host** — in production, restrict this for security.
* Always **flush privileges** after user or permission changes in MySQL.
* When editing Apache or PHP files, use `sed` for quick and safe replacements.
* Default Apache port: **80**.
* To troubleshoot connectivity, check:

  ```bash
  sudo netstat -tulnp | grep 3306  # confirm DB is listening
  sudo firewall-cmd --list-all     # confirm access rules
  ```

---

### ✅ References

* [MariaDB Remote Access Docs](https://mariadb.com/kb/en/configuring-mariadb-for-remote-client-access/)
* [Apache HTTPD Configuration Guide](https://httpd.apache.org/docs/current/configuring.html)
* [PHP MySQL Integration](https://www.php.net/manual/en/book.mysqli.php)
* [KodeKloud – E-Commerce LAMP App GitHub Repo](https://github.com/kodekloudhub/learning-app-ecommerce)
