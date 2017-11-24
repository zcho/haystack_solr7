# Haystack Solr 7

## Test Java
Solr 7 required Java 8+ to run.

```
$ java -version
openjdk version "1.8.0_141"
OpenJDK Runtime Environment (build 1.8.0_141-8u141-b15-3~14.04-b15)
OpenJDK 64-Bit Server VM (build 25.141-b15, mixed mode)
```

## Download and install Solr 7 as a service

```
cd
wget http://www.us.apache.org/dist/lucene/solr/7.1.0/solr-7.1.0.tgz
tar xzf solr-7.1.0.tgz solr-7.1.0/bin/install_solr_service.sh --strip-components=2
sudo bash ./install_solr_service.sh solr-7.1.0.tgz

sudo service solr restart
```

## Create a default Solr core
sudo su - solr -c '/opt/solr/bin/solr create -c mycore1'

