Udacity Fullstack Developer Nanodegree Project 5
================================================


Connecting to the VM via SSH
----------------------------
SSH is running on port 2200 on the VM. Both root login and password auth have been disabled. You can login using the supplied udacity_key.rsa key file using the following command:

ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@52.36.31.156


URL of the deployed Catalog app
-------------------------------
The Catalog app has been deployed on this VM. It should be accessed using the following URL (instead of IP) so that 3rd party OAuth login can work properly:

http://ec2-52-36-31-156.us-west-2.compute.amazonaws.com/


Software Installed
------------------
The following extra software was installed on the base VM provided by AWS.

Ubuntu packages:
python
apache2
libapache2-mod-wsgi
postgresql

Python packages:
Pip
Psycopg2
Flask
SQLAlchemy
oauth2client
requests
httplib2


Configuration changes
---------------------
Following is a complete list of configuration changes that were performed on the VM in order to secure it and make it host the Catalog app properly:

1. Created a new user named 'grader' using the 'adduser' command.
2. Granted sudo permission to user 'grader' without requiring password prompt by adding the following line to '/etc/sudoers':
        grader    ALL=NOPASSWD: ALL
3. Allowed SSH access to user 'grader' with the 'udacity_key.rsa' key file by copying '/root/.ssh/authorized_keys' file to the directory '/home/grader/.ssh/'. Now we can SSH as 'grader' using the following command:
        ssh -i ~/.ssh/udacity_key.rsa grader@52.36.31.156
4. Disabled SSH access for 'root' user by adding the following line to '/etc/ssh/sshd_config':
        PermitRootLogin no
5. Updated all currently installed packages by running the following commands:
        sudo apt-get update
        sudo apt-get upgrade
6. Configured the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123) by running the following commands:
        sudo ufw default allow outgoing
        sudo ufw default deny incoming
        sudo ufw allow 2200
        sudo ufw allow 80
        sudo ufw allow 123
        sudo ufw enable
7. Configured the local timezone to UTC by running the following command:
        dpkg-reconfigure tzdata
8. Installed Apache web server by running the following command:
        sudo apt-get install apache2 
9. Installed mod_wsgi for Apache by running the following command:
        sudo apt-get install libapache2-mod-wsgi
10. Installed postgresql by running the following command:
        sudo apt-get install postgresql
11. Created new postgresql user 'catalog' by running the following command as the 'postgres' user:
        createuser --interactive catalog
    Only grant permission to create databases.
12. Set password for postgresql user 'catalog' by running the following command in the 'psql' console:
        ALTER USER catalog WITH PASSWORD '123';
13. Updated file '/etc/postgresql/9.3/main/pg_hba.conf':
    a. Ensured that remote connections are disallowed by adding the following line:
        host    all             all             127.0.0.1/32            md5
    b. Require password for local connections by adding the following line:
        local   all             all                                     md5
14. Created new postgresql database 'catalog' owned by the 'catalog' user by running the following command as the 'postgres' user:
        createdb -U catalog --locale=en_US.utf-8 -E utf-8 -O catalog catalog -T template0
15. Installed git by running the following command:
        sudo apt-get install git
16. Cloned the git repository for Catalog app project 3 into '/home/grader/' directory:
        git clone git@github.com:salman-awan/fullstack-nanodegree-vm.git
17. Created '/var/www/CatalogApp' directory and copy '/home/grader/fullstack-nanodegree-vm/vagrant/catalog' directory to it.
18. Changed all references to SQLLite database in the Catalog app python code to:
        "postgresql://catalog:123@localhost/catalog"
19. Installed PIP:
        sudo apt-get install python-pip
20. Installed required Python packages:
        sudo pip install -r /var/www/CatalogApp/CatalogApp/requirements.txt
        sudo apt-get install python-psycopg2
21. Ran 'populate_db.py' from directory '/var/www/CatalogApp/CatalogApp' to populate data in the catalog database.
22. Renameed 'application.py' to '__init__.py' in '/var/www/CatalogApp/CatalogApp' directory.
23. Created file '/etc/apache2/sites-available/CatalogApp.conf':

        <VirtualHost *:80>
                #ServerName
                ServerAdmin salman.awan@gmail.com
                WSGIScriptAlias / /var/www/CatalogApp/catalogapp.wsgi
                <Directory /var/www/CatalogApp/CatalogApp/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/CatalogApp/CatalogApp/static
                <Directory /var/www/CatalogApp/CatalogApp/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>

24. Created file '/var/www/CatalogApp/catalogapp.wsgi':

        #!/usr/bin/python
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/CatalogApp/")

        from CatalogApp import app as application
        from CatalogApp import generate_csrf_token

        application.secret_key = 'super_secret_key'
        application.jinja_env.globals['csrf_token'] = generate_csrf_token

25. Enabled Catalog app site:
        sudo a2dissite 000-default
        sudo a2ensite CatalogApp
26. Restarted apache service: 
        sudo service apache2 restart
27. Changed path for 'client_secrets.json':
        client_secrets = os.path.join(os.path.dirname(__file__), 'client_secrets.json')
28. Updated OAuth redirect URIs for Catalog app to add the following URL:
        http://ec2-52-36-31-156.us-west-2.compute.amazonaws.com/oauth2callback
