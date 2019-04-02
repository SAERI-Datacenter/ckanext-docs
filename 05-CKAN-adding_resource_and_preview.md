# Adding single resource (data) to a dataset (metadata)

create a temp folder where the data are saved in the CKAN VM

    sudo mkdir /var/lib/ckan/tmp

then  change the permissions for the folder so that it is possible to write in the folder

    sudo chmod 777 /var/lib/ckan/tmp

install the following packages

    sudo apt-get install gdal-bin python-gdal jq

rename the following files

    cd  /usr/lib/ckan/default/lib/python2.7/

    mv no-global-site-packages.txt renamed-no-global-site-packages.txt

> NOTE the data must be somewhere in the VM or in a VM that CKAN can reach.

> NOTE remember that we can change the upload size of the file by changing two lines in the production.ini file

    sudo nano /etc/ckan/default/production.ini

under storage settings there are:

    ckan.max_resource_size = 512
    ckan.max_image_size = 32

change the number in MB according to what could be the maximum size of the datasets to be uploaded
Restart apache2

    sudo service apache2 restart

Alternatively, run the line below

    sudo sed -i -e '/MAX_CONTENT_LENGTH/d' -e '$aMAX_CONTENT_LENGTH=134217728' deployment/datapusher_settings.py

bearing in mind that the number `134217728 (128MB)` should be changed into the new UPLOAD SIZE
> The formula is SIZE-of-UPLOAD-in-MB*1024*1024

THEN, go to

    cd /usr/lib/ckan/default/src/ckanext-saerischema/ckanext/saerischema/tools

then run the script

    ./ckan_add_resource_to_dataset.py -d dataset_name -f file_name -t file_type -s CRS -r restriction -u allowed_users

example
    ./ckan_add_resource_to_dataset.py -d montserrat-roads -f MainRoad.zip -t shp -s epsg:2004 -r restriction -u test_user

explanation

````
-d dataset_name
-f filename_to_upload
-t type_of_file
-s CRS [please type epsg:NUMBER]
-r is public, registered, any_organization, same_organization, only_allowed_users
-u is a comma-separated list of usernames
````

> Note that if the data are open then the -r and -u are commented in brakets

    ./ckan_add_resource_to_dataset.py -d dataset_name -f file_name -t file_type -s CRS [-r restriction [-u allowed_users]]

## IMPORTANT

Due to the size of some shapefiles, the script simplifies the geometry of the shape file in the preview in proportion to the geographic extent (bbox) of the file. However, the oriinal shapefile (zipped) keeps its original form.

# Adding multiple resource (data) to a dataset (metadata)

first of all go to

    cd /usr/lib/ckan/default/src/ckanext-saerischema/ckanext/saerischema/tools

then run

    ./ckan_add_all_resources.py -c montserrat_metadata20190325.csv -r /home/ckan4a/montserrat/data_repository

````
-c csv file with metadata
-r path to where data are stored (repository) Note that the repo must be in the VM or linked to it

````


## ANOTHER IMPORTANT POINT ABOUT CKAN_ADD_ALL_RESOURCES.PY

in order to allow the geojson preview the script runs a gdal commad that is called ogr2ogr

gdal_cmd = "ogr2ogr -f geojson %s %s -t_srs WGS84  /vsistdout/  /vsizip/%s

this command needs to be integrated with -s_srs epsg:2004 which specifies that the CRS at the original file is in epsg 2004 coordinate system.

HENCE PAY ATTENTION TO THE FILES IN THE OTHER TERRITORIES AS IF THERE IS MORE THAN ONE COORDINATE SYSTEM IN THE SOURCE ANOTHER LINE SHOULD BE ADDED TO THE SCRIPT
