# Adding single resource (data) to a dataset (metadata)

install the following packages

    sudo apt-get install gdal-bin python-gdal jq

rename the following files

    cd  /usr/lib/ckan/default/lib/python2.7/

    mv no-global-site-packages.txt renamed-no-global-site-packages.txt

> NOTE the data must be somewhere in the VM or in a VM that CKAN can reach.

go to

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

REMEMBER THAT WE HAVE SET AT THE BEGINNING A MAXIMUM UPLOAD SIZE SO WE WILL HAVE AN ERROR MESSAGE IF WE UPLOAD A FILE OF A SIZE GREATER THAT THE DEFINED VALUE

To change the max file size upload run the line below but change the number `134217728 (128MB)`

> The formula is SIZE-of-UPLOAD-in-MB*1024*1024

    sudo sed -i -e '/MAX_CONTENT_LENGTH/d' -e '$aMAX_CONTENT_LENGTH=134217728' deployment/datapusher_settings.py

## IMPORTANT

Due to the size of some shapefiles, the script simplifies the geometry of the shape file in the preview in proportion to the geographic extent (bbox) of the file. However, the oriinal shapefile (zipped) keeps its original form.
