# ERPNext 14 Installation
This installation uses the stable version of frappe framework(14.1.0) and erpnext runtime(14.0.0).</br>
*You can skip steps 1 and 2 if you're installing ERPNext directly to your host machine.*
## 1. Virtual Machine
*Note: Deploying the ERP System inside a virtual machine is the simplest and easiest procedure. Through the use of snapshots and clone, it is easier to backup and restore the ERP system image.*

Install VirtualBox.
Create a new virtual machine <br/>
Type: Linux, Version: Ubuntu Server(64-bit) *Note: Tested on 22.04 LTS*<br/>
Memory size: 4096 MB<br/>
File size: 120 GB<br/>
Hard disk file type: VDI<br/>
Storage on physical hard disk: Dynamically allocated<br/>
Processor count: 2, Enable PAE/NX<br/>
Video Memory: 8MB<br/>
Graphics Controller: VBoxVGA<br/>
## 2. Guest Operating System

**Your name**: Administrator<br/>
**Computer name**: ubuntu<br/>
**username**: administrator<br/>
**password**: YOUR_PASSWORD<br/>

Wait for the setup to finish. Reboot.

## 3. Frappe framework/ERPNext Installation
### Pre-requisites
Launch Terminal then run the following commands:

```bash
sudo su

apt-get install libffi-dev python3-pip python3-dev python3-testresources libssl-dev wkhtmltopdf gcc g++ make redis python3.10-venv -y
```

### 4. Database Installation and Configuration

#### Install MariaDB Server and Client
```bash
apt-get install mariadb-server mariadb-client -y
```

The command above will install both server and client automatically.
```bash
mysql_secure_installation
```

The above command will prompt for the following options:

Enter current password for root (enter for none): **(Enter)**<br/>
Switch to `unix_socket authentication`: **n**<br/>
Change the `root` password?: **y**<br/>
New password: YOUR_PASSWORD<br/>
Remove anonymous user?: **y**<br/>
Disallow root login remotely: **y**<br/>
Remove test database and access to it?: **y**<br/>
Reload privilege tables now? **y**<br/>

#### MySQL Development files
```bash
apt-get install libmysqlclient-dev
```

#### MariaDB Configuration
Login to MariaDB console:
```bash
mysql -u root -p
```

Verify MariaDB authentication plugin:
```SQL
MariaDB > use mysql;
MariaDB [mysql]> select User, plugin from user;
```

The result should be similar to this:
| User | plugin |
| -----| -----|
| root | mysql_native_password |

```SQL
MariaDB [mysql] > flush privileges;
MariaDB [mysql] > exit;
```

Edit the file: 
```bash
nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Add or modify the following lines:
```ini
[mysqld]
innodb-file-format=barracuda
innodb-file-per-table=1
innodb-large-prefix=1
character-set-client-handshake=FALSE
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
```

Save the file, then restart MariaDB service:

```bash
systemctl restart mariadb
```

### 5. Create a User for ERPNext

Create a new user named `erpnext`:

```bash
useradd -m -s /bin/bash erpnext
```

Set password for `erpnext` user:

```bash
passwd erpnext
```

Add `erpnext` user to `sudo group` so that it can execute `sudo` command:

```bash
usermod -aG sudo erpnext
```

Login the `erpnext` user and set up the environment variables:

```bash
su - erpnext
sudo nano ~/.bashrc
```

Add the following line:

```bash
PATH=$PATH:~/.local/bin/
```

Save and close the file, then activate the environment variable:

```bash
source ~/.bashrc
```
### 6. Install curl
```bash
sudo apt install curl
```

### 7. Install Node.js
```bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.39.1/install.sh | bash

source ~/.bashrc

nvm install 16
```
### 8. Install Yarn using NPM
```bash
erpnext$:npm install -g yarn
```

### 9. Install ERPNext
Login `erpnext` user
```bash
su - erpnext
```

Create a new directory for `bench`.
```bash
sudo mkdir /opt/bench
```

Grant permission to `erpnext user`:
```bash
sudo chown -R erpnext:erpnext /opt/bench
```

Change directory to your newly created folder. Clone bench.
```bash
cd /opt/bench
git clone https://github.com/frappe/bench bench-repo
```

Install bench.
```bash
pip3 install -e bench-repo
```

Initialize frappe.
```bash
bench init erpnext --frappe-branch version-14
```

Change directory to the newly created folder. Create new site for your desired domain.
```bash
cd erpnext
bench new-site YOUR_SITE
```

Get the ERPNext app with version 14.
```bash
bench get-app --branch version-14 erpnext
```

If you encounter an error regarding payments app, get payments app and install

``` shell
bench get-app payments
bench --site YOUR_SITE install-app payments
```

Install erpnext app in your site.
```bash
bench --site YOUR_SITE install-app erpnext
```

### 10. Production Setup
Install `supervisor`, `nginx` and `nodejs`.
```bash
sudo apt-get install supervisor nginx nodejs
```

Install `frappe-bench` from `pip`.
```bash
sudo pip3 install frappe-bench
```

Setup for production environment. Run this twice.
```bash
sudo ~/.local/bin/bench setup production erpnext
```

Note: If you encountered an error with `nginx` because it is not running yet, simply start it by executing the command below:
``` bash
sudo systemctl start nginx
```

Done.

Check `supervisor` status to verify that `frappe/erpnext` services are running.
```bash
sudo supervisorctl status

erpnext-redis:erpnext-redis-cache                RUNNING
erpnext-redis:erpnext-redis-queue                RUNNING
erpnext-redis:erpnext-redis-socketio             RUNNING
erpnext-web:erpnext-frappe-web                   RUNNING
erpnext-web:erpnext-node-socketio                RUNNING
erpnext-workers:erpnext-frappe-default-worker-0  RUNNING
erpnext-workers:erpnext-frappe-long-worker-0     RUNNING
erpnext-workers:erpnext-frappe-schedule          RUNNING
erpnext-workers:erpnext-frappe-short-worker-0    RUNNING
```
