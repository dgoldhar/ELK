# empow module for Elasticsearch stack
The Elasticsearch stack allows you to ingest log data from many sources, parse and manipulate it, store it in, analyze it, and visualize it. The stack consists of three components, Logstash, for data ingestion and manipulation, Elasticsearch, for storage and analysis of data,and Kibana, to visualize your data.

The empow module extends the capability of this stack at the ingestion stage, by classifying your log data, using the empow classification center, for attack intent and stage. This is added to the log data, and stored with it. You can then use the other elements of the stack to more effectively analyse and visualize your attack log data.


# Supported platforms

  - Ubuntu 18
 
 
# What you will need

Java 8 


Logstash

Elasticsearch

Kibana

empow module

# Install components

We will use apt-get to download & install the components on Ubuntu 18.04, but other methods can also be used (references to the Elasticsearch site are included).

Download the GPG key for the elasticsearch components

```wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -```

(Note: the flag is q*O*, not q0)

```sudo apt-get install apt-transport-https```

```echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list```


## Java 

Check if Java is installed

Run this commmand to check if Java is installed, and which version. If an error is returned, or the version is not Java 8, follow the steps to install Java.

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

## Logstash

### Install logstash

```sudo apt-get update && sudo apt-get install logstash```

### Configure logstash as a service

```sudo systemctl start logstash.service```

### Configure logstash

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


## Elasticsearch

### Install elasticsearch

```sudo apt-get update && sudo apt-get install elasticsearch```


### Configure elasticsearch as a service
```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
```
### Start elasticsearch

```sudo systemctl start elasticsearch.service```


#### Check elasticsearch is running

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


#### Test that Kibana is running in a browser


```http://localhost:5601```


## empow module

# Configuration

