# install_kaspersky_update_utility_3.0_on_Centos-8
how to setup kaspersky update utility 3.0 on Centos 8

# how to setup kaspersky update utility 3.0 on Centos 8

#turned off ssh root access
$ nano /etc/ssh/sshd_config

# find and change
PermitRootLogin no

# restart ssh

$ systemctl restart sshd

# install Apache
dnf install httpd

# run and add to startup
systemctl enable httpd
systemctl start httpd


# create folders

# Updates folder will be created later automatically
$ mkdir -p /var/www/html/kavupdater 

# Kaspersky update utility folder
$ mkdir -p /opt/kuu   

# logs
$ mkdir -p /var/www/html/log

# kladmin - user for updates
$ adduser kladmin

$ chown -R kladmin /var/www/html/kavupdater

$ chmod -R 755 /var/www

# make virtual host
$ mkdir /etc/httpd/sites-available /etc/httpd/sites-enabled

$ nano /etc/httpd/conf/httpd.conf -> IncludeOptional sites-enabled/*.conf  в конце

$ cd /etc/httpd/sites-available

# create and modify file
$ nano kavupdater.conf

<VirtualHost nabu.efbank.local:80>
ServerAdmin jabba@efbank.local
ServerName nabu.efbank.local
DocumentRoot /var/www/html/kavupdater
<Directory /var/www/html/kavupdater>
Options All Indexes FollowSymLinks
Require all granted
</Directory>
ErrorLog logs/nabu.efbank.local-error_log
CustomLog logs/nabu.efbank.local-access_log combined
</VirtualHost>

# create a symbolic link for virtual host
$ ln -s /etc/httpd/sites-available/kavupdater.conf /etc/httpd/sites-enabled/kavupdater.conf

# delete default startup page
$ rm /etc/httpd

# restart apache
$ systemctl restart httpd

# check URL
http://your_server_IP/


$ reboot

# there is one bug in this setup that I didn’t fix
# after rebooting the system, the default directory becomes /var/www/html/ - I don’t know why
# so that after reboot everything works, make a script and put it in autoload

$nano /usr/bin/apachereload.sh

#!/bin/bash
# cat /usr/bin/apachereload.sh
systemctl restart httpd


# save and make executable

$cmod u+x /usr/bin/apachereload.sh

# another file

$ nano /lib/systemd/system/apachereload.service

#  modify file

# apache reload
[Unit]
Description=Apache_reload
After=multi-user.target
[Service]
Type=idle
ExecStart=/usr/bin/apachereload.sh
[Install]
WantedBy=multi-user.target


# change access

$ chmod 644 /lib/systemd/system/apachereload.service

# reload:

$ systemctl daemon-reload

# put it in autoload:

$ systemctl enable apachereload.service

# you can reboot and make sure that the apach service restarts and we get to the correct directory


# Setup kaspersky update utility

$cd /opt/kuu

# download the distribution (at the time of writing the article was relevant kuu3.2.0.153_x86_64_en.tar.gz)

$ wget https://products.s.kaspersky-labs.com/special/kasp_updater3.0/3.2.0.153/english-20181221.11.081/5f207288/kuu3.2.0.153_x86_64_en.tar.gz

# unpack
tar xvzf kuu3.2.0.153_x86_64_en.tar.gz

# modify configuration file
nano updater.ini


[ComponentSettings]
# add below
KasperskyEndpointSecurityForWinWKS_11=true
KasperskyEndpointSecurityForWinWKS_11_256=true

[DirectoriesSettings]
ClearTempFolder=true
MoveToCurrentFolder=false
MoveToCustomFolder=true
TempFolder=/var/www/html/kavupdater/tmp
UpdatesFolder=/var/www/html/kavupdater/Updates

# if you use PROXY modify that section

[ConnectionSettings]
TimeoutConnection=60
UsePassiveFtpMode=true
UseProxyServer=true
AutomaticallyDetectProxyServerSettings=false
UseSpecifiedProxyServerSettings=true
AddressProxyServer=192.168.1.22
PortProxyServer=80
UseAuthenticationProxyServer=false
UserNameProxyServer=
PasswordProxyServer=
ByPassProxyServer=true
DownloadPatches=true
RetranslateDiffs=true

# change owner
$ chown -R kladmin /opt/kuu 

# change user to kladmin

$ su kladmin

# run script:
# accept license /y

$ ./uu-console.sh -u

# check http://your_server_IP/Updates/ 
# you should see two folders Updates and tmp

# create crontab rule
# create empty file
$ touch /home/kladmin/kavupdlog.txt

# create and modify
$ nano /home/kladmin/kavupdater.sh

#!/bin/bash
cd /opt/kuu
./uu-console.sh -u >> /home/kladmin/kavupdlog.txt


# make executable
chmod u+x /home/kladmin/kavupdater.sh

# add task to cron:
$ crontab -e

# add line below - updates will be downloaded every 2 hours
0 */2 * * * /home/kladmin/kavupdater.sh







