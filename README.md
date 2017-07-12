This Github repo is a collection of key configuration files from Digital Grinnell's FEDORA repository (currently RepositoryX) server and the /usr/local/fedora/solr/collection1/conf directory, and others, found on that server. Solr and FedoraGSearch were initially upgraded to this configruation on RepositoryX in late-June and early-July 2017 following the process documented below.  The procedural portion of this document is a reflection of https://github.com/discoverygarden/basic-solr-config/wiki/Guide-to-Setting-up-GSearch-2.7-with-Solr-4.2.0, but with added detail and targeting upgrade to Solr version 4.2.1 rather than 4.2.0.

# Updating GSearch 2.7 and Solr 4.2.1 on RepositoryX
This document is a DG-specific custom copy of https://github.com/discoverygarden/basic-solr-config/wiki/Guide-to-Setting-up-GSearch-2.7-with-Solr-4.2.0.  It includes modifications required to successfully upgrade DG's RepositoryX server in July 2017.  Some portions of the source document have been retained but may appear in strikethrough text like ~~this~~.

This document assumes that $FEDORA_HOME is defined as /usr/local/fedora.

# Clone this Repository
Clone this repository to the target server so that all files within are readily available at /usr/local/fedora/.configuration.
```
cd /usr/local/fedora
git clone https://github.com/DigitalGrinnell/RepositoryX.git /usr/local/fedora/.configuration
```

# Install Solr 4.2.1

Stop Fedora by stopping Tomcat. 
```
service tomcat stop
```
*Remove all instances of Solr 4.2.0, and any other earlier versions, as well as FedoraGSearch from the server.*
```
rm -fr /opt/solr-4.2.0*
rm -f /opt/solr
rm -fr /usr/local/fedora/solr
rm -fr /usr/local/fedora/tomcat/webapps/solr*
rm -fr /usr/local/fedora/tomcat/webapps/fedoragsearch*
mv -f /usr/local/fedora/tomcat/work/ /usr/local/fedora/tomcat/.out-of-the-way/    # just a precaution
```

Download [Solr 4.2.1](http://archive.apache.org/dist/lucene/solr/4.2.1/solr-4.2.1.tgz) to /opt 
```
cd /opt
wget http://archive.apache.org/dist/lucene/solr/4.2.1/solr-4.2.1.tgz
tar -xvzf solr-4.2.1.tgz
```

Make a symbolic link to the solr directory such that /opt/solr => /opt/solr-4.2.1

```
ln -s /opt/solr-4.2.1 /opt/solr
```

Add the following XML file to $FEDORA_HOME/tomcat/conf/Catalina/localhost/solr.xml

```XML
<?xml version="1.0" encoding="UTF-8"?>
<Context docBase="/opt/solr/dist/solr-4.2.1.war" debug="0" crossContext="true">
  <Environment name="solr/home" type="java.lang.String" value="/usr/local/fedora/solr" override="true"/>
</Context>
```

Copy over the default configuration files for "collection1" from /opt/solr/example/solr/collection1 to $FEDORA_HOME/solr/collection1

```
mkdir $FEDORA_HOME/solr
cp -r /opt/solr/example/solr/collection1 $FEDORA_HOME/solr/collection1
```

# Update the solr schema.xml

Goto to your solr conf directory and backup schema.xml in case any problems occur.  Then replace schema.xml with the schema.xml file provided in this GitHub repository.

```
cd $FEDORA_HOME/solr/collection1/conf
cp schema.xml schema.xml.bak 
rm -f schema.xml
cp -f /usr/local/fedora/.configuration/schema.xml .
```

~~Use the provided schema file [schema.xml](../blob/modular/conf/schema.xml)~~

### Some potential problems with using the basic-solr-config schema.xml  
*Note: These issues have already been addressed in the copy of schema.xml found in this repository!*

#### Error loading class 'solr.EnglishPorterFilterFactory'

collection1: org.apache.solr.common.SolrException:org.apache.solr.common.SolrException: Plugin init failure for [schema.xml] fieldType "text": Plugin init failure for [schema.xml] analyzer/filter: Error loading class 'solr.EnglishPorterFilterFactory'

Changed the deprecated "EnglishPorterFilterFactory" to use "SnowballPorterFilterFactory".

```
<filter class="solr.EnglishPorterFilterFactory" protected="protwords.txt"/>
```

Becomes

```
<filter class="solr.SnowballPorterFilterFactory" language="English" protected="protwords.txt"/>
```

#### Unable to use updateLog: _version_field

collection1: org.apache.solr.common.SolrException:org.apache.solr.common.SolrException: Unable to use updateLog: _version_field must exist in schema, using indexed="true" stored="true" and multiValued="false" (_version_ is multiValued

Changed the version of the schema from 1.1 to version 1.5 which includes these changes:

```
       1.2: omitTermFreqAndPositions attribute introduced, true by default
            except for text fields.
       1.3: removed optional field compress feature
       1.4: autoGeneratePhraseQueries attribute introduced to drive QueryParser
            behavior when a single string produces multiple tokens.  Defaults
            to off for version >= 1.4
       1.5: omitNorms defaults to true for primitive field types
            (int, float, boolean, string…)
```

Also Added the following field to the schema.

```
<field name="_version_" type="long" indexed="true" stored="true"/> 
```

# Update the solrconfig.xml
Goto to your solr conf directory and backup the existing solrconfig.xml just in case.  
```
cd $FEDORA_HOME/solr/collection1/conf
cp solrconfig.xml solrconfig.xml.bak 
```
Tell solr that we want changes to be flushed and make changes available right away
```
sed -i 's|<openSearcher>false</openSearcher>|<openSearcher>true</openSearcher>|g' solrconfig.xml
```
Increase the size of the cache
``` 
sed -i 's|<queryResultWindowSize>20</queryResultWindowSize>|<queryResultWindowSize>50</queryResultWindowSize>|g' solrconfig.xml
```
Enable backwards compatibility
```
sed -i 's|<requestDispatcher handleSelect="false" >|<requestDispatcher handleSelect="true" >|g' solrconfig.xml
```
Update the /select requestHandler to tell solr about some of the fun things we want to index - makes islandora basic solr search work. The following commands will add some lines to your config file.
```
sed -i '782i<str name="fl">*</str>' solrconfig.xml
sed -i '783i<str name="q.alt">*:*</str>' solrconfig.xml
sed -i '784i<str name="qf">dc.title^5 dc.subject^3 dc.description^3 dc.creator^3 dc.contributor^3 dc.type^1 dc.relation^1 dc.publisher^1 mods_identifier_local_ms^3 ds.WARC_FILTER^1 text_nodes_HOCR_hlt^1 mods_subject_hierarchicalGeographic_region_ms^3 mods_identifier_hdl_mt^3 dc.identifier^3 PID^0.5 catch_all_fields_mt^0.1</str>' solrconfig.xml
```

# Change Solr File Ownership to Avoid File Permissions Issues During Restart
```
chown -R fedora:fedora /usr/local/fedora/solr
chown -R fedora:fedora /usr/local/fedora .out-of-the-way
chown -R fedora:fedora /usr/local/fedora .configuration
```

# Start Tomcat and Check for Errors
```
/usr/sbin/logrotate /etc/logrotate.conf
service tomcat start
tail -400 /usr/local/fedora/tomcat/logs/catalina.out
```

# Install GSearch 2.7

Place the GSearch 2.7 [fedoragsearch.war](http://downloads.sourceforge.net/fedora-commons/fedoragsearch-2.7.zip) file in $FEDORA_HOME/tomcat/webapps.  Unzip it, rename the resulting folder, and reset file ownership.  Then restart Tomcat to expand the fedoragsearch.war file and check catalina.out for errors.
```
cd $FEDORA_HOME/tomcat/webapps
wget http://downloads.sourceforge.net/fedora-commons/fedoragsearch-2.7.zip
unzip fedoragsearch-2.7.zip
mv -f fedoragsearch-2.7/fedoragsearch.war .
rm -fr fedoragsearch-2.7*
chown -R fedora:fedora fedoragsearch*
rm -f /usr/local/fedora/tomcat/logs/catalina.out
service tomcat restart
tail -400 /usr/local/fedora/tomcat/logs/catalina.out
```
Navigate to $FEDORA_HOME/tomcat/webapps/fedoragsearch/FgsConfig and backup the basic properties file, for reference in case any problems occur.  Copy the contents of the *configDemoOnSolr* directory to a new directory named *configDGOnSolr* and change file ownership to avoid file permissions problems.

```
cd $FEDORA_HOME/tomcat/webapps/fedoragsearch/FgsConfig
cp fgsconfig-basic.properties fgsconfig-basic.properties.bak 
cp -fr configDemoOnSolr/ configDGOnSolr/
chown -R fedora:fedora *
```

Modify the basic properties file (fgsconfig-basic.properties) as needed be sure that the following properties are set correctly:
__&lt;path to fedora home&gt;__ should be the absolute path to your $FEDORA_HOME directory typically __/usr/local/fedora__.

```
configDisplayName=configDGOnSolr
local.FEDORA_HOME=/usr/local/fedora 
indexEngine=Solr
indexDir=${local.FEDORA_HOME}/solr/collection1/data/index
indexBase=http://localhost:8080/solr 
indexingDocXslt=foxmlToSolr    
```

Run the following commands: 

```
ant generateIndexingXslt
ant -f fgsconfig-basic.xml
```

Ensure that you have added the [DGI GSearch Extensions](https://github.com/discoverygarden/dgi_gsearch_extensions) to GSearch, just follow the directions provided at the preceding link.  Use the following commands as directed (sort of) in the linked repository...
```
cd /usr/local/fedora
git clone https://github.com/discoverygarden/dgi_gsearch_extensions.git
cd dgi_gsearch_extensions
mvn package
cp target/gsearch_extensions-*-jar-with-dependencies.jar $CATALINA_HOME/webapps/fedoragsearch/WEB-INF/lib
```
Clone the *basic-solr-config* repository and copy *foxmlToSolr.xslt* to your *$FEDORA_HOME/tomcat/webapps/fedoragsearch/WEB-INF/classes/fgsconfigFinal/index/FgsIndex* directory.  Repeat for the *islandora_transforms* repository and resulting *islandora_transforms* directory.
```
cd /usr/local/fedora
git clone https://github.com/discoverygarden/basic-solr-config.git
git clone https://github.com/discoverygarden/islandora_transforms.git
cd basic-solr-config
cp -f basic-solr-config/foxmlToSolr.xslt $FEDORA_HOME/tomcat/webapps/fedoragsearch/WEB-INF/classes/fgsconfigFinal/index/FgsIndex
cp -fr islandora_transforms/ $FEDORA_HOME/tomcat/webapps/fedoragsearch/WEB-INF/classes/fgsconfigFinal/index/FgsIndex
cd $FEDORA_HOME/tomcat/webapps/fedoragsearch/WEB-INF/classes/fgsconfigFinal/index/FgsIndex
chown -R fedora:fedora *
```

Uncomment the resolver in index.properties

```
fgsindex.uriResolver    = dk.defxws.fedoragsearch.server.URIResolverImpl
```

If you keep Fedora or Tomcat in the non-default location, you will need to update each of the *.xslt files to reflect that as well.

# Add Oral Histories XSLT
The Islandora Oral Histories Solution Pack reqiures an XSLT be added to the *islandora_transforms* directory (that is /usr/local/fedora/tomcat/webapps/fedoragsearch/WEB-INF/classes/fgsconfigFinal/index/FgsIndex/islandora_transforms).  To facilitate this follow the instructions at https://github.com/digitalutsc/islandora_solution_pack_oralhistories/wiki/Configuration:--Basic-Indexing-of-transcripts-in-Solr.  For example...

```
cd /usr/local/fedora/
git clone https://github.com/digitalutsc/islandora_solution_pack_oralhistories.git
cp islandora_solution_pack_oralhistories/xsl/or_transcript_solr.xslt /usr/local/fedora/tomcat/webapps/fedoragsearch/WEB-INF/classes/fgsconfigFinal/index/FgsIndex/islandora_transforms
cp islandora_solution_pack_oralhistories/xsl/vtt_solr.xslt /usr/local/fedora/tomcat/webapps/fedoragsearch/WEB-INF/classes/fgsconfigFinal/index/FgsIndex/islandora_transforms
rm -fr /usr/local/fedora/islandora_solution_pack_oralhistories
chown -R fedora:fedora /usr/local/fedora/tomcat/webapps/fedoragsearch/WEB-INF/classes/fgsconfigFinal/index/FgsIndex/islandora_transforms
```
The remaining steps 2 through 4 from the aforementioned document (https://github.com/digitalutsc/islandora_solution_pack_oralhistories/wiki/Configuration:--Basic-Indexing-of-transcripts-in-Solr) have already been taken care of in the *foxmlToSolr.xslt* and *schema.xml* files in this repository.

Restart Tomcat and Check for Errors
```
service tomcat stop
/usr/sbin/logrotate /etc/logrotate.conf
service tomcat start
tail -400 /usr/local/fedora/tomcat/logs/catalina.out
```

# Clean-up
```
cd /usr/local/fedora
rm -fr basic-solr-config/
rm -fr dgi_gsearch_extensions/
rm -fr islandora_transforms/
rm -fr .out-of-the-way/
```

# FAQ

## Couldn't Access http://<sitename>:8080/fedoragsearch/solr

/usr/local/fedora/tomcat/webapps/fedoragsearch/WEB-INF/classes/fgsconfigFinal/index/FgsIndex/foxmlToSolr.xslt was not present and it is the "default" solr xslt now in gsearch 2.7.

Please read over sort out this issue see [Install GSearch](#install-gsearch-26)

```
cd $FEDORA_HOME/tomcat/webapps/fedoragsearch/FgsConfig 
ant generateIndexingXslt
ant -f fgsconfig-basic.xml
```

## When visiting http://<sitename>:8080/fedoragsearch/rest?operation=updateIndex the following error appears.

Thu Oct 03 13:33:09 UTC 2013 IndexReader open error indexName=FgsIndex : ; nested exception is: org.apache.lucene.store.NoSuchDirectoryException: directory '/usr/local/fedora/solr/data/index' does not exist


Check the basic properties file as needed be sure that:

```
indexDir=${local.FEDORA_HOME}/solr/collection1/data/index 
```
