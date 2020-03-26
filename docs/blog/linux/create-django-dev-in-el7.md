---
date: '2017-04-13 10:21:07'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- linux
- django
title: Create Django1.8LTS deployment env on CentOS7 from Scratch
---

Here we will create a cloud based virtual server from scratch, following things need to be done to prepare develop and deployment env for django1.7+.

#### Background

Personally I prefer to setup deployment env before developing any project, put difficulties ahead may be a good thing. 
As python2.6 is not officially supported and django1.6 also end of the support life when I wrote this document, I choose to use python2.7 + latest django1.8.
As CentOS6 only has python2.6 shipped, other application like yum depend on python2.6 to work properly, of course, we can install python2.7 manually but it may broke other existing system applications, so I choose CentOS7(more specifically CentOS7.1)  as the deployment OS, which has python2.7 shipped by default, anyway still something need you pay attention to:

 * mysql-server package is not shipped by default in CentOS7
 * service start method is totally different with CentOS6

#### WSGI Server Choosing

Though there are many wsgi servers could be deployed for Django, and 3 combination way has been introduced in Django official site, I just choose the most popular one as our targets, the first combination is easy to deploy, and the second one could obtain much high performance and consume much less system resources(e,g: memory).

 * PlanA:  Apache + mod_wsgi
 * PlanB:  uWSGI + nginx

#### Security

 * Create normal user

        # useradd test
        # passwd test

 * Make normal user to suders

        # visudo

        uncomment below line
        ---------------------------
        %wheel  ALL=(ALL)       ALL

 * Add normal user to wheel group

        # usermod -a -G wheel [username]

 * security harden on sshd service, change below on /etc/ssh/sshd_config
   
        port [specific port]
        PermitRootLogin no
        MaxAuthTries 3
        X11Forwarding no
    
#### Installing Packages

 * upgrade system to the latest version

        $ sudo yum update

 * python-pip

        $ sudo yum install python-pip
        $ sudo pip install --upgrade pip

 * mysql-server 

        $ sudo rpm -ivh http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
        $ sudo yum install mysql-server
        $ sudo systemctl enable mysqld
        $ sudo systemctl start mysqld

 * mysql python library

        $ sudo yum install MySQL-python

 * virtualenv related packages

        $ sudo pip install virtualenv virtualenvwrapper
        $ mkdir ~/.virtualenv

        Add following into ~/.bashrc file
        ---------------------------------
        export PIP_REQUIRE_VIRTUALENV=true
        export WORKON_HOME=$HOME/.virtualenvs
        source /usr/bin/virtualenvwrapper.sh

 * Create test virtualenv for Django

        $ source ~/.bashrc
        $ mkvirtualenv test
        $ lsvirtualenv
        $ pip install "django==1.8.15"


#### PlanA(Apache + mod_wsgi)

        # sudo yum install httpd
        # sudo yum install mod_wsgi

#### PlanA Configuration

 * Assume you want deploy django web env for www.example.com
 * Create a dir names "example" under "/var/www/html" directory
 * Create a static dir "static" under "/var/www/html/example/", of course, it can be anywhere you like
 * Create a configuration file e,g: example.conf file under /etc/httpd/conf.d as following:

        LoadModule wsgi_module modules/mod_wsgi.so

        WSGISocketPrefix /var/run/wsgi
        <virtualhost *:80>
            ServerName www.example.com
            DocumentRoot /var/www/html/example
            ErrorLog logs/www.example.com-error_log
            CustomLog logs/www.example.com-access_log common
            Alias /static/ /var/www/html/example/static/
            <directory /var/www/html/example/static>
                Order deny,allow
                Allow from all
            </directory>

        WSGIDaemonProcess example python-path=/var/www/html/example processes=1 threads=10 display-name=%{GROUP}
        WSGIProcessGroup example
        WSGIScriptAlias / /var/www/html/example/wsgi.py
        <directory /var/www/html/example>
            <files wsgi.py>
                Order deny,allow
                Allow from all
            </files>
        </directory>
        </virtualhost>

#### PlanB(Nginx + uWSGI)

    # sudo yum install nginx
    # sudo yum install uwsgi uwsgi-plugin-python

#### PlanB Configuration
 * Assume a virtualenv named 'test' already created, and django1.8 installed there
 * Start project named 'proj' under current user's home directory, e,g: /home/ryan/

    1. Create a vassal common default config file

            (test) [ryan@localhost ~]$  sudo vi /etc/uwsgi-vassals-default.ini
            ------------------------------------------------------------------
            [uwsgi]
            master = true
            processes = 2
            threads = 2
            socket = /run/uwsgi/%(vassal_name).sock
            # clear environment on exit, try to remove all of the generated file/sockets
            vacuum = true
            plugins = python
        
   2. Create a specific vassal config file under django project dir

            (test) [ryan@localhost ~]$ django-admin.py startproject proj
            (test) [ryan@localhost ~]$ cd ~/proj
            (test) [ryan@localhost ~]$ vi proj-uwsgi.ini
            ---------------------------------------------------
            [uwsgi]
            vassal_name = proj
            home_dir = /home/ryan

            chdir = %(home_dir)/%(vassal_name)
            home= %(home_dir)/.virtualenvs/test              < ---- pls use the actual virtualenv dir you created, this is PYTHON_HOME 
            module=%(vassal_name).wsgi:application           <---- pls use the real module path of wsgi.py file
            chmod-socket=660

    3. Link the specific vassal config file in CentOS7.2 default /etc/uwsgi.d/ dir

            (test) [ryan@localhost ~]$ sudo ln -s proj-uwsgi.ini /etc/uwsgi.d/

    4. Change user/group of proj-uwsgi.ini to uwsgi:nginx for "emperor-tyrant = true" is the default setting in /etc/uwsgi.ini

            (test) [ryan@localhost ~]$ sudo chown uwsgi:nginx proj-uwsgi.ini

        > Note: When 'emperor-tyrant = true' is set, the vassal process's uid/gid will be set to the same as vassal config file, here it means file itself, not the uid/gid value in file content.

    5. Change default /etc/uwsgi.ini to make it looks like below

            (test) [ryan@localhost ~]$  sudo vi /etc/uwsgi.ini
            --------------------------------------------------
            [uwsgi]
            uid = uwsgi
            gid = uwsgi
            pidfile2 = /run/uwsgi/uwsgi.pid
            emperor = /etc/uwsgi.d
            stats = /run/uwsgi/stats.sock
            emperor-tyrant = true
            cap = setgid,setuid
        
            # include config templates to vassals config
            vassals-include = /etc/uwsgi-vassals-default.ini
            # clear environment on exit, try to remove all of the generated file/sockets
            vacuum = true

    6. start uwsgi service and check it's status

            (test) [ryan@localhost ~]$ sudo systemctl start uwsgi
            (test) [ryan@localhost ~]$ sudo systemctl status uwsgi -l

 * Configure nginx to establish connection with local unix socket created by uwsgi

        (test) [ryan@localhost ~]$ vi /etc/nginx/conf.d/proj-nginx.conf
        ----------------------------------------------------
        server {
            listen       80;
            server_name  www.test.com;

            location = favicon.ico { access_log off; log_not_found off; }

            location /static/ {
                alias /home/ryan/proj/staticfiles/;
            }

            location / {
                include uwsgi_params;
                uwsgi_pass unix:/var/run/uwsgi/proj.sock;
            }
        }
        (test) [ryan@localhost ~]$ usermod -aG ryan nginx
        (test) [ryan@localhost ~]$ namei -om /path/to/project
        ------------------------------------------------------
        make sure all parent dir of /path/to/project all have execute permission
        (test) [ryan@localhost ~]$ chmod 711 /home/ryan
 
#### FAQ
- Why we still get HTTP 502 error even follow above steps to configure nginx and uwsgi, detail error message recorded in nginx log file as below:
		
		connect() to unix:/var/run/uwsgi/xxx.sock failed (13: Permission denied) while connecting to upstream, client: 192.168.1.1, server: www.test.com, request: "GET / HTTP/1.1", upstream: "uwsgi://unix:/var/run/uwsgi/xxx.sock:", host: "www.test.com", referrer: "http://www.test.com/"

    ANSWER: The reason may be SELinux not having the policy for nginx to write to sockets. You may need completely disable selinux or add the enforcement policy for nginx.


#### Reference

* [How To Serve Django Applications with uWSGI and Nginx on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-serve-django-applications-with-uwsgi-and-nginx-on-centos-7)