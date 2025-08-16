# EcomState: A local-first installation of Hadoop, spark and Hop for E-commerce Data processing pipeline ( Educational purpose, Not for production ).

## Hadoop: `hadoop 3.4.1`

For data lake (Storing raw, and unprocessed data), we'll be using Hadoop. Hadoop is installed via docker for identical repeatable workflow.

### Installing Hadoop ( Docker & Docker compose )

For successful installtion of Hadoop on docker, we'll requiring two files `config` for supplying the docker containers with essential configuration, and `docker-compose.yaml` for starting all the required containers for runnung standalone Hadoop cluster. The clusters includes a `namenode`, `datanode`, `resource manager` and a `node manager`.

#### docker-compose.yaml

```yaml

version: "2"
services:
   namenode:
      image: apache/hadoop:3
      hostname: namenode
      command: ["hdfs", "namenode"]
      ports:
        - 9870:9870
      env_file:
        - ./config
      environment:
          ENSURE_NAMENODE_DIR: "/tmp/hadoop-root/dfs/name"
   datanode:
      image: apache/hadoop:3
      command: ["hdfs", "datanode"]
      env_file:
        - ./config  
   resourcemanager:
      image: apache/hadoop:3
      hostname: resourcemanager
      command: ["yarn", "resourcemanager"]
      ports:
         - 8088:8088
      env_file:
        - ./config
      volumes:
        - ./test.sh:/opt/test.sh
   nodemanager:
      image: apache/hadoop:3
      command: ["yarn", "nodemanager"]
      env_file:
        - ./config
```

#### config

```plaintext
CORE-SITE.XML_fs.default.name=hdfs://namenode
CORE-SITE.XML_fs.defaultFS=hdfs://namenode
HDFS-SITE.XML_dfs.namenode.rpc-address=namenode:8020
HDFS-SITE.XML_dfs.replication=1
MAPRED-SITE.XML_mapreduce.framework.name=yarn
MAPRED-SITE.XML_yarn.app.mapreduce.am.env=HADOOP_MAPRED_HOME=$HADOOP_HOME
MAPRED-SITE.XML_mapreduce.map.env=HADOOP_MAPRED_HOME=$HADOOP_HOME
MAPRED-SITE.XML_mapreduce.reduce.env=HADOOP_MAPRED_HOME=$HADOOP_HOME
YARN-SITE.XML_yarn.resourcemanager.hostname=resourcemanager
YARN-SITE.XML_yarn.nodemanager.pmem-check-enabled=false
YARN-SITE.XML_yarn.nodemanager.delete.debug-delay-sec=600
YARN-SITE.XML_yarn.nodemanager.vmem-check-enabled=false
YARN-SITE.XML_yarn.nodemanager.aux-services=mapreduce_shuffle
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.maximum-applications=10000
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.maximum-am-resource-percent=0.1
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.resource-calculator=org.apache.hadoop.yarn.util.resource.DefaultResourceCalculator
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.queues=default
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.capacity=100
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.user-limit-factor=1
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.maximum-capacity=100
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.state=RUNNING
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.acl_submit_applications=*
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.acl_administer_queue=*
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.node-locality-delay=40
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.queue-mappings=
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.queue-mappings-override.enable=false

```

>  `important` Do not forget to set the `HADOOP_HOME` variable in your shell, otherwise the installation produces error, requiring the variable.

The files `docker-compose.yaml` and `config` are ought to be in the same level in a directory, otherwise change the `.env_file` attribute in `docker-compose.yaml` to appropriate file location.

Above `docker-compose.yaml` starts containers `hadoop_namenode_1`, `hadoop_datanode_1`, `hadoop_nodemanager_1`, and `hadoop_resourcemanager_1` with the specified hadoop version, which can be viewed by typing `docker ps`.

```plaintext

~ docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS                                       NAMES
adf68727c065   apache/hadoop:3.4.1   "/usr/local/bin/dumb…"   21 minutes ago   Up 21 minutes   0.0.0.0:9870->9870/tcp, :::9870->9870/tcp   hadoop_namenode_1
4b054184c8b7   apache/hadoop:3.4.1   "/usr/local/bin/dumb…"   21 minutes ago   Up 21 minutes                                               hadoop_nodemanager_1
bcf5815bbbfc   apache/hadoop:3.4.1   "/usr/local/bin/dumb…"   21 minutes ago   Up 21 minutes   0.0.0.0:8088->8088/tcp, :::8088->8088/tcp   hadoop_resourcemanager_1
bcbd06417df8   apache/hadoop:3.4.1   "/usr/local/bin/dumb…"   21 minutes ago   Up 21 minutes                                               hadoop_datanode_1

```

#### Logging into an node interactively, and executing a `hdfs` command.

To log into an node interactively, go to terminal and type `docker exec -it hadoop_namenode_1 /bin/bash`. This command logs into the `hadoop_namenode_1` container, and starts the bash shell.
