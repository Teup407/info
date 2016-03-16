check memory
free –m
Reset Tripwire
tripwire --check --interactive

update & upgrade 
apt-get update && apt-get upgrade -y

Change Passwords
passwd {username}
DEFINER
sed -i 's/DEFINER=[^+]*\*/\*/g' dump.sql
TAR The contents of a DIR
tar -czvf public_html.tar.gz -C public_html .

Pulling Git to server
git clone -b my-branch git@github.com:user/myproject.git .

Rsync
rsync -ar --progress -e 'ssh -i /root/.ssh/charlotte_rsa' . charlottervcente@50.116.77.36:/home/charlottervcente/subdomains/staging/public_html/wp-content/plugins/Inventory_add_edit/images
301’s from XML - grep -Po 'http(s?)://[^ \"()\<>]*' sitemap.xml
Permissions
Wordpress Specific
chown -R www-data:www-data wp-content/uploads/
chown -R www-data:www-data wp-content/gallery/

Others
Create User
adduser demoadd
Add User To Group
usermod -a -G groupName userName

New Home Directory
usermod  -d new_home_dir  username


Permission Stuff 
chown –R username:www-data {site root dir}
find /var/www/html -type d -exec chmod 755 {} \;
find /var/www/html  -type f -exec chmod 644 {} \;	
Sharing WRITE permissions with two users

chgrp -R www-data /var/www/html/sitename/public_html
find /var/www/html/sitename/public_html -type d -exec chmod g=rwxs "{}" \;
find /var/www/html/sitename/public_html -type f -exec chmod g=rws "{}" \;





allow htaccess to work in 14.04 

nano /etc/apache2/apache2.conf

change to match below

<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
</Directory>



Lock Down Root SSH Access To Key Only

edit the server's SSHd configuration /etc/ssh/sshd_config and update the following line to now read:

PermitRootLogin without-password
# ps auxw | grep ssh
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root       681  0.0  0.1  49948  2332 ?        Ss    2012   3:23 /usr/sbin/sshd -D
# kill -HUP 681
SCP
Generally, if you want to download, it will go:
# download: remote -> local
scp user@remote_host:remote_file local_file
where local_file might actually be a directory to put the file you're copying in. To upload, it's the opposite:
# upload: local -> remote
scp local_file user@remote_host:remote_file
If you want to copy a whole directory, you will need -r. Think of scp as like cp, except you can specify a file with user@remote_host:file as well as just local files.

SSH CONFIG FOR SFTP and Jails at EOF
Nano /etc/ssh/sshd_config
Subsystem     sftp   internal-sftp
Match Group sftp
    ChrootDirectory %h
    ForceCommand internal-sftp
    AllowTcpForwarding no
    PasswordAuthentication yes


usermod -G sftp username
# usermod -s /bin/false username
# chown root:root /var/www/vhst
# chmod 0755 /home/vhost


TAR GZ The contents of a DIR
tar -czvf public_html.tar.gz -C public_html .

Create Virtual Host -> Ubuntu 14.04
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/”NameOfSite.com.conf
a2ensite whatever.com.conf
service apache2 reload
<VirtualHost *:80>
    ServerAdmin admin@yoursite.com
    ServerName yoursite.com
    ServerAlias www.yoursite.com
    DocumentRoot /var/www/html	
    <Directory /var/www/html/>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

Set Up FTP
apt-get install vsftpd
edit /etc/vsftpd.conf:
write_enable=YES
Now restart vsftpd:
service restart vsftpd
Securing FTP
There are options in /etc/vsftpd.conf to help make vsftpd more secure. For example users can be limited to their home directories by uncommenting:
chroot_local_user=YES
You can also limit a specific list of users to just their home directories:
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd.chroot_list
After uncommenting the above options, create a /etc/vsftpd.chroot_list containing a list of users one per line. Then restart vsftpd:
service restart vsftpd
Create User
adduser demoadd
Add User To Group
usermod -a -G groupName userName

New Home Directory
usermod  -d new_home_dir  username

Change Password
passwd USERNAME




.htaccess to make some sites work

;
 
Force non www to use www:
RewriteEngine on
RewriteCond %{HTTP_HOST} ^example.com [NC]
RewriteRule ^(.*)$ http://www.example.com/$1 [L,R=301,NC]

Trouble shooting 
At the top of the file causing the error add this:  <?php  ini_set("display_errors",1); ?>
Log files location: nano /var/log/apache2/error.log or access.log

Mysql command line – Basic Set Up
Log into MySQL using the administrator account you configured during the MySQL installation:
mysql -u root -p
You will be prompted for the MySQL root password and then you will be dropped into a MySQL prompt.
Create the two databases with the following commands:
@ FirstDatabase;
-CREATE DATABASE SecondDatabase;
Create a user t77hat will be associated with each database:
CREATE USER SecondUser@localhost;
Next, set up the password access for each account:
SET PASSWORD FOR FirstUser@localhost= PASSWORD("FirstPassword");
SET PASSWORD FOR SecondUser@localhost= PASSWORD("SecondPassword");
Finish up by granting privileges to the new users. This associates the database users with their respective databases and grants them appropriate permissions:
GRANT ALL PRIVILEGES ON FirstDatabase.* TO FirstUser@localhost IDENTIFIED BY 'FirstPassword';
GRANT ALL PRIVILEGES ON SecondDatabase.* TO SecondUser@localhost IDENTIFIED BY 'SecondPassword';
Refresh MySQL's privilege information to implement the changes:
FLUSH PRIVILEGES;
Exit out of MySQL to return to the shell session:
exit
Mysql dump / import
Dump Database
mysqldump -u USER -p DATABASE_NAME > export_filename.sql 
Import Database
mysql -u USER -p DATABASE_NAME < Name_of_mysql_file.sql 

Reset Mysql Root PW
/etc/init.d/mysql stop
sudo mysqld_safe --skip-grant-tables &
mysql -u root mysql
update user set password=PASSWORD("newpassword") where User='root';
FLUSH PRIVILEGES;

Other beginner stuff
https://www.digitalocean.com/community/tutorials/an-introduction-to-the-linux-terminal

Mysql error 111 
ufw enable 3306/tcp 
then
What IP address is MySQL listening on?
sudo netstat -plutn | grep 3306
If 127.0.0.1, then you will have to configure it to listen on all interfaces instead (127.0.0.1 and your droplet's IP address). Edit /etc/mysql/my.cnfand set bind-address to 0.0.0.0:
bind-address = 0.0.0.0
Then, restart MySQL:
sudo service mysql restart



UFW Logs
cat /var/log/messages | grep UFW
SSH on non standard port
Open /etc/ssh/sshd_config file and look for line Port 22 and change line to Port 2222. Restart sshd server.

vagrant init preconfigured_vm /path/to/mybox.box

go to root and run this command for diskspace – then work your way up
du . --max-depth=1 –h
log files - tail logfilename.log

find stuff find /var/log -type f -name "*.gz" –delete

Sphinx indexer - indexer --config /www/sites/rdev.watsons.com/files/html/var/aw_advancedsearch/searchd.conf

Git Tag
git tag -a v1.0.1 -m "Message describing reason for new version/tag"
        git push origin v1.0.1
        Create release on github off of tag
        On prod server
            <switch to website owner/user>
            git fetch --all
            git checkout -b v1.0.1 v1.0.1

Biadu Spider Issues :
iptables -A INPUT -s 119.63.193.0/24 -j reject
iptables -A INPUT -s 180.76.0.0/20 -j DROP
iptables -A INPUT -s 180.76.2.0/24 -j DROP
iptables -A INPUT -s 180.76.3.0/24 -j DROP
iptables -A INPUT -s 180.76.5.0/24 -j DROP
iptables -A INPUT -s 180.76.6.0/24 -j DROP
iptables -A INPUT -s 180.76.8.0/24 -j DROP
iptables -A INPUT -s 180.76.9.0/24 -j DROP
iptables -A INPUT -s 180.76.11.0/24 -j DROP
iptables -A INPUT -s 180.76.12.0/24 -j DROP
.iptables -A INPUT -s 185.10.104.0/24 -j DROP
iptables -A INPUT -s 185.10.105.0/24 -j DROP
iptables -A INPUT -s 203.90.238.0/24 -j DROP
iptables -A INPUT -s 180.76.15.0/24 -j DROP
