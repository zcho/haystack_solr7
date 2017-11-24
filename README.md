# Haystack Solr 7

## Test Java
Solr 7 required Java 8+ to run.

```sh
$ java -version
openjdk version "1.8.0_141"
OpenJDK Runtime Environment (build 1.8.0_141-8u141-b15-3~14.04-b15)
OpenJDK 64-Bit Server VM (build 25.141-b15, mixed mode)
```

## Download and install Solr 7 as a service

```sh
$ cd
$ wget http://www.us.apache.org/dist/lucene/solr/7.1.0/solr-7.1.0.tgz
$ tar xzf solr-7.1.0.tgz solr-7.1.0/bin/install_solr_service.sh --strip-components=2
$ sudo bash ./install_solr_service.sh solr-7.1.0.tgz

Solr process 1127 running on port 8983
{
  "solr_home":"/var/solr/data",
  "version":"7.1.0 84c90ad2c0218156c840e19a64d72b8a38550659 - ubuntu - 2017-10-13 16:15:59",
  "startTime":"2017-11-24T08:46:11.230Z",
  "uptime":"0 days, 0 hours, 0 minutes, 13 seconds",
  "memory":"22.9 MB (%4.7) of 490.7 MB"}
  
$ sudo service solr restart
```

## Create a default Solr core

Core creation needs to be performed on behalf of solr user because Solr would run into permission problems otherwise

```sh
sudo su - solr -c '/opt/solr/bin/solr create -c mycore1'
```

## Configuring the solrconfig.xml
Open */var/solr/data/mycore1/conf/solrconfig.xml*
Search for configrations for ManagedIndexSchemaFactory and remove them all.
Add following configuration to the solrconfig.xml file
```xml
<schemaFactory class="ClassicIndexSchemaFactory"/>
```
Search for the following XML element and remove the whole element
```xml
<processor class="solr.AddSchemaFieldsUpdateProcessorFactory">
  ...
  ...
</processor>
  ```
Rename the /var/solr/data/mycore1/conf/managed-schema to schema.xml

Restart the solr instance.
```
$ sudo service solr restart
```
Goto Solr admin, and make sure the core is working without any issues.
```
http://localhost:8983/solr/#/mycore1


# Install Haystack and pysolr

[haystack installallation](http://django-haystack.readthedocs.io/en/latest/tutorial.html#installation)

```sh
(env)$ pip install django-haystack
(env)$ pip install pysolr
```

# Override django-haystack default template for schema.xml

* Grab `/search_configuration/solr.xml` file from this repository and put it into your Django project where `django-haystack` will be able to locate the template (more on this [here](http://django-haystack.readthedocs.org/en/v2.4.0/installing_search_engines.html#solr)), such as:

```
<project_name>/templates/search_configuration/solr.xml
```

* `solr.xml` is how `django-haystack` likes to call `schema.xml` templates.
* Now you can use analyzers of your preference by modifying the template.
* You can see the `django-haystack`-specific config near the top of the template, so if you ever need to use *another* initial template, make sure to remove from your template the declarations for `<field name="id" ...` and for `<uniqueKey>` as these will be declared by the `django-haystack`-specific config and copy the `django-haystack`-specific config into the same spot in your template.

### Fine-tune `django-haystack` settings

* Modify the settings for `django-haystack` in your Django settings to make it talk to a specific core:

```python
HAYSTACK_CONNECTIONS = {
    "default": {
        "ENGINE": "haystack.backends.solr_backend.SolrEngine",
        "URL": "http://127.0.0.1:8983/solr/mycore1"
    },
    # ... other settings ...
}
```

### Make it easy to rebuild Solr schema

* There is no need for manual `schema.xml` copying and Solr restarting whenever you rebuild your schema for `django-haystack` as the schema can be put directly into the core's config and the core can then be reloaded, which is way faster than restarting the entire server, all with a single command:

```sh
python manage.py build_solr_schema --filename=/var/solr/data/mycore1/conf/schema.xml && curl 'http://localhost:8983/solr/admin/cores?action=RELOAD&core=mycore1&wt=json&indent=true'
```

* You could also add ` && python manage.py rebuild_index` to the command if you've modified fields in your `SearchIndex` classes and want the changes to get reflected in the index for all model instances regardless of when they were or will be indexed.
* You could place your other Solr config files under `search_configuration`, such as `solrconfig.xml` or `synonyms.txt`, and sync it all together (with e.g. `rsync --exclude=solr.xml ...`).
* You will likely need to change the permissions on `/var/solr/data/<core_name>` to more liberal ones for the above line to execute.


