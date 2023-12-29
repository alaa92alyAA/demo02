# 00-Project

Welcome to the first workshop in the DevOpsWorkshop series! 🚀
A Java fullstack app that functions as a mini social media platform.

## Overview

This repository serves as the starting point for our DevOpsSquad workshop series, focusing on setting up the infrastructure for a Java Full Stack project. In this workshop, we'll leverage Vagrant to orchestrate five virtual machines for various components of our application:

- Ngnix (Load Balancer)
- Tomcat (Web Server)
- RabbitMQ (Message Broker)
- Memcache (DB Caching)
- MySql/MariaDB (DB)

## Directories Structure

- **code directory:** Includes the source code.
- **devops directory:** Encompasses all DevOps-related work.

**Note:** The "devops" directory houses our DevOps-works.

## Getting Started

### Prerequisite

- Oracle VM Virtualbox
- Vagrant
- Vagrant plugins: execute the below command in your computer to install hostmanager plugin

```bash
vagrant plugin install vagrant-hostmanager
```

- Git bash or equivalent editor

### Follow these steps to set up the workshop environment on your local machine:

1. Clone this repository by executing this command:

```bash
git clone https://github.com/DevOpsSquad-Org/00-Project.git
```

2. Open the cloned repo in your fav. IDE.

3. Navigate to the devops directory, where the Vagrantfile lives:

```bash
cd devops
```

4. Use Vagrant to spin up the virtual machines:

```bash
vagrant up
```

Last command will run the instructions within the Vagrantfile in which will create the five VMs.

**Note** Bringing up all the VMs may take a long time based on various factors. If the VM setup stops in the middle, re-run the past command again.

## Setting Up VMs

After running `vagrant up`, five virtual machines (VMs) will be created. Follow the steps below to set up each VM, however it must get done in the following order:

### 1. MySQL/MariaDB Setup

- **Description:** MySQL/MariaDB database for data storage.

- **Access:**

  - **URL:** [http://localhost:3306](http://localhost:3306)

- **Setup Steps:**

1. **Login to the DB VM:**

```bash
vagrant ssh db01
```

2. **Verify Hosts Entry:**
   Check if entries are missing in `/etc/hosts`. Update it with IP and hostnames if necessary:

```bash
cat /etc/hosts
```

3. **Update OS with Latest Patches:**

```bash
yum update -y
```

4. **Set Repository:**

```bash
yum install epel-release -y
```

5. **Install MariaDB Package:**

```bash
yum install git mariadb-server -y
```

6. **Starting & Enabling mariadb-server:**

```bash
systemctl start mariadb
systemctl enable mariadb
```

7. **Run MySQL Secure Installation Script:**

```bash
mysql_secure_installation
```

**Security Tip:** By default, the MySQL root account has no password, posing a significant security risk. It is imperative to configure a secure password for the root account to enhance the overall security of the MySQL installation.

<details>
<summary>Set the DB root password, we'll use admin123 as the password</summary>

      Set root password? [Y/n] <span style="color: red;">Y</span>
      New password:
      Re-enter new password:
      Password updated successfully!
      Reloading privilege tables..
      Success!

      By default, a MariaDB installation has an anonymous user, allowing anyone
      to log into MariaDB without having to have a user account created for
      them. This is intended only for testing, and to make the installation
      go a bit smoother. You should remove them before moving into a
      production environment.

      Remove anonymous users? [Y/n] <span style="color: red;">Y</span>
      ... Success!

      Normally, root should only be allowed to connect from 'localhost'. This
      ensures that someone cannot guess at the root password from the network.

      Disallow root login remotely? [Y/n] <span style="color: red;">n</span>
      ... skipping.

      By default, MariaDB comes with a database named 'test' that anyone can
      access. This is also intended only for testing and should be removed
      before moving into a production environment.

      Remove test database and access to it? [Y/n] <span style="color: red;">Y</span>

      - Dropping test database...
      ... Success!
      - Removing privileges on the test database...
      ... Success!

      Reloading the privilege tables will ensure that all changes made so far
      will take effect immediately.

      Reload privilege tables now? [Y/n] <span style="color: red;">Y</span>
      ... Success!

</details>

#### Create DB & set full privileges to the admin user

```bash
mysql> mysql -u root -padmin123
mysql> create database accounts;
mysql> grant all privileges on accounts.* TO 'admin'@'%' identified by 'admin123';
mysql> FLUSH PRIVILEGES;
mysql> exit;
```

#### Navigate to the code directory

```
cd ../code
```

#### Populate DB

```bash
mysql> use accounts
mysql> mysql -u root -padmin123 accounts < src/main/resources/db_backup.sql
mysql> mysql -u root -padmin123 accounts
```

#### Check result

```bash
mysql> show tables;
mysql> exit;
```

#### Restart mariadb-server

```bash
systemctl restart mariadb
```

#### Starting the firewall and allowing the mariadb to access from port no. 3306

```bash
systemctl start firewalld
systemctl enable firewalld
firewall-cmd --get-active-zones
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
systemctl restart mariadb
```

### 2. Memcache VM

- **Description:** Memcache cache for optimizing data retrieval.

- **Access:**

  - **URL:** [http://192.168.56.14:6379](http://192.168.56.14:6379)

- **Setup Steps:**

  1. **Login to the Memcache vm**

  ```bash
  vagrant ssh mc01
  ```

  2. **Verify Hosts entry, if entries missing update the it with IP and hostnames**

  ```bash
  cat /etc/hosts
  ```

  3. **Update OS with latest patches**

  ```bash
  yum update -y
  ```

  4. **Install, start & enable memcache on port 11211**

  ```bash
  sudo dnf install epel-release -y
  sudo dnf install memcached -y
  sudo systemctl start memcached
  sudo systemctl enable memcached
  sudo systemctl status memcached
  sed -i 's/127.0.0.1/0.0.0.0/g' /etc/sysconfig/memcached
  sudo systemctl restart memcached
  ```

  5. **Starting the firewall and allowing the port 11211 to access memcache**

  ```bash
  firewall-cmd --add-port=11211/tcp
  firewall-cmd --runtime-to-permanent
  firewall-cmd --add-port=11111/udp
  firewall-cmd --runtime-to-permanent
  sudo memcached -p 11211 -U 11111 -u memcached -d
  ```

### 3. RabbitMQ VM

- **Description:** RabbitMQ for message brokering.
- **Access:**

  - **URL:** [http://192.168.56.16:15672](http://192.168.56.16:15672)

- **Setup Steps:**

  1. **Login to the RabbitMQ vm**

  ```bash
  vagrant ssh rmq01
  ```

  2. **Verify Hosts entry, if entries missing update the it with IP and hostnames**

  ```bash
  cat /etc/hosts
  ```

  3. **Update OS with latest patches**

  ```bash
  yum update -y
  ```

  4. **Set EPEL Repository**

  ```bash
  yum install epel-release -y
  ```

  5. **Install Dependencies**

  ```bash
  sudo yum install wget -y
  cd /tmp/
  dnf -y install centos-release-rabbitmq-38
  dnf --enablerepo=centos-rabbitmq-38 -y install rabbitmq-server
  systemctl enable --now rabbitmq-server
  ```

  6. **Setup access to user test and make it admin**

  ```bash
  sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
  sudo rabbitmqctl add_user test test
  sudo rabbitmqctl set_user_tags test administrator
  sudo systemctl restart rabbitmq-server
  ```

  7. **Starting the firewall and allowing the port 5672 to access rabbitmq**

  ```bash
  firewall-cmd --add-port=5672/tcp
  firewall-cmd --runtime-to-permanent
  sudo systemctl start rabbitmq-server
  sudo systemctl enable rabbitmq-server
  sudo systemctl status rabbitmq-server
  ```

### 4. Tomcat VM

- **Description:** Apache Tomcat for deploying and running Java applications.
- **Access:**

  - **URL:** [http://192.168.56.12:8080](http://192.168.56.12:8080)

- **Setup Steps:**

  1. **Login to the tomcat vm**

  ```bash
  vagrant ssh app01
  ```

  2. **Verify Hosts entry, if entries missing update the it with IP and hostnames**

  ```bash
  cat /etc/hosts
  ```

  3. **Update OS with latest patches**

  ```bash
  yum update -y
  ```

  4. **Set Repository**

  ```bash
  yum install epel-release -y
  ```

  5. **Install Dependencies**

  ```bash
  dnf -y install java-11-openjdk java-11-openjdk-devel
  dnf install git maven wget -y
  ```

  6. **Change dir to /tmp**

  ```bash
  cd /tmp/
  ```

  7. **Download & unzip Tomcat Package**

  ```bash
  wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.75/bin/apache-tomcat-9.0.75.tar.gz
  tar xzvf apache-tomcat-9.0.75.tar.gz
  ```

  8. **Add tomcat user**

  ```bash
  useradd --home-dir /usr/local/tomcat --shell /sbin/nologin tomcat
  ```

  9. **Copy data to tomcat home dir**

  ```bash
  cp -r /tmp/apache-tomcat-9.0.75/* /usr/local/tomcat/
  ```

  10. **Make tomcat user owner of tomcat home dir**

  ```bash
  chown -R tomcat.tomcat /usr/local/tomcat
  ```

  **Setup systemctl command for tomcat**

  1. **Create tomcat service file**

  ```bash
  vi /etc/systemd/system/tomcat.service
  ```

  2. **Update the file with below content**

  ```bash
  [Unit]
  Description=Tomcat
  After=network.target
  [Service]
  User=tomcat
  WorkingDirectory=/usr/local/tomcat
  Environment=JRE_HOME=/usr/lib/jvm/jre
  Environment=JAVA_HOME=/usr/lib/jvm/jre
  Environment=CATALINA_HOME=/usr/local/tomcat
  Environment=CATALINE_BASE=/usr/local/tomcat
  ExecStart=/usr/local/tomcat/bin/catalina.sh run
  ExecStop=/usr/local/tomcat/bin/shutdown.sh
  SyslogIdentifier=tomcat-%i
  [Install]
  WantedBy=multi-user.target
  ```

  3. **Reload systemd files**

  ```bash
  systemctl daemon-reload
  ```

  4. **Start & Enable service**

  ```bash
  systemctl start tomcat
  systemctl enable tomcat
  ```

  5. **Enabling the firewall and allowing port 8080 to access the tomcat**

  ```bash
  systemctl start firewalld
  systemctl enable firewalld
  firewall-cmd --get-active-zones
  firewall-cmd --zone=public --add-port=8080/tcp --permanent
  firewall-cmd --reload
  ```

  #### <div style="text-align: center">CODE BUILD & DEPLOY (app01)</div>

  1. **Update configuration**

  ```bash
  cd code/
  vim src/main/resources/application.properties
  Update file with backend server details
  ```

  2. **Run below command inside the repository (code)**

  ```bash
  mvn install
  ```

  3. **systemctl stop tomcat**

  ```bash
  systemctl stop tomcat
  rm -rf /usr/local/tomcat/webapps/ROOT*
  cp target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
  systemctl start tomcat
  chown tomcat.tomcat /usr/local/tomcat/webapps -R
  systemctl restart tomcat
  ```

### 5. Nginx VM

- **Description:** Nginx for load balancing and serving static content.
- **Access:**

  - **URL:** [http://192.168.56.11](http://192.168.56.11)

- **Setup Steps:**

1. **Login to the Nginx vm**

```bash
vagrant ssh web01
sudo -i
```

2. **Verify Hosts entry, if entries missing update the it with IP and hostnames**

```bash
cat /etc/hosts
```

3. **Update OS with latest patches**

```bash
apt update
apt upgrade
```

4. **Install nginx**

```bash
apt install nginx -y
```

5. **Create Nginx conf file**

```bash
vi /etc/nginx/sites-available/vproapp
```

6. **Update with below content**

```bash
upstream vproapp {
server app01:8080;
}
server {
listen 80;
location / {
proxy_pass http://vproapp;
}
}
```

7. **Remove default nginx conf**

```bash
rm -rf /etc/nginx/sites-enabled/default
```

8. **Create link to activate website**

```bash
ln -s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/vproapp
```

9. **Restart Nginx**

```bash
systemctl restart nginx
```

## Screenshots

<img src="./devops/infra.png" alt="Infrastructure map" width="500" height="400">

Credit for the Java App: [Vprofile Project on GitHub](https://github.com/hkhcoder/vprofile-project/tree/main)
