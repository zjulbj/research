#Environments
Environments variable for `JAVA_OPTS` to start JVM.It is recommended to set the -Xmx -Xms with the same value `ES_HEAP_SIZE`. 
```
ES_HEAP_SIZE = 2g
```

# System configuration
## File descriptors
ElasticSearch process will open lots of files,so make sure to increase the open files descriptors on the machine.
We can check node status by rest api as follow:
```
[admin@iZ944dlw6hkZ ~]$ curl localhost:9200/_nodes/stats/process?pretty
{
  "cluster_name" : "elasticsearch",
  "nodes" : {
    "jOwwmZPGQdeTZTKwFanT3A" : {
      "timestamp" : 1471531918277,
      "name" : "D'Spayre",
      "transport_address" : "127.0.0.1:9300",
      "host" : "127.0.0.1",
      "ip" : [ "127.0.0.1:9300", "NONE" ],
      "process" : {
        "timestamp" : 1471531918278,
        "open_file_descriptors" : 146,
        "max_file_descriptors" : 65535,
        "cpu" : {
          "percent" : 0,
          "total_in_millis" : 178780
        },
        "mem" : {
          "total_virtual_in_bytes" : 4797259776
        }
      }
    }
  }
}
```
## Virtual memory
ElasticSearch use a hybrid mmapfs / niofs directory by default to store its indices.So we should increase the limits on mmap counts which by default is to low. 

```
## temporarily
sudo -w vm.max_map_count=262144
```
To set this value permanently, update the vm.max_map_count setting in /etc/sysctl.conf


#ElasticSearch Configuration

Ok,now we go a little more deeper about the basic configuration of ElasticSearch.
Configuration files of ElasticSearch are stored in `${ES_INSTALL_DIR}/config`.
There are two `*.yml` files:`elasticsearch.yml` and `logging.yml`(BTW,I think YML is quite a simple but powerful way to 
represent configuration).ElasticSearch comes with reasonable defaults for most settings which makes it easy to start.
Before we start to tweet and tune the configuration,we should understand what are we doing and the consequences.
The most important file I guess is `elasticsearch.yml`.let's see some important settings:

##Cluster and node,paths
As we know ,ElasticSearch is a distributed system.
We should pick a cool name for our cluster and node instead of the default random names.

```
# Name for our cluster,make sure that you don't reuse the same names in different environments
# For example,if we use `es-syd-pro`(a cluster in production) in a developpment server,that will make big mistake.
cluster:
    name: es-syd-dev
    
# Use a unique name for this node.By default ElasticSearch will randomly pick a name from a list of about 3000 names when 
# node starts up. If we only run one node for that cluster,wh could name the node with hostname.The hostname of the machine 
# is provided in the environment variable HOSTNAME,which is quite cool.
# Name is a identity of a node,make sure keep it unique.
node:
    name: ${HOSTNAME}
    
# Path to directory where to store the data(separate mutilple locations by comma)
# In production use , we should better change paths for data and log files to a directory which mounted on a SSD hardware.

path:
    logs: /mnt/elasticsearch/logs
    data: /mnt/elasticsearch/data
```

## 

