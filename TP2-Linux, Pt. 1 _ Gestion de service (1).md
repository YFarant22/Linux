# TP2-Linux, Pt. 1 : Gestion de service
## I. Un premier serveur web
1. Installation
- Installer le serveur Apache
```
[Y@web ~]$ sudo dnf install httpd
[...]
[Y@web ~]$ sudo ls /etc/httpd/
conf  conf.d  conf.modules.d  logs  modules  run  state
```
- Démarrer les services apache
```
[Y@web ~]$ sudo ss -alnpt
State      Recv-Q     Send-Q         Local Address:Port         Peer Address:Port    Process
LISTEN     0          128                  0.0.0.0:22                0.0.0.0:*        users:(("sshd",pid=858,fd=5))
LISTEN     0          128                     [::]:22                   [::]:*        users:(("sshd",pid=858,fd=7))
LISTEN     0          128                        *:80                      *:*        users:(("httpd",pid=2149,fd=4),("httpd",pid=2148,fd=4),("httpd",pid=2147,fd=4),("httpd",pid=2145,fd=4))
[Y@web ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
success
```
- Tests
```
[Y@web ~]$ sudo service httpd status
Redirecting to /bin/systemctl status httpd.service
● httpd.service - The Apache HTTP Server
...
```
-> vérifier que le service est démarré
```
[...]
   Active: active (running) since Wed 2021-09-29 11:31:01 EDT; 20min ago
[...]
```

-> vérifier qu'il est configuré pour démarrer automatiquement
```
[...]
 Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
[...]
```

-> test avec la commande curl
```
[Y@web ~]$ curl localhost
<!doctype html>
<html>
  <head>
[...]
```

2. Avancer vers la maîtrise du service

-> Activation du lancement automatique d'apache et ouverture du fichier httpd.service
```
[Y@web ~]$ sudo systemctl enable httpd
[Y@web ~]$ sudo systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: d>
[...]
```

-> Déterminer sous quel utilisateur tourne le processus Apache

```
[Y@web ~]$ cat /etc/httpd/conf/httpd.conf | grep User
User apache
```

```
[Y@web ~]$ ps -ef | grep apache
apache       880     849  0 08:38 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache       881     849  0 08:38 ?        00:00:02 /usr/sbin/httpd -DFOREGROUND
apache       882     849  0 08:38 ?        00:00:02 /usr/sbin/httpd -DFOREGROUND
apache       883     849  0 08:38 ?        00:00:02 /usr/sbin/httpd -DFOREGROUND
```

-> Changer l'utilisateur utilisé par Apache
```
[Y@web ~]$ sudo useradd Yapache -m -s /sbin/nologin -u 49
```

```
[Y@web ~]$ sudo systemctl try-restart httpd.service
[Y@web ~]$ ps -ef | grep httpd
root        3616       1  0 10:42 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
Yapache     3618    3616  0 10:42 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
Yapache     3619    3616  0 10:42 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
Yapache     3620    3616  0 10:42 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
Yapache     3621    3616  0 10:42 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
Y           3834    1691  0 10:42 pts/0    00:00:00 grep --color=auto httpd
```

->  Faites en sorte que Apache tourne sur un autre port
```
[Y@web ~]$ sudo firewall-cmd --add-port=2121/tcp --permanent
success
[Y@web ~]$ sudo firewall-cmd --remove-port=80/tcp --permanent
success
[Y@web ~]$ sudo firewall-cmd --reload
success
[Y@web ~]$ sudo firewall-cmd --list-all | grep ports
  ports: 2121/tcp
[Y@web ~]$ sudo systemctl try-restart httpd.service

```

```
[sudo] password for Y:
State     Recv-Q     Send-Q         Local [Address:Port         Peer Address:Port    Process
[...]
LISTEN    0          128                        *:2121                    *:*        users:(("httpd",pid=888,fd=4),("httpd",pid=887,fd=4),("httpd",pid=886,fd=4),("httpd",pid=856,fd=4))
```

-> Test avec la commande "curl" en local
```
[Y@web ~]$ curl localhost:2121
<!doctype html>
<html>
```


---
## II. Une stack web plus avancée
A. Serveur Web et NextCloud
```
[Y@web nextcloud]$ history
[...]
247  sudo dnf install epel-release
  248  dnf update
  249  sudo dnf update
  250  sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
  251  sudo dnf module list php
  252  sudo dnf module enable php:remi-7.4
  253  sudo dnf module list php
  254  sudo dnf install httpd mariadb-server vim wget zip unzip libxml2 openssl php74-php php74-php-ctype php74-php-curl php74-php-gd php74-php-iconv php74-php-json php74-php-libxml php74-php-mbstring php74-php-openssl php74-php-posix php74-php-session php74-php-xml php74-php-zip php74-php-zlib php74-php-pdo php74-php-mysqlnd php74-php-intl php74-php-bcmath php74-php-gmp
  255  sudo dnf install -y httpd php
  [...]
  257  sudo mkdir /etc/httpd/sites-availables
  258  sudo mkdir /etc/httpd/sites-available
  259  sudo mkdir /etc/httpd/sites-enabled
  [..]
  264  sudo mkdir /var/www/sub-domains
  265  sudo nano /etc/httpd/conf/httpd.conf
  266  sudo nano /etc/httpd/sites-available/com.yourdomain.nextcloud
  267  ln -s /etc/httpd/sites-available/com.yourdomain.nextcloud /etc/httpd/sites-enabled/
  268  sudo ln -s /etc/httpd/sites-available/com.yourdomain.nextcloud /etc/httpd/sites-enabled/
  269  mkdir -p /var/www/sub-domains/com.yourdomain.com/html
  270  sudo mkdir -p /var/www/sub-domains/com.yourdomain.com/html
  271  timedatectl
  [...]
  273  sudo wget https://download.nextcloud.com/server/releases/nextcloud-22.2.0.zip -y
  274  sudo wget https://download.nextcloud.com/server/releases/nextcloud-22.2.0.zip
  [...]
  439  sudo nano /etc/httpd/sites-enabled/linux.web.tp2
  440  sudo mv /etc/httpd/sites-enabled/linux.web.tp2 /etc/httpd/sites-available/
  441  sudo ln -s /etc/httpd/sites-available/linux.web.tp2 /etc/httpd/sites-enabled/
  442  sudo mkdir -p /var/www/sub-domains/linux.web.tp2/html
  443  ls /var/www/sub-domains/linux.web.tp2/
  444  unzip nextcloud-22.2.0.zip
  445  cd nextcloud
  446  cp -Rf * /var/www/sub-domains/linux.web.tp2/html/
  447  sudo cp -Rf * /var/www/sub-domains/linux.web.tp2/html/
  448  sudo chown -Rf apache.apache  /var/www/sub-domains/linux.web.tp2/html/
  449  sudo plain text systemctl restart httpd systemctl restart mariadb
  450  sudo systemctl restart httpd systemctl restart mariadb
  451  sudo systemctl restart httpd
  452  history
  453  sudo mv /etc/httpd/sites-available/linux.web.tp2  /etc/httpd/sites-available/web.tp2.linux
  454  sudo ln -s /etc/httpd/sites-available/web.tp2.linux /etc/httpd/sites-enabled/
  455  sudo rm /etc/httpd/sites-enabled/linux.web.tp2
  456  sudo nano /etc/httpd/sites-available/web.tp2.linux
  457  sudo mv /var/www/sub-domains/linux.web.tp2/ /var/www/sub-domains/web.tp2.linux
  458  sudo systemctl restart http
  459  sudo systemctl restart httpd
```

B. Base de données
-> Install de MariaDB sur db.tp2.linux
```
    3  sudo dnf install -y mariadb-server
    4  sudo systemctl enable mariadb
    5  sudo systemctl start mariadb
    6  mysql_sercure_installation
    7  mysql_secure_installation
    8  sudo systemctl restart mariadb
    9  sudo systemctl enable mariadb
   10  sudo systemctl restart mariadb
```
-> Préparation de la base pour NextCloud
```
[Y@db ~]$ sudo mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 9
Server version: 10.3.28-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE USER 'nextcloud'@'10.102.1.11' IDENTIFIED BY 'meow';
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLL
ATE utf8mb4_general_ci;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'10.102.1.11';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.001 sec)
```

-> Exploration de la base de données
```
[Y@web nextcloud]$ mysql -u nextcloud -h 10.102.1.12 -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 18
Server version: 10.3.28-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| nextcloud          |
+--------------------+
2 rows in set (0.002 sec)

MariaDB [(none)]> USE nextcloud
Database changed
MariaDB [nextcloud]> SHOW TABLES;
Empty set (0.001 sec)
```

```
MariaDB [(none)]> SELECT User, Host FROM mysql.user;
+-----------+-------------+
| User      | Host        |
+-----------+-------------+
| nextcloud | 10.102.1.11 |
| root      | 127.0.0.1   |
| root      | ::1         |
| root      | localhost   |
+-----------+-------------+
4 rows in set (0.000 sec)
```

C. Finaliser l'installation de NextCloud
```
MariaDB [mysql]> SHOW TABLES FROM nextcloud;
+-----------------------------+
| Tables_in_nextcloud         |
+-----------------------------+
| oc_accounts                 |
| oc_accounts_data            |
| oc_activity                 |
| oc_activity_mq              |
| oc_addressbookchanges       |
| oc_addressbooks             |
| oc_appconfig                |
| oc_authtoken                |
| oc_bruteforce_attempts      |
| oc_calendar_invitations     |
| oc_calendar_reminders       |
| oc_calendar_resources       |
| oc_calendar_resources_md    |
| oc_calendar_rooms           |
| oc_calendar_rooms_md        |
| oc_calendarchanges          |
| oc_calendarobjects          |
| oc_calendarobjects_props    |
| oc_calendars                |
| oc_calendarsubscriptions    |
| oc_cards                    |
| oc_cards_properties         |
| oc_circles_circle           |
| oc_circles_event            |
| oc_circles_member           |
| oc_circles_membership       |
| oc_circles_mount            |
| oc_circles_mountpoint       |
| oc_circles_remote           |
| oc_circles_share_lock       |
| oc_circles_token            |
| oc_collres_accesscache      |
| oc_collres_collections      |
| oc_collres_resources        |
| oc_comments                 |
| oc_comments_read_markers    |
| oc_dav_cal_proxy            |
| oc_dav_shares               |
| oc_direct_edit              |
| oc_directlink               |
| oc_federated_reshares       |
| oc_file_locks               |
| oc_filecache                |
| oc_filecache_extended       |
| oc_files_trash              |
| oc_flow_checks              |
| oc_flow_operations          |
| oc_flow_operations_scope    |
| oc_group_admin              |
| oc_group_user               |
| oc_groups                   |
| oc_jobs                     |
| oc_known_users              |
| oc_login_flow_v2            |
| oc_mail_accounts            |
| oc_mail_aliases             |
| oc_mail_attachments         |
| oc_mail_classifiers         |
| oc_mail_coll_addresses      |
| oc_mail_mailboxes           |
| oc_mail_message_tags        |
| oc_mail_messages            |
| oc_mail_provisionings       |
| oc_mail_recipients          |
| oc_mail_tags                |
| oc_mail_trusted_senders     |
| oc_migrations               |
| oc_mimetypes                |
| oc_mounts                   |
| oc_notifications            |
| oc_notifications_pushhash   |
| oc_oauth2_access_tokens     |
| oc_oauth2_clients           |
| oc_preferences              |
| oc_privacy_admins           |
| oc_properties               |
| oc_ratelimit_entries        |
| oc_recent_contact           |
| oc_richdocuments_assets     |
| oc_richdocuments_direct     |
| oc_richdocuments_wopi       |
| oc_schedulingobjects        |
| oc_share                    |
| oc_share_external           |
| oc_storages                 |
| oc_storages_credentials     |
| oc_systemtag                |
| oc_systemtag_group          |
| oc_systemtag_object_mapping |
| oc_talk_attendees           |
| oc_talk_bridges             |
| oc_talk_commands            |
| oc_talk_internalsignaling   |
| oc_talk_rooms               |
| oc_talk_sessions            |
| oc_text_documents           |
| oc_text_sessions            |
| oc_text_steps               |
| oc_trusted_servers          |
| oc_twofactor_backupcodes    |
| oc_twofactor_providers      |
| oc_user_status              |
| oc_user_transfer_owner      |
| oc_users                    |
| oc_vcategory                |
| oc_vcategory_to_object      |
| oc_webauthn                 |
| oc_whats_new                |
+-----------------------------+
108 rows in set (0.001 sec)
```

---

### <<< Tableau des machines >>>

|    Machine    |     IP      |         Service         | Port ouvert | IP autorisées |
|:-------------:|:-----------:|:-----------------------:|:-----------:|:-------------:|
| web.tp2.linux | 10.102.1.11 |       Serveur web       |    2121     |  /  |
| db.tp2.linux  | 10.102.1.12 | Serveur Base De Données |     22      |  10.102.1.11  |