# CKAN users settings and management

Start with activate the virtual environment

    . /usr/lib/ckan/default/bin/activate
    cd /usr/lib/ckan/default/src/ckan

use paster for adding/deleting/listing USERS

    paster sysadmin --help
    paster user --help

> NOTE if paster doesn't work and there is an error message related to saerischema or saeritheme production

    cd /usr/lib/ckan/default/src/ckanext-saerischema
    python setup.py develop

## Users management

### List sysadmin and users

    paster sysadmin list -c /etc/ckan/default/production.ini
    paster user list -c /etc/ckan/default/production.ini

> NOTE that the list of users includes sysadmins.

There is not an apparent way to know what the user is able to do: admin, member or editor

### Add BASIC user

    paster user add USERNAME -c /etc/ckan/default/production.ini

### Remove user

    paster user remove USERNAME -c /etc/ckan/default/production.ini

### Set password

    paster user setpass USERNAME -c /etc/ckan/default/production.ini

### Show user properties

> NOTE it doesn't tell whether the user is admin, member or editor or if belongs to any organization or if it is able to see any dataset. It is a useless command.

    paster user USERNAME -c /etc/ckan/default/production.ini

## How the management of users works

Login is not needed to search for and find data, but is needed for all publishing functions and for getting restricted data:  
Datasets can be created, edited, AND RETRIEVED (in case of restricted data) by users with the appropriate permissions.

## Example of registration for accessing restricted data

So, for example, a user interested in a dataset, after finding it and discovering that the dataset is restricted it has to register. Once the user has registered, an email is sent to the data manager who will become aware that somebody has logged in and it is looking for specific datasets. At this point the data manager checks that the data can be released and if so, logs to the data portal as sysadmin finds the datasets, click on the resources, click on the manage icon and then scroll at the end of the page and ADD THE USER IN THE ALLOWED USERNAMES.

> NOTE that if i have multiple users for a dataset, these are going to be separate by a comma NOT FOLLOWED BY SPACE.

## Example of registration for giving a user the rights to be member, editor or admin of an organization

The data manager interact with the user and understand that he/she requires to be either member, admin or editor of data withing his/her organization or other organizations.

The latter only with the permission of the owner of the third party organization.

Once it is clear what a user can or cannot do with the data within and organization, the data manager logs to the data portal as sysadmin click on the organization name, click on manage and then click on the tab called members. IN HERE I WILL ADD THE USER AND GIVE HIM/HER THE STATUS OF ADMIN, MEMBER, or EDITOR.

    Member – can see the organization’s private datasets
    Editor – can edit and publish datasets
    Admin – can add, remove and change roles for organization members

## See all the users registered in the data portal

    ip address of the server/user
