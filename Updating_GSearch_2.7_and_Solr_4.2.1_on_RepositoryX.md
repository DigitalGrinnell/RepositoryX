# Install Solr 4.2.0

Stop Fedora

Download [Solr 4.2.0](http://archive.apache.org/dist/lucene/solr/4.2.0/solr-4.2.0.tgz) to /opt 

Make a symbolic link to the solr directory such that /opt/solr => /opt/solr-4.2.0

```
ln -s /opt/solr-4.2.0 /opt/solr
```

Add the following XML file to $FEDORA_HOME/tomcat/conf/Catalina/localhost/solr.xml

```XML
<?xml version="1.0" encoding="UTF-8"?>
<Context docBase="/opt/solr/dist/solr-4.2.0.war" debug="0" crossContext="true">
  <Environment name="solr/home" type="java.lang.String" value="/usr/local/fedora/solr" override="true"/>
</Context>
```

Copy over the default configuration files for "collection1" from /opt/solr/example/solr/collection1 to $FEDORA_HOME/solr/collection1

```
cp -r /opt/solr/example/solr/collection1 $FEDORA_HOME/solr/collection1
```

# Update the solr schema.xml

Goto to your solr conf directory.

```
cd $FEDORA_HOME/solr/collection1/conf
```

First backup the existing Schema, for reference incase any problems occur.

```
cp schema.xml schema.xml.bak 
```

Use the provided schema file [schema.xml](../blob/modular/conf/schema.xml)

### Some potential problems with using the basic-solr-config schema.xml

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
Goto to your solr conf directory.

```
cd $FEDORA_HOME/solr/collection1/conf
```
First backup the existing Schema, for reference incase any problems occur.

```
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
# Install GSearch 2.7

Start Fedora

Place the GSearch 2.7 [fedoragsearch.war](http://downloads.sourceforge.net/fedora-commons/fedoragsearch-2.7.zip) file in $FEDORA_HOME/tomcat/webapps

Navigate to $FEDORA_HOME/tomcat/webapps/fedoragsearch/FgsConfig.

```
cd $FEDORA_HOME/tomcat/webapps/fedoragsearch/FgsConfig
```

Backup the basic properties file, for reference incase any problems occur.

```
cp fgsconfig-basic.properties fgsconfig-basic.properties.bak 
```

Modify the basic properties file as needed be sure that the following properties are set correctly:
__&lt;path to fedora home&gt;__ should be the absolute path to your $FEDORA_HOME directory typically __/usr/local/fedora__.

```
configDisplayName=configDemoOnSolr
local.FEDORA_HOME=<path to fedora home> 
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

Ensure that you have added the [DGI GSearch Extensions](https://github.com/discoverygarden/dgi_gsearch_extensions) to GSearch, just follow the directions provided at the preceding link.

Go to the index directory of the GSearch

```
cd  $FEDORA_HOME/tomcat/webapps/fedoragsearch/WEB-INF/classes/fgsconfigFinal/index/FgsIndex
```

Download [foxmlToSolr.xslt](../blob/modular/foxmlToSolr.xslt) and [islandora_transforms](../blob/modular/islandora_transforms) to your current directory.

Uncomment the resolver in index.properties

```
fgsindex.uriResolver    = dk.defxws.fedoragsearch.server.URIResolverImpl
```

If you keep Fedora or Tomcat in the non-default location, you will need to update each of the *.xslt files to reflect that as well.

Restart Fedora


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
