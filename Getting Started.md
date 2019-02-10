# empow logstash classification plugin & ELK module 
The ELK stack allows you to ingest log data from many sources, parse and manipulate it, store it in, analyze it, and visualize it. The stack consists of three components, Logstash, for data ingestion and manipulation, Elasticsearch, for storage and analysis of data,and Kibana, to visualize your data.

The empow classification plugin extends the functionality of logstash by classifying your log data, using the empow classification center, for attack intent and attack stage. 

The empow module has preconfigured configurations for the entire ELK stack, that you can use 'out-of-the-box' to ingest, store, and visualize log data from your network devices.




# Supported platforms

The plugin and module will run on any platform on which the ELK stack is supported.

We will use Ubuntu 18.04 as the reference platform for this note.


# What you will need

To get started, you will need to install these components:

Java 8 

Logstash


After this, you can add the other elements of the stack, and the empow module:

Elasticsearch

Kibana

empow module

<hr>

# Get started with logstash and the empow plugin

The procedures below use _dpkg_ on Ubuntu. Other methods to acquire and install packages can also be used (we have references to relevant sites for packages and methods)


## Java 

_Check if Java is installed_


Java must be installed before installing logstash. Run this commmand to check if Java is installed, and which version. If an error is returned, or the version is not Java 8, follow the steps to install Java.

```java -version```

### Install Java 8


```sudo apt install openjdk-8-jdk```

Confirm the installation

```java -version```

This should return something like this:

```
openjdk version "1.8.0_162"
OpenJDK Runtime Environment (build 1.8.0_162-8u162-b12-1-b12)
OpenJDK 64-Bit Server VM (build 25.162-b12, mixed mode)
```

<hr>


## Logstash

We will install the Debian version ```6.5.4```. If there are later versions available, you can use them as well. Check this [page](https://www.elastic.co/downloads/logstash) for the link for this package (and other download options). 

### Install logstash

```
wget https://artifacts.elastic.co/downloads/logstash/logstash-6.5.4.deb 
sudo dpkg -i logstash-6.5.4.deb
````



### Configure logstash

Configure the logstash.yml file (```/etc/logstash/logstash.yml```)

Disable automatic config file loading:
```config.reload.automatic: false```

Add the following lines:
```
modules:
- name: empow
  var.elasticsearch.ssl.enabled: false
  var.kibana.ssl.enabled: false
  var.classification.username: "******"
  var.classification.pass: "******"
```

Restart Logstash


```sudo service logstash restart```

Check the service is running with this:

```sudo service logstash status```
<hr>

## empow classification plugin
The empow classification plugin classifies attack log information to show attack intent and attack stage. It uses the cloud-based emplow classification center.

### Install the plugin


```sudo /usr/share/logstash/bin/logstash-plugin install logstash-filter-empowclassifier```

Install additional plugins :

```
sudo /usr/share/logstash/bin/logstash-plugin install  logstash-filter-uuid logstash-filter-translate logstash-filter-prune
```


  Restart the logstash service:

```sudo service logstash restart```


## Create a logstash pipeline (example)

This example illustrates how to create a full logstash pipeline in the ```/etc/logstash/conf.d/``` directory to receive logs from a source, filter them, and send them to an output destination. It will also use the empow plugin to classify the attack.



A logstash [pipeline](https://www.elastic.co/guide/en/logstash/current/pipeline.html) consists of three main sections:

_input_ - this determine the way logs are read by logstash.

_filter_ - a set of configuration processing and manipulation of the logs, used to change its structure, extrants, add, remove, and process, fields within the logs

_output_ - determining the destination of the processed logs

Consider, for example, a snort IDS (Intrusion Detection System). As part of the configuration of the IDS, one should direct its logs to logstash. Let's assume that these logs are sent using UDP protocol to port number 2055 (the destination address, the protocol and the port number are part of the snort configuration).
In this case, configure the input section of the logstash pipeline like this:
```   
input{
  udp{
    port => 2055
  }
}
```
This configures logstash to receive log sources on UDP port number 2055.

When logs enter  the pipeline they are proceessed using filters. A typical default snort log structure has the following structure:

```
<NUMBER>MONTH DAY HOUR:MINUTE:SECOND snort[NUMBER]: [SIGNATURE_ID] DESCRIPTION [Priority: 3] {PROTOCOL} SRC_IP:SRC_PORT_NUMBER -> DST_IP:DST_PORT_NUMBER
```

Some of the field are optional. Here is an example:
```
<33>Jan 21 15:10:37 snort[11934]: [1:1234:1] (http_inspect) NO CONTENT-LENGTH OR TRANSFER-ENCODING IN HTTP RESPONSE [Classification: Unknown Traffic] [Priority: 3] {TCP} 192.168.21.120:80 -> 192.168.22.135:62508 
 ```

Logstash has many methods and options to process logs, and in this case we will use the logstash [grok](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html ) plugin, which parses unstructured event data into fields similar to regular expression engines.

This filter configuration extracts the following fields from the snort log using grok: 
```
filter{
  grok{
    match => {"message" => "%{NUMBER}\>%{SYSLOGTIMESTAMP:date} snort\[%{NUMBER}\]: \[(?<sig_id>%{NUMBER}:%{NUMBER}):%{NUMBER}\] .\* %{IP:src_addr}(:%{NUMBER})? -> %{IP:dst_addr}(:%{NUMBER})?"}
  }
}
```
where:

_date_ - the log date

_sig_id_ - the threat signature id (in the format NUMBER:NUMBER, where the third number is omitted)

_src_addr_ - the source IP address

_dst_addr_ - the destination IP address


In order to use the empow classification plugin, we add the following fields:
_product_type_ (set to IDS in this example)
_product_name_ (set to snort in this example)
_threat_ - a JSON structure with the  threat information (signature_id for IDS, or hash and malware name for Anti Virus or Anti Malware)

We will also use the logstash [mutate](https://www.elastic.co/guide/en/logstash/current/plugins-filters-mutate.html) and [grok](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html) plugins, as follows:

```
filter{
  grok{
    match => {"message" => "%{NUMBER}\>%{SYSLOGTIMESTAMP:date} snort\[%{NUMBER}\]: \[(?<sig_id>%{NUMBER}:%{NUMBER}):%{NUMBER}\] .\* %{IP:src_addr}(:%{NUMBER})? -> %{IP:dst_addr}(:%{NUMBER})?"}
  }
  mutate{
     add_field => {"product_type" => "IDS" "product_name" => "snort"}
     add_field => {"[threat][signature]" => "%{sig_id}"}     
  }
}
```


Now, we will add the empowclassier plugin to the pipeline, and set the classification center username and password (link to the registration page). Note, the empow classification plugin has many other configuration options

```
filter{
  grok{
    match => {"message" => "%{NUMBER}\>%{SYSLOGTIMESTAMP:date} snort\[%{NUMBER}\]: \[(?<sig_id>%{NUMBER}:%{NUMBER}):%{NUMBER}\] .\* %{IP:src_addr}(:%{NUMBER})? -> %{IP:dst_addr}(:%{NUMBER})?"}
  }

  mutate{
     add_field => {"product_type" => "IDS" "product_name" => "snort"}
     add_field => {"[threat][signature]" => "%{sig_id}"}     
  }
  empowclassifier {
     username => "*******" # replace with your username
     password => "*******"  # replace with your password
  }
}
```

Using this pipeline, the empowclassifier plugin will generate new fields containing the classification results from the snort logs (or error fields)

A valid response will contain a JSON ,```empow_classification_response```, that includes an ```intents``` field which includes, containing other things, the tactics and the attack stage of the attack (link to the page explaining this terms).


To complete the pipeline configuration, we will setup the [output](https://www.elastic.co/guide/en/logstash/current/pipeline.html#_outputs) section. This determines where the filtered logs will be sent. Logs can be sent to many destinations, including databases, elastic nodes, etc.
In our example we will send the log by udp to localhost port 1237. The output section is then:

```
output{
  udp{
   host => "127.0.0.1"
   port => 1237
  }
}
```

The entire pipline configuration file now looks like this:

```
input{
  udp{
    port => 2055
  }
}

filter{
  grok{
    match => {"message" => "%{NUMBER}\>%{SYSLOGTIMESTAMP:date} snort\[%{NUMBER}\]: \[(?<sig_id>%{NUMBER}:%{NUMBER}):%{NUMBER}\] .\* %{IP:src_addr}(:%{NUMBER})? -> %{IP:dst_addr}(:%{NUMBER})?"}
  }

  mutate{
     add_field => {"product_type" => "IDS" "product_name" => "snort"}
     add_field => {"[threat][signature]" => "%{sig_id}"}     
  }
  empowclassifier {
     username => "*******"  # replace with your username
     password => "*******"  # replace with your password
  }
}

output{
  udp{
   host => "127.0.0.1"
   port => 1237
  }
}
```


This file has the three components of a logstash config file: input & output sections, and a section defining the filtering actions. In this file, the filter refers to the _empowclassifier_ (the plugin), which in turn accesses the empow cloud-based classification system. The input listens on UDP port 2055, and the output is to port 1237 on the localhost.


Add this line to the ```logstash.yml``` file, to enable automatic reload of new configuration files when they are put in the logstash configuration folder (```/etc/logstash/conf.d/```):

``` 
config.reload.automatic: true
```

Copy the pipeline  file (rename it *snort_empow_example.conf*, say) to the logstash config file folder ```/etc/logstash/conf.d/```, and wait a few seconds for logstash to it.

```sudo cp empow_example.conf  /etc/logstash/conf.d/```

 If it doesn't load, restart logstash (all config files are loaded when logstash starts):

```sudo service logstash restart```

To test how logstash processes log strings using this example config file, open two terminal sessions, one to listen for logstash output, and one to send a log string to logstash:

In the first, enter:

```nc -luk 1237```

In the second, enter:

```nc -u 127.0.0.1 2055```

Then (still in the second console) enter:

```
1:1237<33>Jan 21 15:10:37 snort[11934]: [1:1234:1] (http_inspect) NO CONTENT-LENGTH OR TRANSFER-ENCODING IN HTTP RESPONSE [Classification: Unknown Traffic] [Priority: 3] {TCP} 192.168.21.120:80 -> 192.168.22.135:62508
```

In the first (the listener), the following should appear (after few seconds):

```
{"sig_id":"1:1234","tags":["dummy"],"empow_warnings":["src_internal_wrong_value","dst_internal_wrong_value"],"empow_classification_response":{"intents":[{"tactic":"Full compromise - active patterns","isSrcPerformer":true,"attackStage":"Infiltration"}]},"threat":{"signature":"1:1234"}}
```
This is the logstash output data block for the input string, filtered according to the config file, and with the empow classification fields included, obtained using plugin.

### Register with empow to use the plugin
<!--- add this later...
-->



## Elasticsearch

### Install elasticsearch

```wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.6.0.deb```

```sudo dpkg -i elasticsearch-6.6.0.deb```


### Start elasticsearch

```sudo service  elasticsearch  start```


#### Check elasticsearch is running

sudo service status elasticsearch

Run the following curl command

```curl -X GET "localhost:9200/"```

You should see something like this returned:
<details>

```
{
  "name" : "Cp8oag6",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "AT69_T_DTp-1qgIJlatQqA",
  "version" : {
    "number" : "6.5.4",
    "build_flavor" : "default",
    "build_type" : "zip",
    "build_hash" : "f27399d",
    "build_date" : "2016-03-30T09:51:41.449Z",
    "build_snapshot" : false,
    "lucene_version" : "7.5.0",
    "minimum_wire_compatibility_version" : "1.2.3",
    "minimum_index_compatibility_version" : "1.2.3"
  },
  "tagline" : "You Know, for Search"
}
```
</details>


## Kibana

### Install Kibana

```
wget https://artifacts.elastic.co/downloads/kibana/kibana-6.6.0-amd64.deb
sudo dpkg -i kibana-6.6.0-amd64.deb
```

### Configure kibana
Add the following line to the kibana configuration file (```/etc/kibana/kibana.yml``` )  to enable remote access to kibana:

```server.host: "0.0.0.0"```



Note: if an the entry for ```server.host``` alreay exists, change it  to ```"0.0.0.0"```




### Start the Kibana as a service

```sudo service  kibana  start```


#### Test that Kibana is running

```sudo service kibana status```

Open the following URL in a browser


```http://localhost:5601```


The Kibana home page should appear.


## empow module

### Download the empow module
Download and install the empow module (empow_classification_module.tar)

```
cd /usr/share/logstash/modules
sudo wget ".../empow_classification_module.tar
sudo tar xvf empow_classification_module.tar
```
