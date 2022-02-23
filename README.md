# Multinode Cloudera Cluster on CentOS7s

This is a complete guide on how to setup a multi-node cloudera cluster on CentOS7 using VirtualBox. The cluster is developed to design and 
implemenet Bigdata Solutions and provide all the services that are required to serve the purpose.

**`Note:`** During the installation, if you are facing any issues, please visit the `Issues` part of the documentation as you might be facing the same issue that I faced. 

<details>
<summary>Downloading Required Stuff</summary>

## 1. Downloading Required Stuff:
* [CentOS7 ISO Image](https://www.centos.org/download/)
* [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

Now open virtual box and click on the **new** button in order to create a new VM (Virtual Machine).

</details>


<details>
<summary>Creating a VM using VirtualBox</summary>

## 2. Creating a VM using VirtualBox
* Click on the new button.
* Add the name of the VM as per your choice.
       1. Type: Linux
       2. Version: Red-Hat (64 bit)
* Assign memory according to the current Memory installed on your system. (4GB Recommended)
* Select **Create a virtual hard disk now** option
* Select **VDI (Virtualbox Disk Image)**
* Select **Dynamically Allocated** 
* Assign the storage according to the available storage on your system. (Atleast 8GB)
* Creating Adapters
       1. Goto File > Host Network Manager
       2. There are 3 buttons shown namely: **create, remove** and **properties**
       3. Create 2 Adapters and name them **vboxnet0** and **vboxnet1**
* Now go back to the VirtualBox and select the VM that is just created and click on the **settings** button.
* Goto **Adapter** tab.
       1. Under **Adapter2** tab, select the Enable Network Adapter checkbox and assign vboxnet0.
       2. Goto **Adapter3** tab, select the Enable Network Adapter checkbox and assign vboxnet1.
* Now goto **Storage** tab in settings and Select the **empty** disk. on the right side, add the CentOS7 ISO Image.
* Click **OK** and you are done.
* Now run the VM. 

After the creation of the VM is done, you can repeat step 2 in order to create as many nodes as you need for the cluster.
Note that the more nodes you create, the more resources will be consumed from your system. 
For a normal cluster, you need atleast **1 Master and 2 slaves**

</details>

<details>
<summary>Installation of CentOS7 on VM</summary>

## 3. Installation of CentOS7 on VM
   The Installation is relatively simple.
* Select the language.
* Add the appropriate date and time.
* Goto **Network and Hostname** and select all 3 adapters
* Click on **Begin Installation**
* After the Installation add the **Root Password**. (Creating a user is not that necessary)

After the Installation is done, the system will ask to reboot. after the reboot it will ask for the following credentials
* username: **root**
* password: **password that you just set in step 5**

after entering the following credentials, you are currently in the localhost.

</details>

<details>
<summary>Setup Static IP Address</summary>

## 4. Setup Static IP Address
   It is important to setup a static ip address in order to access the node on the network.

* Before setting the IP address, Goto VirtualBox Settings, file>Host Network Manager, and add an adapter(vboxnet0) if not available. Make sure the IP(192.168.XX.YY) of the adapter and the IP of the VM that you are about to set are same in XX.

* To set a static IP address, use the following command to add configurations:

       vi /etc/sysconfig/network-scripts/ifcfg-enp0s8

Note that **enp0s8 is the name of the adapter in my machine**
* You will be prompted with the following contents of the file:

       TYPE=Ethernet
       BOOTPROTO=dhcp
       DEFROUTE=yes
       PEERDNS=yes
       PEERROUTES=yes
       IPV4_FAILURE_FATAL=no
       IPV6INIT=yes
       IPV6_AUTOCONF=yes
       IPV6_DEFROUTE=yes
       IPV6_PEERDNS=yes
       IPV6_PEERROUTES=yes
       IPV6_FAILURE_FATAL=no
       NAME=enp0s8
       UUID=c3fb64f0-6ff5-4294-9354-8e1279971d7e
       DEVICE=enp0s8
       ONBOOT=no

* make the following changes, set:

       BOOTPROTO=static
       ONBOOT=yes
       IPADDR=192.168.XXX.XXX
       NETMASK=255.255.255.0

* restert the network by executing the command:

       systemctl restart network

* Static IP will be set after the restart.

## Update Yum
Update yum using the following command:

       yum update

</details>

<details>
<summary>Check the availlable memory and storage</summary>

## Check the availlable memory and storage
* to check the available memory of the system, use the following command:

       free -m

* to check the available storage, use the following command:

       df -h

</details>

<details>
<summary>Maximum number of open file requirementss</summary>

## 5. Maximum number of open file requirements
* To check the current number of available number of open file descriptors, use the following command:

       ulimit -Sn
       ulimit -Hn

* To set the value of open file descriptors. (recommended: 10000)

       ulimit -n 10000

</details>

<details>
<summary>Changing the hostname of the machine</summary>

## 6. Changing the hostname of the machine
* By default, the hostname is: `localhost`
* To check the current running hostname, run the following command:

       hostname -f

* To check the details of hostname, run the command:

       hostnamectl

* To change the hostname of the system:

       hostnamectl set-hostname hostname

* reboot the system using the `reboot` command to make the changes.

</details>

<details>
<summary>Setting the hostname in the hostfile</summary>

## 7. Setting the hostname in the hostfile
* To view the hosts in the hosts file, enter the following command:

       vi /etc/hosts

* add the hosts in a fashion of adding an IP of a node and the hostname, the sample is given below:

       127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
       ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
       192.168.XXX.XXX master.company.com master
       192.168.YYY.XXX datanode1.company.com datanode1 
       192.168.XXX.YYY datanode2.company.com datanode2

* in the above contents of the file, `master`, `datanode1` and `datanode2` are the names of the hosts.
* make sure to add these settings on each host of the cluster in order for the hosts to work.
* once these hosts are set, we can SSH any hosts using the hostname instead of the IP Address

       ssh master
       ssh datanode1
       ssh datanode2

</details>

<details>
<summary>Setup Password-less SSH</summary>

## 8. Setup Password-less SSH (Not Important)
Password-less enables each node in the cluster to ssh any node without authentication or password. follow the steps given below in order to setup passwordless SSH
* Use the following command to generate a key. 

       ssh-keygen

* Press `enter` for all the default options provided to save the key. The command will generate 2 keys, a public key and a private key.

       Generating public/private rsa key pair.
       Enter file in which to save the key (/root/.ssh/id_rsa): 
       Created directory '/root/.ssh'.
       Enter passphrase (empty for no passphrase): 
       Enter same passphrase again: 
       Your identification has been saved in /root/.ssh/id_rsa.
       Your public key has been saved in /root/.ssh/id_rsa.pub.
       The key fingerprint is:
       97:0e:4c:a4:21:97:42:58:86:11:f1:74:e2:5a:82:cb root@localhost.localdomain
       The key's randomart image is:
       +--[ RSA 2048]----+
       |  +OB +..        |
       | .o=.=.+         |
       |. . +.. .        |
       |.. +   o   .     |
       |.E.     S o      |
       |         +       |
       |          .      |
       |                 |
       |                 |
       +-----------------+

* Now enter the following command to copy the generated key to the `authorized_keys` file:

       cat .ssh/id_rsa.pub >> authorized_keys

* now change the following permissions:

       chmod 700 ~/.ssh
       chmod 600 ~/authorized_keys

* Now use the following command to copy the key to the target host in order to set the password-less SSH:

       ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.XYX.XYX

* Note that in the command above, the target host can also be accessed using the hostname instead of using the IP.
* Enter the password of the target host. And now you can access 192.168.XYX.XYX from the current host
* repeat the above process in order to setup the Password-less SSH on other nodes as well.

</details>

<details>
<summary>Setup NTP on the server and the browser host</summary>

## 9. Setup NTP on the server and the browser host
NTP ensures that the clocks of the nodes in the cluster are synchronized. follow the steps given below in order to set the NTP on each node:
* Install NTP using the command: 

       yum install -y ntp

* Enable NTP using the command; 

       systemctl enable ntpd

* following commands are used to start and restart the NTP service.

       systemctl start ntpd
       systemctl restart ntpd

* If you want to check the sync status, use the following command:

       ntpstat

* Edit the file `/etc/ntp.conf` to add the following ntp servers:

       server 1.pk.pool.ntp.org
       server 0.asia.pool.ntp.org
       server 3.asia.pool.ntp.org  

</details>

<details>
<summary>Disabling Firewalls / Configuring IP tables</summary>

## 10. Disabling Firewalls / Configuring IP tables

* execute the following 2 commands given below to disable the firewall

       systemctl disable firewalld
       service firewalld stop

</details>

<details>
<summary>Set the Network Config File</summary>

## 11. Set the Network Config File

* Change the contents of the following file by using the command:

       vi /etc/sysconfig/network

* make the following changes in the file:

       NETWORKING=yes
       HOSTNAME=localhost

</details>

<details>
<summary>Disable SELinux and PackageKit and check umask value</summary>

## 12. Disable SELinux and PackageKit and check umask value
It is important to disable SELinux for the ambari function to setup. 
* Use the following command to disable SELinux:

       setenforce 0

* to permenantly disable SELinux, make sure to set `SELINUX=disabled` in the following file to make sure that it is not enables after the system is rebooted:

       vi /etc/selinux/config

* For packagekit, set `enabled=0` in the following file:

       vi /etc/yum/pluginconf.d/refresh-packagekit.conf

* To check the current umask, run thhe following command:

       umask

* set the umask just fot the current login session: 

       umask 0022

* permenantly changing the umask for all the users: 

       echo umask 0022 >> /etc/profile

</details>

<details>
<summary>Set Swappiness</summary>

## 13. Set Swappiness 

* Cloudera recommends to set the value of swappiness between 1 and 10, based upon the strength of the machine, but its recommended to set `swappiness=1`

* execute this command to set the value of swappiness:

       sysctl vm.swappiness=1

* Also, open the file `/etc/sysctl.conf` and add the line:

       vm.swappiness=1

* use the following commands to check the swappiness:

       cat /proc/sys/vm/swappiness

</details>

<details>
<summary>Disable Huge pages</summary>

## 14. Disable Huge pages
* It is highly recommended to disable huge pages. for that, add the following lines in `/etc/rc.local`:

       echo never > /sys/kernel/mm/transparent_hugepage/defrag
       echo never > /sys/kernel/mm/transparent_hugepage/enabled

* change the permission of the file after adding the above lines using the following command:

       chmod +x /etc/rc.d/rc.local

</details>

<details>
<summary>Install JDK for cloudera manager</summary>

## 15. Install JDK for cloudera manager (all nodes)

It is important to install JDK on all nodes, use the following command to install JDK on all nodes:

       yum install java-1.8.0-openjdk

</details>

<details>
<summary> Configuring MySQL for Cloudera</summary>

## 16. Configuring MySQL for Cloudera

Installing and setting-up MySQL is different as the default database that is provided by ambari/cloudera is PostgreSQL, so follow the steps bellow to install SQL and Configuring it with Cloudera.

* We need the package called `wget` to get repositoried and download services from CentOS. Install wget using the commmand:

       yum install wget

* Now we need to download MySQL.
* **Note:** The version of ambari used in this guide is 2.7.3.0 and it does not support MySQL version 8, it has some issues. make sure to install the MySQL version 5 or 6.
* Download MySQL v57 using the following `wget` link and installation commands.

       wget https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
       md5sum mysql57-community-release-el7-9.noarch.rpm
       sudo rpm -ivh mysql57-community-release-el7-9.noarch.rpm

* Now use the following command to install MySQL:

       yum install mysql-server

* Once MySQL is downloaded, check the status of MySQL before and after starting the service, execute the following commands:

       systemctl status mysqld
       systemctl start mysqld
       systemctl status mysqld

* Once MySQL service is running, log in with the `root` user with the following command: 

       mysql -u root -p

* after the above command, it will ask for the password, if the password was not set earlier, then we get get the password using:

       grep 'password' /var/log/mysqld.log

* The first step that you have to perform is to change the password using the `ALTER USER` command, because without changing password you cannot query anything when MySQL is just installed, so execute the following command:

       ALTER USER 'root'@'localhost' IDENTIFIED BY 'Ambari123!';

* Once we are logged in, we have to create a user and set a password that matches the ambari-setup configuration and MySQL, so for that we have to reduce the level of password strength policy, for that we can use the given command to check if the `validate_password_policy` is high or not:

       SHOW VARIABLES LIKE 'validate_password%';

* Following result is shown once the above query is executed: 

       +--------------------------------------+--------+
       | Variable_name                        | Value  |
       +--------------------------------------+--------+
       | validate_password_check_user_name    | OFF    |
       | validate_password_dictionary_file    |        |
       | validate_password_length             | 8      |
       | validate_password_mixed_case_count   | 1      |
       | validate_password_number_count       | 1      |
       | validate_password_policy             | MEDIUM |
       | validate_password_special_char_count | 1      |
       +--------------------------------------+--------+

* now execute the following to reduce the `validate_password_policy` level using the following command: 

       SET GLOBAL validate_password_policy=LOW;

* now exit my sql and move old InnoDB log files `/var/lib/mysql/ib_logfile0` and `/var/lib/mysql/ib_logfile1` out of `/var/lib/mysql/` to a backup location. for that create a backup location using the command:

       mkdir innodbackup

* now move both the files from the targer directory using the following command. 

       mv /var/lib/mysql/ib_logfile0 innodbackup
       mv /var/lib/mysql/ib_logfile1 innodbackup

* install MySQL java connector using the following commands:

       wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.46.tar.gz
       tar zxvf mysql-connector-java-5.1.46.tar.gz
       mkdir -p /usr/share/java/
       cd mysql-connector-java-5.1.46
       cp mysql-connector-java-5.1.46-bin.jar /usr/share/java/mysql-connector-java.jar

* install java (recommended), on all datanodes, not on cloudera manager server.

       yum install java-1.8.0-openjdk

* Once logged in, you have to create a user for all the required users(see the users in cloudera docs), for that execute the following on MySQL: 

       CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
       GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'cloudera';

       CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
       GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY 'cloudera';

       CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
       GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY 'cloudera';

       CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
       GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY 'cloudera';

       CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
       GRANT ALL ON metastore.* TO 'hive'@'%' IDENTIFIED BY 'cloudera';

       CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
       GRANT ALL ON sentry.* TO 'sentry'@'%' IDENTIFIED BY 'cloudera';

       CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
       GRANT ALL ON nav.* TO 'nav'@'%' IDENTIFIED BY 'cloudera';
       
       CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
       GRANT ALL ON navms.* TO 'navms'@'%' IDENTIFIED BY 'cloudera';

       CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
       GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY 'cloudera';


* to check all the preveleges, use the following command: 

       SHOW GRANTS FOR '<user>'@'%';

* Now use the `ALTER USER` command again to set the same password for sql ass well:

       ALTER USER 'root'@'localhost' IDENTIFIED BY 'cloudera';

*  To check the path of java, use the command:

       which java

* Now you can login using the `ambari` user: 

       mysql -u ambari -p

* Check the current logged in user using the command:

       SELECT USER();


* in the above command, in my case, database type is `mysql`, database name and user is `cloudera`.

</details>

<details>
<summary> Installing JDBC Driver</summary>

## 17. Installing JDBC Driver (all nodes)
* install MySQL java connector on all nodes using the following commands:

       wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.46.tar.gz
       tar zxvf mysql-connector-java-5.1.46.tar.gz
       mkdir -p /usr/share/java/
       cd mysql-connector-java-5.1.46
       cp mysql-connector-java-5.1.46-bin.jar /usr/share/java/mysql-connector-java.jar

</details>

<details>
<summary>Installation of Cloudera Manager, Deamons and Agents</summary>

## 18. Installation of Cloudera Manager, Deamons and Agents
The installation of cloudera manager, deamons and agents is a bit different as the downloaded repo is not saved in the path where it can be installed, so `wget` the cloudera manager repo from the link given below:

       sudo wget https://archive.cloudera.com/cm6/6.0.0/redhat7/yum/cloudera-manager.repo -P /etc/yum.repos.d/

* Make sure to check the python version before the installation of cloudera manager, the required version is `python2.7.5`. bu default it is installed. you can check by the command:

       python --version

* once the download is done, you can check by the command `yum repolist` of `ls /etc/yum.repos.d` to ensure that the cloudera-manager.repo is present there. In my case, it was not in the right path, it was in the main root folder. we can view it by `ls -ltr` command. Now execute the following command to move the cloudera-manager.repo file in correct folder:

       mv cloudera-manager.repo /etc/yum.repos.d

* once executed, you can view the repo using the command:

       yum repolist

* now install the cloudera manager server, deamon and agent in the master node only using the command:

       yum install cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server

* before doing installations in any of the datanodes, make sure that java is installed, you can ensure the path of java "if exists":

       which java

* also you can check the version of java using the command:

       java -version

* If java is not installed, install java using the command:

       yum install java-1.8.0-openjdk

* Once java is installed, install the cloudera agents and deamons on all the datanodes using the command:

       yum install cloudera-manager-daemons cloudera-manager-agent

</details>

<details>
<summary>Connecting Cloudera manager with MySQL</summary>

## 19. Connecting Cloudera manager with MySQL
* last and most important, on master, use the following command to connect mysql with cloudera manager, if this command is not executed, the server will not run. if this command fails to execute, run this command after installing cloudera manager on the master node.

       /opt/cloudera/cm/schema/scm_prepare_database.sh <databaseType> <databaseName> <databaseUser>

* Generally the command is:

       /opt/cloudera/cm/schema/scm_prepare_database.sh mysql scm scm

</details>

<details>
<summary>Opening Cloudera Manager UI</summary>

## 20. Opening Cloudera Manager UI
Once everything is installed and setup, restart cloudera agents on all nodes using the command:

       systemctl start cloudera-scm-agent

Make sure that java is installed, JDBC is installed and mysql is running. Now run the following command to run the cloudera manager server:

       systemctl start cloudera-scm-server

Use this command to track the log file of cloudera manager server:

       sudo tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log

Open the web browser, and Access the Cloudera manager UI using the following link

       http://server.ip.address:7180


</details>

<details>
<summary>Issues</summary>

## Issues

### 1. Under Replicated Blocks
Once the installation of the cluster is done using the web interface, now a warning was showing up. When the details were checked, it showed that there were under replicated blocks in the system. 

* Under replicated blocks are the blocks/files with replication factor 3 but are actually replicated less than 3 times. To fix this issue, we need to find and delete all the files that are replicated. Run the following command to enter bash as an HDFS user in order to avoid any permission issues.

       hdfs fsck /

* once the above command is executed, it might display a long list of files that are corrupted. to delete them, execute the following command:

       hdfs dfs -rm -skipTrash /path/to/curropted/files/*

* execute the above commands again and again with the different paths of different files, and check by `hdfs fsck /` to see how many are deleted. In my case, I used the following commands:

       hdfs dfs -rm -skipTrash /user/oozie/share/lib/lib_20201028110301/sqoop/*
       hdfs dfs -rm -skipTrash /user/oozie/share/lib/lib_20201028110301/spark/*
       hdfs dfs -rm -skipTrash /user/oozie/share/lib/lib_20201028110301/pig/*
       hdfs dfs -rm -skipTrash /user/oozie/share/lib/lib_20201028110301/hive2/*
       hdfs dfs -rm -skipTrash /user/oozie/share/lib/lib_20201028110301/hive/*
       hdfs dfs -rm -skipTrash /user/oozie/share/lib/lib_20201028110301/hcatalog/*
       hdfs dfs -rm -skipTrash /user/oozie/share/lib/lib_20201028110301/distcp/*
       hdfs dfs -rm -skipTrash /user/oozie/share/lib/lib_20201028110301/oozie/*
       hdfs dfs -rm -skipTrash /user/oozie/share/lib/lib_20201028110301/mapreduce-streaming/*
       hdfs dfs -rm -skipTrash /user/oozie/share/lib/lib_20201028110301/sharelib.properties

* [Source of the solution.](https://stackoverflow.com/questions/19205057/how-to-fix-corrupt-hdfs-files)

### 2. Web Server Status
If there are any issues/errors related to getting `web server status`, then wait for some time, the issue will be resolved bu itself

### 3. Service Canary
If there are any issues/errors related to getting `[service] canary`, for example `hdfs canary` issue of `hive canary`, then wait for some time, the issue will be resolved bu itself.

### 4. Issues while creating namenode directory
If there are any issues while the `command details` section of adding the cluster, there will be a folder named `current` in `/dfs/nn` path, delete this folder using the following command and try again:

       rm -rf /dfs/nn/current

</details>

<details>
<summary>Post Installations Setups</summary>

# Post Installations Setups

## Setup Datanodes
Make sure that datanodes are installed on all the nodes of the cluster, it is important. Also it is related to the issues of `over-replicated` and `under-replicated` blocks.

**`Recommended:`** Install datanodes during the installation of the cluster. 

## Setup Zookeeper
Cloudera suggests to install at-least 3 Zookeeper servers in the cluster, so make sure to install `zookeeper server` on 3 different nodes. 

**`Recommended:`** Install 3 Zookeeper servers during the installation of the cluster. 

## Setup Kerberos
Kerberos is a network authentication protocol developed by MIT, which eliminates the need for transmision of password accross the network and removes the threat of any attack.

**`KDC (Key Distribution Center): `** KDC is the authentication server in kerberos environment. In most of the cases KDC is resided in the seperate physical server. We can devide KDC into 3 parts:

### 1. Database
This database stores users and service identities, these identities are known as `Principles`. It also stores information like encryption key, ticket validity duration and expiration date etc.

### 2. AS (Authentication Server)
This server authenticates a user and issues a `TGT (Ticket Granting Ticket)`. If you have a valid TGT, means authentication server has verified your credentials.

### 3. TGS (Ticket Granting Server)
TGS is the application server of KDC which provides service ticket. To access any hadoop service on the cluster, we need to get a service ticket from TGS.

**`Principal:`** Kerberos database maintains identities, then assigns tickets to these identities. There identities are called `principals`. A principal has 3 parts, `primary`, `instance` and `realm`. In the principal `cm/admin@METIS.COM`, `cm` is primary, `admin` is instance and `METIS.COM` is realms. The instance part is optional.
 
### 1. Install Kerberos Client on all Machines (Nodes):
* to setup kerberos, install kerberos workstation on all the nodes of the cluster. use the following commands to install the kerberos workstation:

       yum install krb5-workstation krb5-libs krb5-auth-dialog

### 2. Install Server on one Machines (Node):
* install the kerberos server in just 1 node, which will act as the kerberos server. use the following command to install kerberos server:

       yum install krb5-server

### 3. Configure Kerberos
Configuring Kerberos includes configuring the `realm` for kerberos, you can think off `realm` as a **namespace** or a **logical boundry** within which kerberos has the authority to authenticate someone.

* Now configure this file `vi /var/kerberos/krb5kdc/kdc.conf` to define realm. Perform this on the kerberos server node. Enter the file, delete all of its contents and add the contents given below:

       [kdcdefaults]
       kdc_ports = 88
       kdc_tcp_ports = 88
       
       [realms]
       METIS.COM = {
       #master_key_type = aes256-cts
       acl_file = /var/kerberos/krb5kdc/kadm5.acl
       dict_file = /usr/share/dict/words
       admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
       supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
       max_renewable_life = 7d
       }


* now we have to configure the `/etc/krb5.conf` file on every node. Use the folliowing command to configure the file: `vi /etc/krb5.conf`. Enter the file, delete all the contents of the file and paste the content given below:

       [logging]
       default = FILE:/var/log/krb5libs.log
       kdc = FILE:/var/log/krb5kdc.log
       admin_server = FILE:/var/log/kadmind.log

       [libdefaults]
       default_realm = METIS.COM
       dns_lookup_realm = false
       dns_lookup_kdc = false
       ticket_lifetime = 24h
       renew_lifetime = 7d
       forwardable = true


       default_tgs_enctypes = aes256-cts-hmac-sha1-96 aes128-cts-hmac-sha1-96 arcfour-hmac-md5
       default_tkt_enctypes = aes256-cts-hmac-sha1-96 aes128-cts-hmac-sha1-96 arcfour-hmac-md5
       permitted_enctypes = aes256-cts-hmac-sha1-96 aes128-cts-hmac-sha1-96 arcfour-hmac-md5


       [realms]
       METIS.COM = {
       kdc = node2.metis.com
       admin_server = node2.metis.com
       max_renewable_life = 7d
       }     

* **`Note:`** In the above content, the `kdc` and `admin_server` values are set to the node in which the kerberos server is installed.


### 4. Create KDC Database
* use the following command to create a KDC database and set a master password, In my case the password is `Admin@123`.

       kdb5_util create -r METIS.COM -s

### 5. Update the ACL File
* Now configure the file `/var/kerberos/krb5kdc/kadm5.acl` only one the node in which the kerberos server is installed, use the command `vi /var/kerberos/krb5kdc/kadm5.acl` to enter the file, delete all the current content and paste the following contents:

       */admin@METIS.COM      *

### 6. Create KDC Admin
* Enter the command `kadmin.local`. it wil ask you to add a principal, add `addprinc root/admin@METIS.COM` in response. then it will ask for password, the password is `admin`. type `exit` after that to exit kadmin.local.

### 7. Start Kerberos
use the following set of commands to start kerberos services:(start krb5kdc on all nodes)

       service krb5kdc start OR systemctl start krb5kdc
       service kadmin start OR systemctl start kadmin

It is also recommended to enable them, to start the service on boot.

### 8. Install JCE Security policy files (All Nodes)

To installll JCE Policy files, use the following wget command to download the required file:

       wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jce/8/jce_policy-8.zip"

once the file is downloaded, we have to unzip it, for that, use this command to install `unzip`:

       yum install unzip

Now use the following command to unzip

       unzip -o -j -q jce_policy-8.zip -d /usr/java/jdk1.8.0_141-cloudera/jre/lib/security/

### 8. Testing Kerberos

* Now goto another host, other than the kerberos server node, and then enter `kinit root/admin` command and enter the password `admin`. now enter the command `klist` to check if the ticket is available. To delete a ticket, `kdestroy` is used.

* **`Note:`** You can only `kinit` the principle that you added in the `kadmin.local`.

### Beforing enabling Kerberos from cloudera manager, read some points below:

* Cloudera manager supports MIT KDC and Active directory.
* The KDC should be configured to have non-zero ticket lifetime and renewall lifetime. CDH will not work properly if the tickets are not renewable.
* OpenLDap Client libraries should be installed on the cloudera server host if you want to use active directory. Also kerberos client liabraries should be installed on all the hosts.
* Cloudera Manager needs an account that has permissions to create another accounts in the KDC.

Enabling Kerberos on Cloudera Manager

* Goto Administration > Security and click on `enable kerberos` button. 
* Now it will ask for some permissions that are mentioned above, tick all and continue.
* Now, select KDC Type as `MIT KDC`, add 3 encryption types and remove the current one. the three encryption types are given below, add them:

       aes256-cts-hmac-sha1-96
       aes128-cts-hmac-sha1-96
       arcfour-hmac-md5

Kerberos Security Name is `HADOOPSECURITY.COM` and KDC Server host, KDC Admin Server host is `node2` (Node in which the kerberos server is installed).

* Now in the next step, untick to manage the `krb5.conf` file.

* enter the credentials, `cm/admin` and password will be `123`.

* Now the in the rest of the setup, keep the defaults and continue.

</details>





