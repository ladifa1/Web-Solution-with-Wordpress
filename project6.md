# WEB SOLUTION WITH WORDPRESS

## WEB SERVER CREATION

Launch an EC2 instance that will serve as a webserver for the wordpress with 3 volumes attached

Verify volumes are added to the webserver 

`lsblk` 

![](images/1.png)

Create a single partiton on each of the 3 disks 

`sudo gdisk /dev/xvdf`

`sudo gdisk /dev/xvdg`

`sudo gdisk /dev/xvdh`

![](images/2.png)

![](images/3.png)

Use lsblk to verify the partitions 

![](images/4.png)

Install lvm2 packages

`sudo yum install lvm2`

![](images/5.png)

Check for available partions 
`sudo lvmdiskscan` 

![](images/6.png)

Mark the 3 partioned disks as physical volumes

`sudo pvcreate /dev/xvdf1`

`sudo pvcreate /dev/xvdg1`

`sudo pvcreate /dev/xvdh1`

![](images/7.png)

Verify that physical volumes are created

`sudo pvs`

![](images/8.png)

Add the physical volumes to a volume group

`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

![](images/9.png)

Verify that volume group has been created 

`sudo vgs`

![](images/10.png)

Create 2 logical volumes, one to store data for the website and another to store data for logs

`sudo lvcreate -n apps-lv -L 14G webdata-vg`

`sudo lvcreate -n logs-lv -L 14G webdata-vg`

![](images/11.png)

Verify the logical volumes were created

`sudo lvs`

![](images/12.png)


Verify the entire setup

`sudo vgdisplay -v`

![](images/13.png)

Format the logical volumes with ext4 filesystem

`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`

`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

![](images/14.png)

Create /var/www/html directory to store website data

`sudo mkdir -p /var/www/html`

Create /home/recovery/logs directory to store backup log data

`sudo mkdir -p /home/recovery/logs`

Mount /var/www/html on apps-lv logical volume 

`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`
 

Backup all files in log directory /var/log into /home/recovery/logs because all existing data in /var/log would be deleted when mounted

`sudo rysnc -av /var/log/. /home/ecovery/logs/

 ![](images/15.png)

Mount /var/log on logs-lv logical volume 

`sudo mount /dev/webdata-vg/logs-lv /var/log`

Restore log files 

`sudo rsync -av /home/recovery/logs/. /var/log`

Get UUID of devices that will be updated in /etc/fstab file

`sudo blkid`

![](images/16.png)

Update /etc/fstab file

`sudo vi /etc/fstab/`

![](images/17.png)

Test the configuration and reload daemon 

Verify setup with df -h

`sudo mount -a`

`sudo systemctl daemon-reload`


![](images/18.png)

Update redhat

`sudo yum -y update`

![](images/25.png)

Install php and verify

`php -v`

![](images/26.png)

![](images/27.png)

Start and enable php 

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

![](images/28.png)

Check status of apache 

`sudo systemctl status httpd`

![](images/29.png)

Create directory wordpress

`sudo mkdir wordpress`

In wordpress directory download wordpress

`sudo wget http://wordpress.org/latest.tar.gz`

![](images/30.png)
Extract the tar file

`sudo tar xzvf latest.tar.gz`



Copy wordpress to /var/www/html

`sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php`

`sudo cp -R wordpress /var/www/html/`

![](images/31.png)

Configure SElinux Polices 

`sudo chown -R apache:apache /var/www/html/wordpress`

`sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R`

`sudo setsebool -P httpd_can_network_connect=1`

![](images/32.png)

## DATABASE SERVER CREATION

Followed same process as the creation of webserver 

On database server created a volume group named vg-database

`sudo vgcreate vg-database /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

verify volume group was sucessful

![](images/19.png)

Create and verify logical volume 

`sudo lvcreate -n db-lv -L 24G vg-databas`

![](images/20.png)

Format the logical volumes with ext4 filesystem

` sudo mkfs.ext4 /dev/vg-database/db-lv`

![](images/21.png)

Create a directory /db

`sudo mkdir /db`

Mount db-lv to /db

`sudo mount /dev/vg-database/db-lv /db`

Check for UUID for the device

`sudo blkid`

![](images/22.png)

Update /etc/fstab/

`sudo vi /etc/fstab`

![](images/23.png)

Test the configuration and reload daemon 

`sudo mount -a`

`sudo systemctl daemon-reload`

Check setup 

`df -h`

![](images/24.png)

Update redhat 

`sudo yum update`

Install mysql server

![](images/33.png)

![](images/34.png)

Configure database

`sudo mysql`

![](images/35.png)

Verify database

`mysql> SHOW DATABASES;`

![](images/36.png)

## connecting database to webserver

update redhat webserver

`sudo yum update`

Install mysql on webserver

`sudo yum install mysql`

![](images/37.png)

Test the connection between webserver and database server

`sudo mysql -u ladi -h 172.31.42.175 -p`

`mysql> SHOW DATABASES;`

![](images/38.png)

Update wp-config file in webserver with database details

`sudo vi wp_config.php`

![](images/wpconfig.png)

Open port 80 on ec2 security groups 

Configure wordpress 

![](images/39.png)

![](images/40.png)

![](images/41.png)

![](images/42.png)
