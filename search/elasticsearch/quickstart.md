#Install and setup
- 1.Download package
```
wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.3.4/elasticsearch-2.3.4.tar.gz
tar -zxvf elasticsearch-2.3.4.tar.gz
```
- 2.Start the process
```
cd elasticsearch-2.3.4
bin/elasticsearch -d -p pid
```
`-d` means run as a daemon process and `pid` to save the process id.

- 3.Stop it
```
kill `cat pid`
```
Kill the process softly.

- 4.Check the logs
By default the log files are stored in `${ElasticSearchInstallDir}/logs`,and there would be a series of logs as below:
```
[admin@iZ944dlw6hkZ logs]$ ll
total 4
-rw-rw-r-- 1 admin admin    0 Aug 18 02:27 elasticsearch_deprecation.log
-rw-rw-r-- 1 admin admin    0 Aug 18 02:27 elasticsearch_index_indexing_slowlog.log
-rw-rw-r-- 1 admin admin    0 Aug 18 02:27 elasticsearch_index_search_slowlog.log
-rw-rw-r-- 1 admin admin 1834 Aug 18 02:27 elasticsearch.log
```
Let's see what `elasticsearch.log` prints.
```
[2016-08-18 02:27:38,776][INFO ][node                     ] [D'Spayre] version[2.3.4], pid[14246], build
[e455fd0/2016-06-30T11:24:31Z]
[2016-08-18 02:27:38,776][INFO ][node                     ] [D'Spayre] initializing ...
[2016-08-18 02:27:39,305][INFO ][plugins                  ] [D'Spayre] modules [reindex, lang-expression
, lang-groovy], plugins [], sites []
[2016-08-18 02:27:39,335][INFO ][env                      ] [D'Spayre] using [1] data paths, mounts [[/
(rootfs)]], net usable_space [109.5gb], net total_space [117.9gb], spins? [unknown], types [rootfs]
[2016-08-18 02:27:39,335][INFO ][env                      ] [D'Spayre] heap size [990.7mb], compressed o
rdinary object pointers [true]
[2016-08-18 02:27:39,335][WARN ][env                      ] [D'Spayre] max file descriptors [65535] for
elasticsearch process likely too low, consider increasing to at least [65536]
[2016-08-18 02:27:41,034][INFO ][node                     ] [D'Spayre] initialized
[2016-08-18 02:27:41,035][INFO ][node                     ] [D'Spayre] starting ...
[2016-08-18 02:27:41,097][INFO ][transport                ] [D'Spayre] publish_address {127.0.0.1:9300},
 bound_addresses {127.0.0.1:9300}
[2016-08-18 02:27:41,103][INFO ][discovery                ] [D'Spayre] elasticsearch/jOwwmZPGQdeTZTKwFan
T3A
[2016-08-18 02:27:44,140][INFO ][cluster.service          ] [D'Spayre] new_master {D'Spayre}{jOwwmZPGQde
TZTKwFanT3A}{127.0.0.1}{127.0.0.1:9300}, reason: zen-disco-join(elected_as_master, [0] joins received)
[2016-08-18 02:27:44,153][INFO ][http                     ] [D'Spayre] publish_address {127.0.0.1:9200},
 bound_addresses {127.0.0.1:9200}
[2016-08-18 02:27:44,153][INFO ][node                     ] [D'Spayre] started
[2016-08-18 02:27:44,198][INFO ][gateway                  ] [D'Spayre] recovered [0] indices into cluste
r_state
```
Without going too much into detail,we can see that a node name `D'Spayre` has started and elected itself as a master node in a
single cluster.And also port `9200` has been bound to `127.0.0.1` with a http protocol service.
And visit the port,we see:
```
[admin@iZ944dlw6hkZ ~]$ curl localhost:9200?pretty
{
  "name" : "D'Spayre",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "2.3.4",
    "build_hash" : "e455fd0c13dceca8dbbdbb1665d068ae55dabe3f",
    "build_timestamp" : "2016-06-30T11:24:31Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.0"
  },
  "tagline" : "You Know, for Search"
}
```
It seems to be a welcome message for elasticsearch.

Ok,now we just start a elasticsearch cluster in less than 5 minutes!
How simple it is and it evokes my great interest. Simple but powerful stuffs are all my love.
