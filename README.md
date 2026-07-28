# v-profile-deployment (locally)

![Screenshot from 2024-07-21 21-22-53](https://github.com/user-attachments/assets/a1dc8b31-7e0b-4446-81ad-c9fab150f355)


NOTE : This is manual method, I will upload the files that automate the process


 
# Introduction

Welcome to the v-profile project! This repository contains a comprehensive profile management system tailored to provide an intuitive and seamless experience for deploying user profiles in your application. The v-profile project is designed with scalability, security, and customization in mind, making it an ideal solution for a variety of applications, including social networks, professional networking sites, and more.

# Deployment Guide
This section provides a step-by-step guide to deploying the v-profile project in your preferred environment. Follow the instructions below to set up and deploy the project quickly and efficiently

# Prerequisites
Before you begin, ensure you have met the following requirements:

1- Oracle VM VirtualBox: A powerful x86 and AMD64/Intel64 virtualization product for enterprise as well as home use.
<br>
2- Vagrant: A tool for building and managing virtual machine environments in a single workflow.
<br>
3- Vagrant Plugins: Necessary plugins to extend Vagrant's functionality. Install the hostmanager plugin by executing the following command in your terminal: 
<br>
![Screenshot from 2024-07-21 23-10-16](https://github.com/user-attachments/assets/b9000f82-b7f5-4f79-a60e-87c8c7f97fe7)

<br>
4- Git bash or equivalent editor

# VM Setup
Follow these steps to set up the virtual machine:
<br>
![Screenshot from 2024-07-21 21-31-46](https://github.com/user-attachments/assets/5d8568b8-26c2-4596-998a-26363276af6c)
<br>
1- git clone https://github.com/hkhcoder/vprofile-project.git 
<br>
2- cd vagrant
<br>
3- git checkout main
<br>
4- cd vagrant/Manual_provisioning
<br>
5- Bring up vm’s 
<br>
![Screenshot from 2024-07-21 21-36-40](https://github.com/user-attachments/assets/5a0053dd-89f9-4971-a852-6b110f8e63d8)
<br>
# Important Notes
1- Bringing up all the VMs may take a long time depending on various factors such as system performance, internet speed, and more.
<br>
2- All the vm’s hostname and /etc/hosts file entries will be automatically updated.
<br>
3- take care of spaces between commands 

# Service Setup Order
To ensure proper setup and functioning, the services should be set up in the following order:

1-MySQL (Database SVC)
<br>
2-Memcache (DB Caching SVC)
<br>
3-RabbitMQ (Broker/Queue SVC)
<br>
4-Tomcat (Application SVC)
<br>
5-Nginx (Web SVC)

# 1. MYSQL Setup
<h3> 1- Login to the db vm </h3>
$ vagrant ssh db01
<br>
$ sudo -i
<br>
<h3> 2- Update OS with latest patches </h3>
# yum update -y
<br>
<h3> 3- Set Repository </h3>
# yum install epel-release -y
<br>
<h3> 4- Install Maria DB Package </h3>
# yum install git mariadb-server -y
<br>
<h3> 5- Starting & enabling mariadb-server </h3>
# systemctl start mariadb
# systemctl enable mariadb
<br>
<h3> 6- RUN mysql secure installation script.</h3>
# mysql_secure_installation
<br>
<p>NOTE: Set db root password, I will be using admin123 as password</p>
<br>









![Screenshot from 2024-07-21 22-00-05](https://github.com/user-attachments/assets/64a7d8f9-a9b5-41d4-9673-27000ac9958f)













<h3> 7- Set DB name and users.</h3>
# mysql -u root -padmin123
<br>
mysql> create database accounts;
<br>
mysql> grant all privileges on accounts.* TO 'admin'@'%' identified by 'admin123';
<br>
mysql> FLUSH PRIVILEGES;
<br>
mysql> exit;
<br>
<h3> 8- Download Source code & Initialize Database.</h3>
# git clone -b main https://github.com/hkhcoder/vprofile-project.git
<br>
# cd vprofile-project
<br>
# mysql -u root -padmin123 accounts < src/main/resources/db_backup.sql
  <br>
# mysql -u root -padmin123 accounts
  <br>
 mysql> show tables;
  <br>
mysql> exit;
<br>
h3> 9- Restart mariadb-server </h3>
# systemctl restart mariadb
  <br>
<h2>well done you just setuped MYSQL </h2>

# 2.MEMCACHE SETUP
<h3>1- Login to the Memcache vm</h3>
$ vagrant ssh mc01
<br>
$ sudo -i
<br>
<h3>2- Update OS with latest patches </h3>
# yum update -y
<br>
<h3>3- Install, start & enable memcache on port 11211</h3>
# sudo dnf install epel-release -y
<br>
# sudo dnf install memcached -y
<br>
# sudo systemctl start memcached
<br>
# sudo systemctl enable memcached
<br>
# sudo systemctl status memcached
<br>
# sed -i 's/127.0.0.1/0.0.0.0/g' /etc/sysconfig/memcached
<br>
# sudo systemctl restart memcached
<br>
<h2>well done you just setuped MEMCACHE </h2>

# 3.RABBITMQ SETUP
<h3>1- Login to the RabbitMQ vm </h3>
$ vagrant ssh rmq01
<br>
$ sudo -i
<br>
<h3>2- Update OS with latest patches</h3>
# yum update -y
<br>
<h3>3- Set EPEL Repository </h3>
# yum install epel-release -y
<br>
<h3>4- Install Dependencies</h3>
# sudo yum install wget -y
<br>
# cd /tmp/
<br>
# dnf -y install centos-release-rabbitmq-38
<br>
# dnf --enablerepo=centos-rabbitmq-38 -y install rabbitmq-server
<br>
# systemctl enable --now rabbitmq-server
<br>
<h3>5- Setup access to user test and make it admin</h3>
# sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
  <br>
# sudo rabbitmqctl add_user test test
  <br>
# sudo rabbitmqctl set_user_tags test administrator
<br>
# sudo systemctl restart rabbitmq-server
<br>
<h2>well done you just setuped RABBITMQ </h2>

# 4.TOMCAT SETUP

<h3>1- Login to the tomcat vm</h3>
$ vagrant ssh app01
<br>
$ sudo -i
<br>
<h3>2- Update OS with latest patches</h3>
# yum update -y
<br>
<h3>3- Set Repository</h3>
# yum install epel-release -y
<br>
<h3>4- Install Dependencies</h3>
# dnf -y install java-11-openjdk java-11-openjdk-devel
<br>
# dnf install git maven wget -y
<br>
<h3>5- Change dir to /tmp</h3>
# cd /tmp/
<br>
<h3>6- Download & Tomcat Package</h3>
# wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.75/bin/apache-tomcat-9.0.75.tar.gz
<br>
# tar xzvf apache-tomcat-9.0.75.tar.gz
<br>
<h3>7- Add tomcat user</h3>
# useradd --home-dir /usr/local/tomcat --shell /sbin/nologin tomcat
<br>
<h3>8- Copy data to tomcat home dir</h3>
# cp -r /tmp/apache-tomcat-9.0.75/* /usr/local/tomcat/
<br>
<h3>9- Make tomcat user owner of tomcat home dir</h3>
# chown -R tomcat.tomcat /usr/local/tomcat
<br>
<h3> 10- Create tomcat service file</h3>
# vi /etc/systemd/system/tomcat.service
<br>
<h4>Update the file with below content</h4>
[Unit]
<br>
Description=Tomcat
<br>
After=network.target
<br>
[Service]
<br>
User=tomcat
<br>
WorkingDirectory=/usr/local/tomcat
<br>
Environment=JRE_HOME=/usr/lib/jvm/jre
<br>
Environment=JAVA_HOME=/usr/lib/jvm/jre
<br>
Environment=CATALINA_HOME=/usr/local/tomcat
<br>
Environment=CATALINE_BASE=/usr/local/tomcat
<br>
ExecStart=/usr/local/tomcat/bin/catalina.sh run
<br>
ExecStop=/usr/local/tomcat/bin/shutdown.sh
<br>
SyslogIdentifier=tomcat-%i
<br>
[Install]
<br>
WantedBy=multi-user.target
<br>

<h3>11- Reload systemd files</h3>
# systemctl daemon-reload
<br>
<h3> 12- Start & Enable service</h3>
# systemctl start tomcat
<br>
# systemctl enable tomcat
<br>

# CODE BUILD & DEPLOY (app01)
<h3>1- Download Source code</h3>
# git clone -b main https://github.com/hkhcoder/vprofile-project.git
<br>
<h3>2- Update configuration</h3>
# cd vprofile-project
<br>
# vim src/main/resources/application.properties
<br>
# Update file with backend server details
<h3>3- Build code</h3>
Run below command inside the repository (vprofile-project)
<br>
# mvn install
<br>
<h3>4- Deploy artifact</h3>
# systemctl stop tomcat 
<br>
# rm -rf /usr/local/tomcat/webapps/ROOT*
<br>
# cp target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
<br>
# systemctl start tomcat
<br>
# chown tomcat.tomcat /usr/local/tomcat/webapps -R
<br>
# systemctl restart tomcat
<br>
<h2>well done you just setuped TOMCAT </h2>

# 5.NGINX SETUP
<h3> 1- Login to the Nginx vm</h3>
$ vagrant ssh web01
<br>
$ sudo -i
<br>
<h3>2- Update OS with latest patches</h3>
# apt update
<br>
# apt upgrade
<br>
<h3>3- Install nginx</h3>
# apt install nginx -y
<br>
<h3> 4- Create Nginx conf file</h3>
# vi /etc/nginx/sites-available/vproapp
<br>
Update with below content: 
<br>
<br>
upstream vproapp {
server app01:8080;
}
server {
listen 80;
location / {
proxy_pass http://vproapp;
}
}
<br>
<h3>5- Remove default nginx conf</h3>
# rm -rf /etc/nginx/sites-enabled/default
<br>
<h3>6- Create link to activate website</h3>
# ln -s /etc/nginx/sites-available/vproapp  /etc/nginx/sites-enabled/vproapp
<br>
<h3>6- Restart Nginx</h3>
# systemctl restart nginx

<h2>well done you just setuped NGINX </h2>


<h1> congratulations you just deployed the V-PROFILE application manually  </h1>





























































































