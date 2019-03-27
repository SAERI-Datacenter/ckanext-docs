# Instructions for saeri-theme

Activate virtual environment

    . /usr/lib/ckan/default/bin/activate

Then run the following

    pip install pyproj

get the theme extension from Github

    cd /usr/lib/ckan/default/src

    git clone https://github.com/SAERI-Datacenter/ckanext-saeritheme.git
    cd ckanext-saeritheme
    python setup.py develop

add the `saeritheme` plugin to the `production.ini`

    cd /etc/ckan/default

    nano production.ini
    sudo service apache2 restart

> NOTE to update the extension just type

    cd /usr/lib/ckan/default/src/ckanext-saeritheme

then

    git pull

then make the logo images accessible using

    sudo ln -s /usr/lib/ckan/default/src/ckanext-saeritheme/ckanext/saeritheme/public/logo /var/lib/ckan/default/storage/uploads/group/logo

create two text files with indication of the CKAN API and IP ADDRESS OF THE CKAN SERVER

> The API key can be found once logged into CKAN on sysadmin profile (bottom of the left-hand column).

    cd /usr/lib/ckan/default/src/ckanext-saeritheme/ckanext/saeritheme/tools

    nano ckan_ip.txt
    nano ckan_api_key.txt

check that the ownership of the text files is NOT root

    ls -l

if it is root then write

    sudo chown ckan4a:ckan4a ckan_api_key.txt

`ckan4a` is the ubuntu user, notice it will change from VM to VM

Then copy in this folder two csv files one listing the topic categories and the other the organisations

Call the files `organisation_list.csv` and `topic_categories.csv`

these two files can be created outside CKAN and imported into this directory by using the following lines

Open a terminal window and type the following lines:

    scp path_to/organisation_list.csv  ckan4a@192.168.254.129:/usr/lib/ckan/default/src/ckanext-saeritheme/ckanext/saeritheme/tools

    scp path_to/topic_category/topic_categories.csv  ckan4a@192.168.254.129:/usr/lib/ckan/default/src/ckanext-saeritheme/ckanext/saeritheme/tools

Add organizations logos here

    scp /media/warrah/1TB/Dundee/CKAN/organisations_logos/montserrat/*.* ckan4a@192.168.254.129:/usr/lib/ckan/default/src/ckanext-saeritheme/ckanext/saeritheme/public/logo

Restart Apache

    service apache2 restart

**Best practice** is to maintain the logo names as given in the first upload to CKAN. So if a logo for organisation or topic category is changed is better to rename the logo with the same name of the file already in the `saeritheme/public/logo` directory . The reason is because the logo, although changed, will not appear changed also when apache2 is restarted. This is a known issue of CKAN.

The alternative to the above best practice is to open `ckan_add_organisations.py` and uncomment the line which says `#org_dict['clear_upload'] = True` so that a change to `image_url` is forced, only use if image has actually changed

**REMEMBER** that after running the script this line **MUST BE COMMENTED BACK** again.

verify that the file below look at the `organisation_list.csv` under configuration

    nano ckan_add_organisations.py

verify that the file below look at the `topic_categories.csv` under configuration

    nano ckan_add_groups.py

run

    pip install ckanapi

disregard the warning message

Then run the python scripts to add NEW organisations and list the organisations ALREADY in CKAN

    ./ckan_add_organisations.py

    ./ckan_list_organisations.py

run the command below to add the groups/themes (topic_categories)

    ./ckan_add_groups.py

## How to set the about section

In order to fill in the about section in CKAN it is necessary to upload the `about_text.html` file from the local directory to the server.

BEFORE DOING SO, however, make sure that any image used by the `about_text.html` is uplaoded in

    cd /usr/lib/ckan/default/src/ckanext-saeritheme/ckanext/saeritheme/public/media/fi/

> NOTE that the images must be copied into the territory folder in `/public/media/`  
  `fi/` for Falklands  
  `ms/` for Montserrat  
  and so on

then import the `about_text.html` file

    scp /pathTO/about_text.html  ckan4a@192.168.254.129:/usr/lib/ckan/default/src/ckanext-saeritheme/ckanext/saeritheme/templates/home/snippets

restart apache2 afterwards

    sudo service apache2 restart

Once the above tasks have been completed move to the ckanext schema directory

    cd /usr/lib/ckan/default/src
    git clone https://github.com/SAERI-Datacenter/ckanext-saerischema.git
    cd ckanext-saerischema
    python setup.py develop

add the `saerischema` plugin to the `production.ini` file plugin list and restart apache

    cd /etc/ckan/default

    nano production.ini
    sudo service apache2 restart

> NOTE to update the extension just type

    cd /usr/lib/ckan/default/src/ckanext-saerischema

then

    git pull

intall the `restricted` extension

    cd /usr/lib/ckan/default/src
    git clone https://github.com/SAERI-Datacenter/ckanext-restricted.git
    cd ckanext-restricted
    python setup.py develop
    pip install -r dev-requirements.txt

add the restricted plugin to the `production.ini` file plugin list and restart apache

    cd /etc/ckan/default

    nano production.ini
    sudo service apache2 restart

> NOTE to update the extension just type

    cd /usr/lib/ckan/default/src/ckanext-restricted

then

    git pull

**IMPORTANT** if a server error appears on the browser than follow this procedure check the log files of apache2

    ls -l /var/log/apache2

look at the last one

    tail /var/log/apache2/ckan_default.error.log

**If the error is with a language file,** E.G. `fr.js` then do the change the permission of the file so that everyone can read the file

    sudo chmod 666 /usr/lib/ckan/default/src/ckan/ckan/public/base/i18n/fr.js

go to the `saerischema` directory and create two txt files: `ckan_ip.txt` and `ckan_api_key.txt`

    cd /usr/lib/ckan/default/src/ckanext-saerischema/ckanext/saerischema

    nano ckan_api_key.txt

    sudo nano ckan_id.txt

in the same directory import the metadata file, which is a csv file. MAKE SURE THAT THE IMPORTED VERSION HAS ALL THE QULITY CHECKS DONE

REMEMBER TO OPEN A NEW TERMINAL and run the line below from there

    scp /media/warrah/1TB/Dundee/CKAN/metadata/montserrat/METADATA_FILENAME.csv ckan4a@192.168.254.129:/usr/lib/ckan/default/src/ckanext-saerischema/ckanext/saerischema

run a test on the imported metadata file which allow to find and spot mistakes if any is still left

use the single quotes if the metadata file name has got spaces

for example to print a list of the number of times each unique "limitations_access" occurs:

    ./metadata_read_test.py METADATA_FILENAME.csv limitations_access | sort | uniq -c | sort -n

import the metadata file content to CKAN

    ./ckan_add_dataset.py METADATA_FILENAME.csv

**REMEMEBER** that everytime a metadata file is imported to CKAN the `. /ckan_add_dataset.py` METADATA_FILENAME.csv MUST be RUN

Run this program ONLY to delete all recently-added datasets. This is intended to allow you to delete datasets which were added mistakenly or during testing, but keep all older ones. The cut-off date is configured inside the code so you will need to edit it.

    ./ckan_delete_dataset.py
