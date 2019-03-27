# Instructions on how to modify the front page of CKAN

It is worth reminding that each customized extension requires two folders: `templates` and `public` that need to be added to the search path so that any files placed in them will override or supplement the built-in ones.

## How to personalize CKAN pages

The files that need to be considered for the customization are located in:

    /usr/lib/ckan/default/src/ckanext-saeritheme/ckanext/saeritheme/

Under the following structure

````
saeritheme/
  ├─ fanstatic/
  ├─ i18n/
  ├─ public/
  │   ├─ logo/
  │   └─ media/
  │       ├─ fi/
  │       ├─ ms/
  │       └─ saeritheme.css
  ├─ templates/
  │   ├─ group/
  │   │   └─ snippets/
  │   │       ├─ group_item.html
  │   │       └─ group_list.html
  │   ├─ home/
  │   │   ├─ snippets/
  │   │   │   └─ about_text.html  
  │   │   ├─ index.html
  │   │   └─ layout1.html
  │   └─ base.html
  └─ plugin.py
````
### saeritheme.css

    /usr/lib/ckan/default/src/ckanext-saeritheme/ckanext/saeritheme/public/saeritheme.css

The additional styles must be added in this file. Current ones can be redefined as well by assigning different values to the correspondent classes.  
To use the original CKAN color schemes (Default, Green...) instead the SAERI's one, just comment `/* */` the lines `background: #004368;` and `background: #4f85c0;`.  

````css
/* SAERI's dark_blue = #004368 */
.account-masthead,
.homepage .module-search .tags {
	background: #004368;
}

/* SAERI's pastel_light_blue = #4f85c0 */
.masthead,
.site-footer,
.homepage .module-search .module-content {
	background: #4f85c0;
}
````

### layout1.html

This file is determining the content of the first page, while the css file was only providing the style of it.

    /usr/lib/ckan/default/src/ckanext-saeritheme/ckanext/saeritheme/templates/home/layout1.html

The block `<div class="images-wrapper">` defines the changing images (slide) on the top of the fromtpage. In this case the images comes from `media/fi/` in the `public` folder (notice that it is a relative path), but can be used any image reachable by Apache even an external URL.  
Notice that the first one does not include `display:none;` which means that it is the first one that appears when the page is loaded.

> NOTE to the changes take effect apache2 must be restarted

    sudo service apache2 restart

````html
{% resource 'saeritheme/homepage-slider.css' %}
{% resource 'saeritheme/homepage-slider.js' %}

<div class="homepage-slider-ng">
  <div class="images-wrapper">
    <div class="text-wrapper" style="background-image:url(media/fi/P1050796.JPG)"></div>
    <div class="text-wrapper" style="background-image:url(media/fi/P1060902.JPG);display:none;"></div>
    <div class="text-wrapper" style="background-image:url(media/fi/P1070744.JPG);display:none;"></div>
  </div>

  <div class="boxes-wrapper container">
    <div class="row">
      <div class="col-md-6 col-md-offset-6">
        <div class="home-box">
          {% block search %}
          {# we use the built-in search snippet #}
          {% snippet 'home/snippets/search.html' %}
          {% endblock %}
        </div>
      </div>
    </div>
  </div>
</div>

<div role="main">
  <div class="container">
    <div class="row">
      {# we've provided our own helper function to get a list of groups/themes #}
      {# see plugin.py in the saeritheme plugin #}
      {# each group is rendered using the code in this file: #}
      {# "/usr/lib/ckan/default/src/ckan/ckan/templates/group/snippets/group_item.html" #}
      {% snippet 'group/snippets/group_list.html', groups=h.saeritheme_get_groups_list() %}
    </div>

    <div class="row">
        <div id="latest-addings" class="box">
          <header class="module-heading">
            <h2>Latest addings</h2>
          </header>
          <section class="module-content">
            {# This sh
              ows the recent activity but has been modified from #}
            {# what is in the docs to prevent raw html being displayed #}
            {{ h.recently_changed_packages_activity_stream(limit=4)|safe }}
          </section>
        </div>
      </div>
  </div>
</div>
````

### base.html

The html file contains the reference to the block that tells CKAN to consider the stylesheet that we have personalised (and indeed created). The stylesheet is called `saeritheme.css`

    /usr/lib/ckan/default/src/ckanext-saeritheme/ckanext/saeritheme/templates/base.html

content:

````html
{% ckan_extends %}

{# This file "extends" the base.html file #}
{# /usr/lib/ckan/default/src/ckan/ckan/templates/base.html #}
{# and replaces the "styles" block within in #}

{% block styles %}

  {# include the content of the original master "styles" block #}
  {{ super() }}

  <link rel="stylesheet" href="saeritheme.css" />

{% endblock %}
````

### group_item.html

    /usr/lib/ckan/default/src/ckanext-saeritheme/ckanext/saeritheme/templates/group/snippets/group_item.html

````html
{% ckan_extends %}
{# this file extends the original #}
{# /usr/lib/ckan/default/src/ckan/ckan/templates/group/snippets/group_item.html #}
{# so that we can change or replace blocks #}

{# remove the 'number of datasets' from the description of the group #}
{# but only if the homepage variable is 1, which is only set in layout1.html #}

{% block datasets %}
  {% if homepage==1 %}
  {% else %}
    {{ super() }}
  {% endif %}
{% endblock datasets %}

{# we have copied exactly the original <img> tag but added saeri_group_icon to the list of classes #}
{# this will allow you to add your own CSS in saerickan.css eg. to make the icons smaller #}
{% block image %}
  <img src="{{ group.image_display_url or h.url_for_static('/base/images/placeholder-group.png') }}" alt="{{ group.name }}" class="media-image img-responsive saeri_group_icon">
{% endblock %}
````

### group_list.html

    /usr/lib/ckan/default/src/ckanext-saeritheme/ckanext/saeritheme/templates/group/snippets/group_list.html

````html
{% ckan_extends %}
{# this file extends the original #}
{# /usr/lib/ckan/default/src/ckan/ckan/templates/group/snippets/group_list.html #}
{# so that we can change or replace blocks #}

{# all we have done is copy exactly the original block, but pass on the homepage variable to the child #}
{% block group_list_inner %}
  {% for group in groups %}
    {% snippet "group/snippets/group_item.html", group=group, position=loop.index, homepage=homepage %}
  {% endfor %}
{% endblock %}
````
### plugin.py

    /usr/lib/ckan/default/src/ckanext-saeritheme/ckanext/saeritheme/plugin.py

This plugin defines the theme for CKAN. At the very end there is the line that call the `h.saeri_theme_get_list` function as it is written the function is based on another function that comes from the defaul CKAN `group/snippets/group_list.html` below the content of the `group_list.html`. Basically it makes a list of the topic categories then it used the `group_item.html` to position them in order.

````python
# IConfigurer

  def update_config(self, config_):
      toolkit.add_template_directory(config_, 'templates')
      toolkit.add_public_directory(config_, 'public')
      toolkit.add_resource('fanstatic', 'saeritheme') # needed to add .js
````

## Change Groups to Themes in the top navigation bar

    cd /usr/lib/ckan/default/src/ckan
    python setup.py update_catalog --locale en_GB
    sed -i.bck -e 's/^msgstr "Groups"/msgstr "Themes"/'  /usr/lib/ckan/default/src/ckan/ckan/i18n/en_GB/LC_MESSAGES/ckan.po
    python setup.py compile_catalog --use-fuzzy --locale en_GB

### Change the locale in your configuration file, called $ini here:

    crudini --set --inplace $ini app:main ckan.locale_default en_GB
    crudini --set --inplace $ini app:main ckan.locales_offered en_GB
    crudini --set --inplace $ini app:main ckan.locales_filtered_out '' # was en_GB!!
    crudini --set --inplace $ini app:main ckan.locale_order "en_GB en pt_BR ja it cs_CZ ca es fr el sv sr sr@latin no sk fi ru de pl nl bg ko_KR hu sa sl lv"

### Ensure web server can create new JS file

> might not need to do this

    sudo chmod 666 /usr/lib/ckan/default/src/ckan/ckan/public/base/i18n/*.js
