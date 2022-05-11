## DEVOPS TOOLING WEBSITE SOLUTION

**NFS SERVER**

`sudo lsblk`

![LSBLK.PNG](./images/NFS%20LSBLK.jpg)

`sudo gdisk /dev/xvdf`

`sudo gdisk /dev/xvdg`

`sudo gdisk /dev/xvdh`

![LSBLK.GDISK.PNG](./images/NFS%20LSBLK%20AFTER%20GDISK.jpg)

`sudo yum install lvm2`

`sudo lvmdiskscan`

![LVMSCANDISK.PNG](./images/NFS%20LVMSCANDISK.jpg)

`sudo pvcreate /dev/xvdf1`

`sudo pvcreate /dev/xvdg1`

`sudo pvcreate /dev/xvdh1`

`sudo pvs`

`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

`sudo vgs`

![LVMSCANDISVGCREATE.PNG](./images/NFS%20VG%20CREATE.jpg)

`sudo lvcreate -n lv-apps  -L 9G webdata-vg`

`sudo lvcreate -n  lv-logs  -L 14G webdata-vg`

`sudo lvcreate -n  lv-opt  -L 14G webdata-vg`

`sudo lvs`

![LVSCREATE.PNG](./images/NFS%20LVS%20CREATE.jpg)

`sudo lsblk`

![WHOLESETUP.PNG](./images/NFS%20LSBLK%20WHOLE%20CHECK%20UP.jpg)

`sudo mkfs -t xfs /dev/webdata-vg/lv-apps`

`sudo mkfs -t xfs /dev/webdata-vg/lv-logs`

`sudo mkfs -t xfs /dev/webdata-vg/lv-opt`

`sudo mkdir /mnt/apps`

`sudo mkdir /mnt/logs`

`sudo mkdir /mnt/opt`

`sudo mount /dev/webdata-vg/lv-apps /mnt/apps`

`sudo mount /dev/webdata-vg/lv-logs /mnt/logs`

`sudo mount /dev/webdata-vg/lv-opt /mnt/opt`

![WHOLESETUP.PNG](./images/NFS%20WHOLE%20SET%20UP.jpg)

`sudo yum -y update`

`sudo yum install nfs-utils -y`

`sudo systemctl start nfs-server.service`

`sudo systemctl enable nfs-server.service`

`sudo systemctl status nfs-server.service`

`sudo chown -R nobody: /mnt/apps`

`sudo chown -R nobody: /mnt/logs`

`sudo chown -R nobody: /mnt/opt`

`sudo chmod -R 777 /mnt/apps`

`sudo chmod -R 777 /mnt/logs`

`sudo chmod -R 777 /mnt/opt`

`sudo systemctl restart nfs-server.service`

`sudo vi /etc/exports

/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

Esc + :wq!

sudo exportfs -arv`

`rpcinfo -p | grep nfs`

![PORT.PNG](./images/NFS%20PORT.jpg)

**DATABASE**
`sudo apt update`

`sudo apt install mysql-server -y`

`sudo mysql`

`create database tooling;`

`create user 'webaccess'@'webserver CIDR' idenfified by 'password';`

`grant all privileges on tooling.* to 'webaccess'@'webserver CIDR';`

`flush privileges;`

`show databases;`

`use tooling`

`sudo systemctl status mysql`

`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`

`set bind-address = 0.0.0.0`
`set mysqlx-bind-address = 0.0.0.0`

`sudo systemctl restart mysql`
`sudo systemctl status mysql`

`select * from users;`


![ADMIN.PNG](./images/DB%20ADMIN%20MY%20SQL.jpg)


**WESERVER**

`sudo yum install nfs-utils nfs4-acl-tools -y`

`sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www`

`df -h`

![PORT.PNG](./images/WEB1%20SERVER%20MOUNTED%20DF-H.jpg)

`sudo vi /etc/fstab`

`<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0`

`sudo yum install httpd -y`

`sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

`sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

`sudo dnf module reset php`

`sudo dnf module enable php:remi-7.4`

`sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

`setsebool -P httpd_execmem 1`

`sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/logs /var/log/httpd`

`sudo vi /etc/fstab`

`<NFS-Server-Private-IP-Address>:/mnt/logs /var/log/httpd nfs defaults 0 0`

`sudo yum install git -y`

`git init`

`git clone https://github.com/darey-io/tooling.git`

`sudo cp -R html/. /var/www/html`

`sudo setenforce 0`

`sudo vi /etc/sysconfig/selinux`

`SELINUX=disabled`

`sudo vi  /var/www/html/functions.php`

`cd tooling`

`mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql`

`http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php`

![PORT.PNG](./images/WBS1%20LOGIN%20PAGE.jpg)

![PORT.PNG](./images/TOOLING%20WEBPAGE.jpg)

The same procedure was followed for the other two webservers.




