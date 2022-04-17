# Docker multi-container environment with Hadoop, Spark and Hive

This is it: a Docker multi-container environment with Hadoop 3.2 (HDFS), Spark 2.4.5 and Hive. But without the large memory requirements of a Cloudera sandbox.

The services composing the environment are:

- spark-worker-1
- hive-server
- spark-master
- datanode
- nodemanager
- historyserver
- presto-coordinator
- hive-metastore-postgresql
- namenode
- resourcemanager
- hive-metastore


## Quick Start

To deploy an the HDFS-Spark-Hive cluster, run:
```
  docker-compose up -d --build
```
The `--build` parameter is needed as there are some services with slightly customized Dockerfiles to fix missing features
/add another ones (like persistence in the HDFS space after across restarts)

`docker-compose` creates a docker network that can be found by running `docker network list`, e.g. `docker-hadoop-spark-hive_default`.

Run `docker network inspect` on the network (e.g. `docker-hadoop-spark-hive_default`) to find the IP the hadoop interfaces are published on. Access these interfaces with the following URLs:

* Namenode: http://<dockerhadoop_IP_address>:9870/dfshealth.html#tab-overview
* History server: http://<dockerhadoop_IP_address>:8188/applicationhistory
* Datanode: http://<dockerhadoop_IP_address>:9864/
* Nodemanager: http://<dockerhadoop_IP_address>:8042/node
* Resource manager: http://<dockerhadoop_IP_address>:8088/
* Spark master: http://<dockerhadoop_IP_address>:8080/
* Spark worker: http://<dockerhadoop_IP_address>:8081/
* Hive: http://<dockerhadoop_IP_address>:10000

## Important note regarding Docker Desktop
Since Docker Desktop turned “Expose daemon on tcp://localhost:2375 without TLS” off by default there have been all kinds of connection problems running the complete docker-compose. Turning this option on again (Settings > General > Expose daemon on tcp://localhost:2375 without TLS) makes it all work. I’m still looking for a more secure solution to this.


## Quick Start HDFS

Copy breweries.csv to the namenode.
```
  docker cp breweries.csv namenode:breweries.csv
```

Go to the bash shell on the namenode with that same Container ID of the namenode.
```
  docker exec -it namenode bash
```


Create a HDFS directory /data//openbeer/breweries.

```
  hdfs dfs -mkdir -p /data/openbeer/breweries
```

Copy breweries.csv to HDFS:
```
  hdfs dfs -put breweries.csv /data/openbeer/breweries/breweries.csv
```

To access to the _namenode_ programmatically is necessary to use the endpoint `hdfs://namenode:9000/`

## Quick Start Spark (PySpark)

Go to http://<dockerhadoop_IP_address>:8080 or http://localhost:8080/ on your Docker host (laptop) to see the status of the Spark master.

Go to the command line of the Spark master and start PySpark.
```
  docker exec -it spark-master bash

  /spark/bin/pyspark --master spark://spark-master:7077
```

Load breweries.csv from HDFS.
```
  brewfile = spark.read.csv("hdfs://namenode:9000/data/openbeer/breweries/breweries.csv")

  brewfile.show()
+----+--------------------+-------------+-----+---+
| _c0|                 _c1|          _c2|  _c3|_c4|
+----+--------------------+-------------+-----+---+
|null|                name|         city|state| id|
|   0|  NorthGate Brewing |  Minneapolis|   MN|  0|
|   1|Against the Grain...|   Louisville|   KY|  1|
|   2|Jack's Abby Craft...|   Framingham|   MA|  2|
|   3|Mike Hess Brewing...|    San Diego|   CA|  3|
|   4|Fort Point Beer C...|San Francisco|   CA|  4|
|   5|COAST Brewing Com...|   Charleston|   SC|  5|
|   6|Great Divide Brew...|       Denver|   CO|  6|
|   7|    Tapistry Brewing|     Bridgman|   MI|  7|
|   8|    Big Lake Brewing|      Holland|   MI|  8|
|   9|The Mitten Brewin...| Grand Rapids|   MI|  9|
|  10|      Brewery Vivant| Grand Rapids|   MI| 10|
|  11|    Petoskey Brewing|     Petoskey|   MI| 11|
|  12|  Blackrocks Brewery|    Marquette|   MI| 12|
|  13|Perrin Brewing Co...|Comstock Park|   MI| 13|
|  14|Witch's Hat Brewi...|   South Lyon|   MI| 14|
|  15|Founders Brewing ...| Grand Rapids|   MI| 15|
|  16|   Flat 12 Bierwerks| Indianapolis|   IN| 16|
|  17|Tin Man Brewing C...|   Evansville|   IN| 17|
|  18|Black Acre Brewin...| Indianapolis|   IN| 18|
+----+--------------------+-------------+-----+---+
only showing top 20 rows

```



## Quick Start Spark (Scala)

Go to http://<dockerhadoop_IP_address>:8080 or http://localhost:8080/ on your Docker host (laptop) to see the status of the Spark master.

Go to the command line of the Spark master and start spark-shell.
```
  docker exec -it spark-master bash

  spark/bin/spark-shell --master spark://spark-master:7077
```

Load breweries.csv from HDFS.
```
  val df = spark.read.csv("hdfs://namenode:9000/data/openbeer/breweries/breweries.csv")

  df.show()
+----+--------------------+-------------+-----+---+
| _c0|                 _c1|          _c2|  _c3|_c4|
+----+--------------------+-------------+-----+---+
|null|                name|         city|state| id|
|   0|  NorthGate Brewing |  Minneapolis|   MN|  0|
|   1|Against the Grain...|   Louisville|   KY|  1|
|   2|Jack's Abby Craft...|   Framingham|   MA|  2|
|   3|Mike Hess Brewing...|    San Diego|   CA|  3|
|   4|Fort Point Beer C...|San Francisco|   CA|  4|
|   5|COAST Brewing Com...|   Charleston|   SC|  5|
|   6|Great Divide Brew...|       Denver|   CO|  6|
|   7|    Tapistry Brewing|     Bridgman|   MI|  7|
|   8|    Big Lake Brewing|      Holland|   MI|  8|
|   9|The Mitten Brewin...| Grand Rapids|   MI|  9|
|  10|      Brewery Vivant| Grand Rapids|   MI| 10|
|  11|    Petoskey Brewing|     Petoskey|   MI| 11|
|  12|  Blackrocks Brewery|    Marquette|   MI| 12|
|  13|Perrin Brewing Co...|Comstock Park|   MI| 13|
|  14|Witch's Hat Brewi...|   South Lyon|   MI| 14|
|  15|Founders Brewing ...| Grand Rapids|   MI| 15|
|  16|   Flat 12 Bierwerks| Indianapolis|   IN| 16|
|  17|Tin Man Brewing C...|   Evansville|   IN| 17|
|  18|Black Acre Brewin...| Indianapolis|   IN| 18|
+----+--------------------+-------------+-----+---+
only showing top 20 rows

```

### Deploy remotely

Usually we use the `spark-submit` script to submit Spark _programs_ (_scripts_). The following script can be used to
deploy automatically a Scala Spark program to the spark environment, given it is running in the `remote_host` machine
and that machine has the proper configuration to accept ssh connections (**ssh daemon** is installed and running)

```sh
# $1 -> scala version
# $2 -> jar filename
# $3 -> class to run
sbt package
scp target/scala-"$1"/"$2" user@remote_host:/tmp
ssh user@remote_host docker cp /tmp/"$2" spark-master:/tmp
ssh user@remote_host docker exec spark-master spark-submit \
--master spark://spark-master:7077 --deploy-mode client --class "$3" /tmp/"$2"
```

This script is intended to be run from a _development_ box and it has to be located in the root folder of a Scala project.
It:
- **package**s the project
- **remote-copy** the generated jar package to the `/tmp` folder of the remote host
- **copy** the jar file into the `spark-master` docker container
- runs **spark-submit** inside the `spark-master` container to deploy the program in the cluster

Further customizations and configuration for the _spark-submit_ script can be added (refer to the [documentation](https://spark.apache.org/docs/2.4.5/submitting-applications.html)).

### Remote debug from Intellij

This method allows to debug a Spark program deployed and submitted to the cluster, but we need some adjustments in order to
connect Intellij with the docker (in a remote machine):

- Be aware we've published the port `4747` in `spark-master` container. This is the port we will use to make the jvm listen for connections to start a debugging session
- Given this port we have to create in Intellij a **Run configuration** with type **Remote JVM debug**
- Debug mode has to be **Attach to remote JVM**
- **Host** is the IP where the remote JVM is going to be running. This is **not** the Docker IP as the host IP. We've already published the port in the container, and this is automatically published to the network by Docker. So if the `spark-master` container IP is _172.29.1.10_ and the host machine is **192.168.1.22**, the latter is what we have to set in Intellij for **Host**.
- **Port** is 4747, but it can be whatever always it matches with the port (address) in the configuration **and** is published in the container
- So the preview port is related to the arguments for the remote JVM: `-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=4747`

Then, we can use the following script to automate the packaging and deployment

```sh
# $1 -> scala version
# $2 -> jar filename
# $3 -> class to run
sbt package
scp target/scala-"$1"/"$2" telekosmos@192.168.1.50:/tmp
ssh telekosmos@192.168.1.50 docker cp /tmp/"$2" spark-master:/tmp
ssh telekosmos@192.168.1.50 docker exec -t spark-master spark-submit --packages org.apache.spark:spark-avro_2.12:3.0.0 \
--master spark://spark-master:7077 --deploy-mode client \
--conf "spark.driver.extraJavaOptions=-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=4747" --class "$3" /tmp/"$2"
```

Be aware of `--conf "spark.driver.extraJavaOptions=-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=4747"`, that port is the same we have to configure in Intellij and published in the container.

Once the script is run, we'll see in the terminal

`Listening for transport dt_socket at address: 4747`

To start debugging we just have to hit the **Debug** 🪲 button in Intellij.

If the debugging session ends up with a uncaught exception. we may have to kill the process in the cluster as it will be listening for connections and we won't be able to deploy another debugging session.

The same debugging session can be started straight from the `spark-master` container always we have the jar file persisted inside. To do that, just
```sh
$ docker exec -it spark-master /bin/bash
bash-5.0# spark-submit --master spark://spark-master:7077 --deploy-mode client \
>  --conf spark.driver.extraJavaOptions=-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=4747 \
>  --class fully.qualified.class --packages org.apache.spark:spark-avro_2.12:3.0.0 /path/to/package.jar
Listening for transport dt_socket at address: 4747
```

## Quick Start Hive

Go to the command line of the Hive server and start hiveserver2

```
  docker exec -it hive-server bash

  hiveserver2
```

Maybe a little check that something is listening on port 10000 now
```
  netstat -anp | grep 10000
tcp        0      0 0.0.0.0:10000           0.0.0.0:*               LISTEN      446/java

```

Okay. Beeline is the command line interface with Hive. Let's connect to hiveserver2 now.

```
  beeline -u jdbc:hive2://localhost:10000 -n root --showDbInPrompt=true

  !connect jdbc:hive2://127.0.0.1:10000 scott tiger
```

Didn't expect to encounter scott/tiger again after my Oracle days. But there you have it. Definitely not a good idea to keep that user on production.

Not a lot of databases here yet.
```
  show databases;

+----------------+
| database_name  |
+----------------+
| default        |
+----------------+
1 row selected (0.335 seconds)
```

Let's change that.

```
  create database openbeer;
  use openbeer;
```

And let's create a table.

```
CREATE EXTERNAL TABLE IF NOT EXISTS breweries(
    NUM INT,
    NAME CHAR(100),
    CITY CHAR(100),
    STATE CHAR(100),
    ID INT )
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
location '/data/openbeer/breweries';
```

And have a little select statement going.

```
  select name from breweries limit 10;
+----------------------------------------------------+
|                        name                        |
+----------------------------------------------------+
| name                                                                                                 |
| NorthGate Brewing                                                                                    |
| Against the Grain Brewery                                                                            |
| Jack's Abby Craft Lagers                                                                             |
| Mike Hess Brewing Company                                                                            |
| Fort Point Beer Company                                                                              |
| COAST Brewing Company                                                                                |
| Great Divide Brewing Company                                                                         |
| Tapistry Brewing                                                                                     |
| Big Lake Brewing                                                                                     |
+----------------------------------------------------+
10 rows selected (0.113 seconds)
```

There you go: your private Hive server to play with.


## Configure Environment Variables

The configuration parameters can be specified in the hadoop.env file or as environmental variables for specific services (e.g. namenode, datanode etc.):
```
  CORE_CONF_fs_defaultFS=hdfs://namenode:8020
```

CORE_CONF corresponds to core-site.xml. fs_defaultFS=hdfs://namenode:8020 will be transformed into:
```
  <property><name>fs.defaultFS</name><value>hdfs://namenode:8020</value></property>
```
To define dash inside a configuration parameter, use triple underscore, such as YARN_CONF_yarn_log___aggregation___enable=true (yarn-site.xml):
```
  <property><name>yarn.log-aggregation-enable</name><value>true</value></property>
```

The available configurations are:
* /etc/hadoop/core-site.xml CORE_CONF
* /etc/hadoop/hdfs-site.xml HDFS_CONF
* /etc/hadoop/yarn-site.xml YARN_CONF
* /etc/hadoop/httpfs-site.xml HTTPFS_CONF
* /etc/hadoop/kms-site.xml KMS_CONF
* /etc/hadoop/mapred-site.xml  MAPRED_CONF

If you need to extend some other configuration file, refer to base/entrypoint.sh bash script.
