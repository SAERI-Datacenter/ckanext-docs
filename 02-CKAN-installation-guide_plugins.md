# Plugins and extensions

## Start development environment

    . /usr/lib/ckan/default/bin/activate

## Add the recline plugin to the list of plugin in the `development.ini` file

    sudo nano /etc/ckan/default/development.ini

Check first that `recline_view` is already in the list. Is not add it with all the others

    ckan.plugins: recline_grid_view recline_map_view recline_graph_view

    ckan.views.default_views: recline_view

## Add PDF viewer

    pip install ckanext-pdfview

    sudo nano /etc/ckan/default/development.ini

    ckan.plugins: pdf_view

## Add Geo(spatial) viewer

    pip install ckanext-geoview

    sudo nano /etc/ckan/default/development.ini

    ckan.plugins: resource_proxy geo_view geojson_view wmts_view
    ckan.views.default_views: geo_view geojson_view wmts_view

## Copy the `development`.ini to the production and restart apache2

    cp /etc/ckan/default/development.ini /etc/ckan/default/production.ini
    sudo service apache2 restart

## Install PostGIS

    sudo apt-get install postgresql-9.5-postgis-2.2

### Create tables, functions; fill spatial reference table

     sudo -u postgres psql -d ckan_default -f /usr/share/postgresql/9.5/contrib/postgis-2.2/postgis.sql
     sudo -u postgres psql -d ckan_default -f /usr/share/postgresql/9.5/contrib/postgis-2.2/rtpostgis.sql
     sudo -u postgres psql -d ckan_default -f /usr/share/postgresql/9.5/contrib/postgis-2.2/spatial_ref_sys.sql

### Change owner of spatial tables, from postgres to ckan_default

    sudo -u postgres psql -d ckan_default -c 'ALTER VIEW geometry_columns OWNER TO ckan_default;'
    sudo -u postgres psql -d ckan_default -c 'ALTER TABLE spatial_ref_sys OWNER TO ckan_default;'

### Check installation

    sudo -u postgres psql -d ckan_default -c "SELECT postgis_full_version()"

> the above should return  
PostGIS 2.2.1, GEOS 3.5.0, Proj 4.9.2, libxml 2.9.3, libjson 0.11.99, GDAL, RASTER

## Adding more stuff

    sudo apt-get install python-dev libxml2-dev libxslt1-dev libgeos-c1v5

## Spatial extension

    pip install -e "git+https://github.com/SAERI-Datacenter/ckanext-spatial.git#egg=ckanext-spatial"
    pip install -r /usr/lib/ckan/default/src/ckanext-spatial/pip-requirements.txt

## backup database

    pg_dump --blobs --clean --if-exists --create --inserts -d ckan_default -h localhost -U ckan_default -f ~/ckan_default.initialised_with_postgis.pg_dump

    sudo nano /etc/ckan/default/development.ini

## add plugins

    ckan.plugins spatial_metadata spatial_query

### add the following under the plugins section

    ckan.spatial.srid = 4326
    ckanext.spatial.search_backend = solr

### and also add the following to set the map tile source

    ckanext.spatial.common_map.type = custom
    ckanext.spatial.common_map.custom.url = http://tile.stamen.com/terrain/{z}/{x}/{y}.jpg
    ckanext.spatial.common_map.attribution = Map tiles by <a href="http://stamen.com">Stamen Design</a>, under <a href="http://creativecommons.org/licenses/by/3.0">CC BY 3.0</a>. Data by <a href="http://openstreetmap.org">OpenStreetMap</a>, under <a href="http://creativecommons.org/licenses/by-sa/3.0">CC BY SA</a>.

### Copy `development.ini` file to `production.ini` file to configure ckan

    cp /etc/ckan/default/development.ini /etc/ckan/default/production.ini

## Initialise spatial database, a database with postgis extension and default 4326 crs**

    cd /usr/lib/ckan/default/src/ckan

    paster --plugin=ckanext-spatial spatial initdb 4326 --config=/etc/ckan/default/production.ini

## backup database and restart apache2

    pg_dump --blobs --clean --if-exists --create --inserts -d ckan_default -h localhost -U ckan_default -f ~/ckan_default.initialised_with_postgis_and_table.pg_dump

    sudo service apache2 reload

## backup database

  pg_dump --blobs --clean --if-exists --create --inserts -d ckan_default -h localhost -U ckan_default -f ~/ckan_default.initialised_with_spatial.pg_dump


## To list all the plugins added to CKAN

Type `http://CKANSERVERIP/api/action/status_show` in the browser
