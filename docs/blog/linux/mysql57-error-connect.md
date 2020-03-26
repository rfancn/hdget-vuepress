---
date: '2017-04-13 10:21:07'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- linux
- mysql
title: MySQL 5.7 ERROR 1045 (28000) Access denied
---

The reason is MySQL v 5.7 or higher generates a temporary random password after installation and stored that in mysql error log file, located at /var/log/mysqld.log for an installation by the MySQL Yum repository on CentOS 7. It doesn't use empty root password for mysql as previous version does, how to revert to old fashion, pls refer to below:

1. Check what's random password it generated when installing:

        # sudo grep 'temporary password' /var/log/mysqld.log

2. Login to mysql server as root 

        # mysql -u root -p

3. Change mysql root user's password for the first time

        mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '{new password}';

    > Note: Pls replace {new password} with below rules, e,g: 'Goodpwd@1'
        - at least 8 characters
        - at least contains one Uppercase character
        - at least contains one digit
        - at least contains one special character

4. Run following sql command to remove validate_password plugin

        mysql> uninstall plugin validate_password;

5. set root password to the empty one

        mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '';