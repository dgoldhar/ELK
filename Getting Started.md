# empow logstash classification plugin & ELK module 
The Elasticsearch stack allows you to ingest log data from many sources, parse and manipulate it, store it in, analyze it, and visualize it. The stack consists of three components, Logstash, for data ingestion and manipulation, Elasticsearch, for storage and analysis of data,and Kibana, to visualize your data.

The empow classification plugin extends the functionality of logstash by classifying your log data, using the empow classification center, for attack intent and stage. 

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


<!--- don't need this:
Download the GPG key for the elasticsearch components

```wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -```

(Note: the flag is q*O*, not q0)

```sudo apt-get install apt-transport-https```

```echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list```

-->


## Java 

_Check if Java is installed_


Java must be installed before installing logstash. Run this commmand to check if Java is installed, and which version. If an error is returned, or the version is not Java 8, follow the steps to install Java.

```java -version```

### Install Java 8


```sudo apt install openjdk-8-jdk```

Confirm the installation
```java -version```

This should return something like this:

```openjdk version "1.8.0_162"
OpenJDK Runtime Environment (build 1.8.0_162-8u162-b12-1-b12)
OpenJDK 64-Bit Server VM (build 25.162-b12, mixed mode)```
```
<hr>


## Logstash
su
### Install logstash

```wget https://artifacts.elastic.co/downloads/logstash/logstash-6.5.4.deb ```

```sudo dpkg -i logstash-6.5.4.deb```

Start the logstash service

```su```

Check the service is running with this:

```sudo service logstash status```

## Install the empow classification plugin

sudo /usr/share/logstash/bin/logstash-plugin install <plugin name>


Now, you can start using logstash....

Example logstash conf file 

This file uses the empow plugin.

```
input{
  udp{
    port => 2055
  }
}

filter{
  grok{
    match => { "message" => "(?<sig_id>[0-9]:[0-9]+)"}
  }

  mutate{
    add_field => {"product_type" => "IDS"}
    add_field => {"product_name" => "snort"}
    rename => {"sig_id" => "[term][signature]"}
    add_field => {"is_src_internal" => 1}
    add_field => {"is_dst_internal" => 0}
  }

  empowclassifier {
     classification_url => "https://s0apxz9wik.execute-api.us-east-2.amazonaws.com"
     classification_username => "assaf"
     classification_password => "Empow2018!"
  }


}

output{
  udp{
   host => "127.0.0.1"
   port => 1237
  }
}
```

Copy this file to the config file folder & restart logstash


```sudo cp empow_example.conf  /etc/logstash/conf.d/```

restart logstash

```sudo service logstash restart```

Open two terminal sessions, one to listen for logstash output, and one to send a log string to logstash:

In the first:

```nc -luk 1237```

In the second:

```nc -u 127.0.0.1 2055```

Then enter:

```1:1234```

In the first (the listener), the following should appear:



```
{"is_dst_internal":"0","message":"1:1234\n","product_name":"snort","@timestamp":"2019-01-16T14:37:56.889Z","term":{"signature":"1:1234"},"@version":"1","product_type":"IDS","is_src_internal":"1","tags":["_src_internal_wrong_value","_dst_internal_wrong_value"],"empow_intents":[{"attackStage":"Infiltration","isSrcPerformer":true,"tactic":"Full compromise - active patterns"}],"host":"127.0.0.1"}
```



### register with empow to use plugin 
<!--- add this later...
-->




### Configure logstash as a service (not sure if this is needed)

<!--- replace this
```sudo systemctl start logstash.service```
-->

sudo service start logstash (for ubuntu)
<hr>

## Configure logstash

<!--
--  pipeline config
-- execution config
-->


Logstash uses a configuration file (e.g., logstash.config) to tell it where to listen for log records (a port), how to parse the log info once received, and what to store in elasticsearch

The config files are in ```/etc/logstash/conf.d``` and have extension ```.conf```,  e.g., ```logstash.conf```.

To test logstash, edit a config file, ```logstash-test.conf``` and add the following:
```
input {
udp{ port => 2055 }
}
filter {
grok {match => {"message" => "%{IP:src_ip} %{IP:dst_ip}"}}
}
output {
udp{ 
host => "127.0.0.1"
port => 1237
}
}
```
Run logstash with this config file

``` 
bin/logstash -f logstash-test.conf
```

Send the following text to port 2055:

```"10.0.0.1 1.2.3.4"```


<hr>

# Elasticsearch & Kibana


## Elasticsearch

### Install elasticsearch

wget ...
sudo dpkg ...

<!---
```sudo apt-get update && sudo apt-get install elasticsearch```
-->


### Configure elasticsearch as a service
```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
```
sudo service start elasticsearch


### Start elasticsearch

```sudo systemctl start elasticsearch.service```


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

```sudo apt-get update && sudo apt-get install kibana```

### Configure Kibana as a service
```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable kibana.service
```


### Start the Kibana as a service

```sudo systemctl start kibana.service```


#### Test that Kibana is running

Open the following URL in a browser


```http://localhost:5601```


The Kibana home page should appear.


## empow module

Download the empow module

Augment your logstash config file to use the empow module

Test log data for the empow module


# Configuration

