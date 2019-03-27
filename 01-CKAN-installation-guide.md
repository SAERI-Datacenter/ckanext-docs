# Installing CKAN from source

> Documentation for Ubuntu 16.04

## Summary

### Ports

    80    = nginx  
    8080  = apache2  
    8800  = datapusher  
    8983  = jetty/tomcat for solr

### Passwords

    postgres: user = ckan_default, password = saerickan
    postgres: user = datastore_default, password = saeridatastore
    ckan: sysadmin = ilaria, password = saeriadmin

### Config files

    /etc/ckan/default/production.ini

### Virtualenv

    cd /usr/lib/ckan/default/src/ckan
    . /usr/lib/ckan/default/bin/activate

### Paster

    /usr/lib/ckan/default/bin/paster --plugin=ckan jobs list --config=/etc/ckan/default/production.ini


## 1. Install the required packages

    sudo apt-get install python-dev postgresql libpq-dev python-pip python-virtualenv git-core solr-jetty openjdk-8-jdk redis-server

## 2. Install CKAN into a Python virtual environment

>### Optional
If you’re installing CKAN for development and want it to be installed in your home directory, you can symlink the directories used in this documentation to your home directory. This way, you can copy-paste the example commands from this documentation without having to modify them, and still have CKAN installed in your home directory:  
>>  mkdir -p ~/ckan/lib  
    sudo ln -s ~/ckan/lib /usr/lib/ckan  
    mkdir -p ~/ckan/etc  
    sudo ln -s ~/ckan/etc /etc/ckan  

#### Create a Python virtual environment (virtualenv) to install CKAN into, and activate it

    sudo mkdir -p /usr/lib/ckan/default
    sudo chown `whoami` /usr/lib/ckan/default
    virtualenv --no-site-packages /usr/lib/ckan/default
    . /usr/lib/ckan/default/bin/activate

#### to check the ownership of the /usr/lib/ckan/default

    ls -l /usr/lib/ckan/default


**NOTE: from now onwards work always in the virtual env, hence make sure that**

    . /usr/lib/ckan/default/bin/activate

#### has been run once ahead every command

#### Install the recommended setuptools version

#### check version of setuptools

    easy_install --version

> if less than 36.1 install as below, if greater than 36.1 then skip the step directly below**

    pip install setuptools==36.1

#### run

    pip install --ignore-installed setuptools

#### Install the CKAN source code into your virtualenv

    pip install -e 'git+https://github.com/ckan/ckan.git@ckan-2.8.2#egg=ckan'

#### Install the Python modules that CKAN requires into your virtualenv

    pip install -r /usr/lib/ckan/default/src/ckan/requirements.txt
    pip install -r /usr/lib/ckan/default/src/ckan/dev-requirements.txt

>**if the following error message appears we ignore it**  
httpretty 0.8.3 has requirement urllib3==1.7.1, but you'll have urllib3 1.23 which is incompatible.  
mox3 0.26.0 has requirement pbr!=2.1.0,>=2.0.0, but you'll have pbr 1.10.0 which is incompatible.


#### Deactivate and reactivate your virtualenv, to make sure you’re using the virtualenv’s copies of commands like paster rather than any system-wide installed copies

    deactivate
    . /usr/lib/ckan/default/bin/activate

## 3. Install Postgresql

    sudo apt-get update
    sudo apt-get install postgresql postgresql-contrib

### Setup a PostgreSQL database

#### Check that the encoding of databases is UTF8

    sudo -u postgres psql -l

#### to exit

    \q

#### Create a new PostgreSQL database user called ckan_default, and enter a password for the user when prompted

    sudo -u postgres createuser -S -D -R -P ckan_default

> user = ckan_default   
  password = saerickan

#### Create a new PostgreSQL database, called ckan_default, owned by the database user you just created

    sudo -u postgres createdb -O ckan_default ckan_default -E utf-8

So in postgres we have a database called `ckan_default` with a user called `ckan_default` and password `saerickan`

## 4. Create a CKAN config file

### Create a directory to contain the site’s config files

    sudo mkdir -p /etc/ckan/default
    sudo chown -R `whoami` /etc/ckan/

> **DO NOT RUN**    
  sudo chown -R `whoami` ~/ckan/etc

### Create the CKAN config file

    paster make-config ckan /etc/ckan/default/development.ini

### Edit the development.ini file in a text editor, changing the sqlalchemy.url  options:

    sudo nano /etc/ckan/default/development.ini

Replace pass with the password that you created in the setup of the postgres database above

    sqlalchemy.url = postgresql://ckan_default:pass@localhost/ckan_default

each CKAN site should have a unique site_id

    ckan.site_id = ims_gis

also provide the site’s URL (used when putting links to the site into the FileStore, notification emails etc). Do not add a trailing slash to the URL.

    ckan.site_url = http://127.0.0.1

## 5. Setup Solr

CKAN uses Solr as its search platform, and uses a customized Solr schema file that takes into account CKAN’s specific search needs  

### Edit the Jetty configuration file (/etc/default/jetty8)

    sudo nano /etc/default/jetty8

in the file

    NO_START=0            # (line 4)
    JETTY_HOST=127.0.0.1  # (line 16)
    JETTY_PORT=8983       # (line 19)/etc/nginx/sites-available/ckan

### Restart Solr

    sudo service jetty8 restart

You should now see a welcome page from Solr if you open `http://localhost:8983/solr/` in your web browser (replace localhost with your server address if needed).  

> Notice that the check on the browser is only possible if the virtual machine has installed a desktop version of Ubuntu, instead of a server version

### Uncomment the solr_url setting in your CKAN configuration file (/etc/ckan/default/development.ini) to point to your Solr server, for example:

    sudo nano /etc/ckan/default/development.ini

    solr_url=http://127.0.0.1:8983/solr

### Config SOLR and restart jetty8

    sudo mv /etc/solr/conf/schema.xml /etc/solr/conf/schema.xml.bak
    sudo ln -s /usr/lib/ckan/default/src/ckan/ckan/config/solr/schema.xml /etc/solr/conf/schema.xml
    sudo service jetty8 restart

## 6. Link to who.ini

who.ini (the Repoze.who configuration file) needs to be accessible in the same directory as your CKAN config file, so create a symlink to it:

    ln -s /usr/lib/ckan/default/src/ckan/who.ini /etc/ckan/default/who.ini

## 7. Create database tables

    cd /usr/lib/ckan/default/src/ckan
    paster db init -c /etc/ckan/default/development.ini

> You should see  
  Initialising DB: SUCCESS.

### Backup database

Contains empty tables except for one user `saeri` and some entries in `public.revision`

    pg_dump --blobs --clean --if-exists --create --inserts -d ckan_default -h localhost -U ckan_default -f ~/ckan_default.initialised.pg_dump

> user = saeri  
  password = saerickan

## 8. Set up the File Store

    sudo mkdir -p /var/lib/ckan/default
    sudo chown www-data /var/lib/ckan/default
    sudo chmod u+rwx /var/lib/ckan/default

    sudo nano /etc/ckan/default/development.ini

Change `ckan.storage_path` into the path created above and uncomment

    ckan.storage_path '/var/lib/ckan/default'

## 9. Set up the Data Store

DataStore is like a database in which individual data elements are accessible and "queryable".

    sudo mkdir -p /var/lib/ckan/default/storage/uploads/group
    sudo chown -R www-data /var/lib/ckan/default/storage

### Enable the plugin

Add the datastore plugin to your CKAN config file:

    sudo nano /etc/ckan/default/development.ini

add the name `datastore` to the list of plugins already there

    ckan.plugins = datastore

### Create users and databases

Create a `database_user` called `datastore_default`. This user will be given read-only access to your DataStore database in the Set Permissions step below:

    sudo -u postgres createuser -S -D -R -P -l datastore_default

> user = datastore_default  
  password = saeridatastore

Create the database (owned by `ckan_default`), which we’ll call `datastore_default` and give ownership to `ckan_default` user

    sudo -u postgres createdb -O ckan_default datastore_default -E utf-8

Uncomment the `ckan.datastore.write_url` and `ckan.datastore.read_url` lines in your CKAN config file and edit them

    sudo nano /etc/ckan/default/development.ini

    ckan.datastore.write_url = postgresql://ckan_default:pass@localhost/datastore_default
    ckan.datastore.read_url = postgresql://datastore_default:pass@localhost/datastore_default

> Replace pass with the passwords you created for your ckan_default and datastore_default database users (saerickan and saeridatastore)

### Set permissions

    paster --plugin=ckan datastore set-permissions -c /etc/ckan/default/development.ini | sudo -u postgres psql --set ON_ERROR_STOP=1

### backup the datastore database

    pg_dump --blobs --clean --if-exists --create --inserts -d ckan_default -h localhost -U ckan_default -f ~/ckan_default.initialised_with_datastore.pg_dump

## 10. Add Data pusher

### install requirements for the DataPusher

    sudo apt-get install python-dev python-virtualenv build-essential libxslt1-dev libxml2-dev git libffi-dev

in the 'development.ini' file add plugin 'datapusher' and uncomment the 'data_pusher.url

    sudo nano /etc/ckan/default/development.ini

    ckan.datapusher.url 'http://127.0.0.1:8800/'

### create a virtualenv for datapusher

    sudo virtualenv /usr/lib/ckan/datapusher

### create a source directory and switch to it

    sudo mkdir /usr/lib/ckan/datapusher/src
    cd /usr/lib/ckan/datapusher/src

### clone the source (this should target the latest tagged version)

    sudo git clone -b 0.0.14 https://github.com/ckan/datapusher.git

### install the DataPusher and its requirements

    cd datapusher
    sudo /usr/lib/ckan/datapusher/bin/pip install -r requirements.txt
    sudo /usr/lib/ckan/datapusher/bin/python setup.py develop

## 11. Create a production.ini File

Create your site’s `production.ini` file, by copying the `development.ini` file

    cp /etc/ckan/default/development.ini /etc/ckan/default/production.ini

## 12. Install Apache, modwsgi, modrpaf

    sudo apt-get install apache2 libapache2-mod-wsgi libapache2-mod-rpaf

## 13. Create the WSGI script file

Create your site’s WSGI script file `/etc/ckan/default/apache.wsgi` with the following contents:

    sudo nano /etc/ckan/default/apache.wsgi

with the following contents:

    import os
    activate_this = os.path.join('/usr/lib/ckan/default/bin/activate_this.py')
    execfile(activate_this, dict(__file__=activate_this))

    from paste.deploy import loadapp
    config_filepath = os.path.join(os.path.dirname(os.path.abspath(__file__)), 'production.ini')
    from paste.script.util.logging_config import fileConfig
    fileConfig(config_filepath)
    application = loadapp('config:%s' % config_filepath)

check if you are owner of the file

    cd /etc/ckan/default/
    ls -l

if not the owner run

    sudo chown `whoami` apache.wsgi

## 14. Create the Apache config file

    sudo nano /etc/ckan/default/apache_ckan_default.conf

with the following contents:

````
<VirtualHost 127.0.0.1:8080>
    ServerName default.ckanhosted.com
    ServerAlias www.default.ckanhosted.com
    WSGIScriptAlias / /etc/ckan/default/apache.wsgi

    # Pass authorization info on (needed for rest api).
    WSGIPassAuthorization On

    # Deploy as a daemon (avoids conflicts between CKAN instances).
    WSGIDaemonProcess ckan_default display-name=ckan_default processes=2 threads=15

    WSGIProcessGroup ckan_default

    ErrorLog /var/log/apache2/ckan_default.error.log
    CustomLog /var/log/apache2/ckan_default.custom.log combined

    <IfModule mod_rpaf.c>
        RPAFenable On
        RPAFsethostname On
        RPAFproxy_ips 127.0.0.1
    </IfModule>

    <Directory />
        Require all granted
    </Directory>

</VirtualHost>
````

Then run

    sudo cp /etc/ckan/default/apache_ckan_default.conf /etc/apache2/sites-available/ckan_default.conf

## 15. Modify the Apache ports.conf file

    sudo nano /etc/apache2/ports.conf

manually replace the default port 80 with the 8080 one.

## 16. Install DataPusher apache config

    cd /usr/lib/ckan/datapusher/src/datapusher

### Modify the max upload size from 10MB to 128MB

I can change the 128 with another upload size just remember that the formula is SIZEUPLOADING*1024*1024

    sudo sed -i -e '/MAX_CONTENT_LENGTH/d' -e '$aMAX_CONTENT_LENGTH=134217728' deployment/datapusher_settings.py

### check which version of apache

    apache2 -v

if not 2.4 replace below with apache2

    sudo cp deployment/datapusher.apache2-4.conf /etc/apache2/sites-available/datapusher.conf
    sudo cp deployment/datapusher.wsgi /etc/ckan/
    sudo cp deployment/datapusher_settings.py /etc/ckan/

DataPusher requires port 8800. Open up port 8800 on Apache where the DataPusher accepts connections. Make sure you only run these 2 functions once otherwise you will need to manually edit /`etc/apache2/ports.conf`.

````
sudo sh -c 'echo "NameVirtualHost *:8800" >> /etc/apache2/ports.conf'
sudo sh -c 'echo "Listen 8800" >> /etc/apache2/ports.conf'
````

check that all went well

    sudo nano /etc/apache2/ports.conf

### Enable the CKAN and datapusher sites in Apache and disable the default site

    sudo a2ensite ckan_default
    sudo a2ensite datapusher
    sudo a2dissite 000-default

## 17. Install nginx now that apache has been moved to port 8080**

    sudo service apache2 stop
    sudo apt-get install nginx

### Create the Nginx config file

    sudo nano /etc/ckan/default/nginx_ckan

with the following contents:
````
proxy_cache_path /tmp/nginx_cache levels=1:2 keys_zone=cache:30m max_size=250m;
proxy_temp_path /tmp/nginx_proxy 1 2;

server {
    client_max_body_size 100M;
    location / {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host $host;
        proxy_cache cache;
        proxy_cache_bypass $cookie_auth_tkt;
        proxy_no_cache $cookie_auth_tkt;
        proxy_cache_valid 30m;
        proxy_cache_key $host$scheme$proxy_host$request_uri;
        # In emergency comment out line to force caching
        # proxy_ignore_headers X-Accel-Expires Expires Cache-Control;
    }

}
````
--

    sudo cp /etc/ckan/default/nginx_ckan /etc/nginx/sites-available/ckan
    sudo rm -vi /etc/nginx/sites-enabled/default

#### reply yes

    sudo ln -s /etc/nginx/sites-available/ckan /etc/nginx/sites-enabled/ckan_default

### Enable apache and nginx

    sudo service apache2 start
    sudo service nginx start
    sudo service apache2 reload
    sudo service nginx reload

### check that datapusher service is working

    curl 127.0.0.1:8800

this should result with

````
{
"help": "\n        Get help at:\n        http://ckan-service-provider.readthedocs.org/."
}
````
### Backup database (now that the datapusher has been installed)

    pg_dump --blobs --clean --if-exists --create --inserts -d ckan_default -h localhost -U ckan_default -f ~/ckan_default.initialised_with_datapusher.pg_dump

### Test if the DataStore is set-up properly

    curl -X GET "http://127.0.0.1/api/3/action/datastore_search?resource_id=_table_metadata"

This should return a JSON page without errors. a line in the terminal starting with `{“help”`, nothing on the browser

## 18.Create a sysadmin

> Remember the password needs to be 8 character long or more  
  user = ilaria  
  password = saeri2019  

    . /usr/lib/ckan/default/bin/activate
    cd /usr/lib/ckan/default/src/ckan
    paster sysadmin add ilaria email=imarengo@saeri.ac.fk name=ilaria -c /etc/ckan/default/production.ini

apikey and id will change everytime, just COPY ON A TEXT FILE AS WILL BE NEEDED LATER ON

>'apikey': u'2052bcbf-e698-40ed-95b2-9e309320e1c1',  
'id': u'ff69e695-3747-4c8c-9379-d9ae5ca18a6a',

### Backup database (now that the sysadmin has been created)

    pg_dump --blobs --clean --if-exists --create --inserts -d ckan_default -h localhost -U ckan_default -f ~/ckan_default.initialised_with_sysadmin.pg_dump
