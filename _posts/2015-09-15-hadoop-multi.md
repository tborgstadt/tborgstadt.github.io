---
layout: post
title: Configure Multi-node Hadoop 2.6 Cluster
description: How to expand a Hadoop 2.6 single node cluster to a multi-node cluster on Ubuntu with VirtualBox.
tags: [hadoop, spark, virtualbox, ubuntu,]
comments: True
---
This project is the final of a series of 3 posts beginning with the following posts:

1. [Install Hadoop 2.6 on Ubuntu 14.04](/blog/2015/08/15/hadoop.html)
2. [Install Spark 1.5 with Hadoop 2.6](/blog/2015/08/23/hadoop-spark.html)

The concept is to take a working single node Hadoop cluster with Spark, and expand to 3 nodes, 1 master namenode and 2 slave data worker nodes.

To simplify this project, each node will run as a virtual machine (VM) on a single hosted virtual environment. This makes it easier to build out the cluster. The single node cluster is modified with common configuration to all nodes and then the single node is cloned, wash and repeat to desired number of nodes. VirtualBox 4.3 is the virtualization platform running on Ubuntu 14.04.

In a production environment, it is most likely that a Hadoop cluster would consist of 100s or even 1000s of nodes hosted in various environments, physical and/or virtual. I want to emphasize this project is for academic purposes only. 

* TOC
{:toc}

## Plan
Each node of the hadoop cluster must communicate with other nodes. Since this cluster will run in a virtual environment on a single host the following static IP addresses will be used, yours might be different.

|**VM Node(Server)**|**Role**|**IP Address**|
|hd-master|hdfs namenode/spark master|192.168.1.100|
|hd-slave1|hdfs datanode/spark worker|192.168.1.101|
|hd-slave2|hdfs datanode/spark worker|192.168.1.102|

## Create Master and apply common cluster configuration
I will begin by cloning the working single cluster VM **hd-spark**, built in previous post, to new VM **hd-master** using the VirtualBox administration GUI. I can save time and have a more consistent implementation if I apply common configuration to the first VM, propogate the common configuration using the cloning process, and then apply specific master and slave configuration later to individual VMs as needed.

Start the new VM **hd-master** and login as administrator (user **tom** in my case).

Note the terminal will still show the server name that **hd-master** was cloned from as show below, but these next steps will correct the host name.

Change hostname.
{% highlight bash %}
# edit file
tom@hd-spark:~$ sudo nano /etc/hostname

# change hd-spark to hd-master:
  hd-master

# save
{% endhighlight %}

Change hosts table
{% highlight bash %}
# edit file
tom@hd-spark:~$ sudo nano /etc/hosts

# remove line '127.0.0.1 localhost'
# set contents of hosts file with:
    192.168.1.100     hd-master
    192.168.1.101     hd-slave1
    192.168.1.102     hd-slave2

# save

# exit terminal
tom@hd-master:~$ exit
{% endhighlight %}

Reopen terminal and login as **hduser**. Note the host name is now correct.
{% highlight bash %}
tom@hd-master:~$ su hduser
hduser@hd-master:/home/tom$ cd
hduser@hd-master:~$
{% endhighlight %}

### Hadoop
Update the following Hadoop configuration files as specified.

#### core-site.xml
{% highlight bash %}
# edit file
hduser@hd-master:~$ sudo nano /usr/local/hadoop/etc/hadoop/core-site.xml

# set the following properties:
<configuration>

  <property>
    <name>fs.default.name</name>
    <value>hdfs://hd-master:9000/</value>
  </property>

</configuration>

# save
{% endhighlight %}

#### hdfs-site.xml
Update dfs.replication to 2 as data will be replicated on the 2 slave datanodes. Verify other properties as shown.
{% highlight bash %}
# edit file
hduser@hd-master:~$ sudo nano /usr/local/hadoop/etc/hadoop/hdfs-site.xml

# set the following properties:
<configuration>

  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/usr/local/hadoop_store/hdfs/namenode</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/usr/local/hadoop_store/hdfs/datanode</value>
  </property>
  <property>
    <name>dfs.webhdfs.enabled</name>
    <value>true</value>
  </property>

</configuration>

# save
{% endhighlight %}


#### yarn-site.xml
{% highlight bash %}
# edit file
hduser@hd-master:~$ sudo nano /usr/local/hadoop/etc/hadoop/yarn-site.xml

# set the following properties:
<configuration>

  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>
  <property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>hd-master:8025</value>
  </property>
  <property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>hd-master:8035</value>
  </property>
  <property>
    <name>yarn.resourcemanager.address</name>
    <value>hd-master:8050</value>
  </property>

</configuration

# save
{% endhighlight %}


#### mapred-site.xml
{% highlight bash %}
# edit file
hduser@hd-master:~$ sudo nano /usr/local/hadoop/etc/hadoop/mapred-site.xml

# set the following properties:
<configuration>

  <property>
    <name>mapred.job.tracker</name>
    <value>hd-master:5431</value>
  </property>
  <property>
    <name>mapred.framework.name</name>
    <value>yarn</value>
  </property>

</configuration>

# save
{% endhighlight %}

#### set master, slave, and worker nodes config
The single master node, **hd-master** will have both primary and secondary HDFS NameNodes. Note that SecondaryNameNode is not really a clone for the NameNode, it is meant to periodically retrieve the NameNode data from memory and persist it on the drive.

List master node for Hadoop in masters config file.
{% highlight bash %}
# edit file
hduser@hd-master:~$ sudo nano /usr/local/hadoop/etc/hadoop/masters

# add master node name:
    hd-master

# save
{% endhighlight %}

List slave nodes for Hadoop in slaves config file.
{% highlight bash %}
# edit file
hduser@hd-master:~$ sudo nano /usr/local/hadoop/etc/hadoop/slaves

# add slave nodes names:
    hd-slave1
    hd-slave2

# save
{% endhighlight %}

List slave workers for Spark in slaves file.
{% highlight bash %}
# copy the template 
hduser@hd-master:~$ sudo cp /usr/local/spark/conf/slaves.template slaves

# edit the file
hduser@hd-master:~$ sudo nano /usr/local/spark/conf/slaves

# add slave workers names:
    hd-slave1
    hd-slave2

# save
{% endhighlight %}

#### clear logs
Since the single node cluster, **hd-master**, was run previously clear the logs before cloning.
{% highlight bash %}
# remove all files in logs folder
hduser@hd-master:~$ sudo rm -rf /usr/local/hadoop/logs
{% endhighlight %}

## Create and configure Slave nodes
Clone VirtualBox **hd-master** to **hd-slave1** using the VirtualBox manager.

Again, at this point note the terminal will show the server name that **hd-slave1** was cloned from as show below, but these next steps will correct the host name.


Update hostname file with server name.
{% highlight bash %}
# edit file
tom@hd-master:~$ sudo nano /etc/hostname

# update server name:
    hd-slave1

# save

# shutdown for network config
tom@hd-master:~$ sudo shutdown -P now
{% endhighlight %}

From VirtualBox manager, configure a network adapter for VM **hd-slave1**.

|**VBox**|**Attached to:**|**Purpose**|
|Adapter 1:|Bridged Adapter|for static IP (for HDFS)|

From Ubuntu create an IPv4 Ethernet network connection with the following configuration and assign to the adapter made available by VirtualBox.

|**VBox**|**Ubuntu**|**Type**|**IP**|**Netmask**|**Gateway**|
|Adapter2:|eth0:|Manual|192.168.1.101|255.255.255.0|0|

Repeat steps in this section for additional slave(s), in this case **hd-slave2** with IP: 192.168.1.102

## Complete Master node configuration
To have Internet connectivity for downloads, etc., it is necessary to configure two network adapters. This is done on the **hd-master** node only but could also be done on slave nodes if needed.

From VirtualBox manager UI, configure 2 network adapters as show below for VM **hd-master**.

|**VBox**|**Attached to:**|**Purpose**|
|Adapter1:|NAT|for Internet access|
|Adapter2:|Bridged Adapter|for static IP|

From Ubuntu create 2 IPv4 Ethernet network connections and assign to the adapters made available by VirtualBox.

|**VBox**|**Ubuntu**|**Type**|**IP**|**Netmask**|**Gateway**|
|Adapter1:|eth1:|DHCP|n/a|n/a|n/a|
|Adapter2:|eth0:|Manual|192.168.1.100|255.255.255.0|0|

## Test connectivity between nodes
Ensure all nodes in the cluster can communicate with other nodes.
{% highlight bash %}
hduser@hd-master:~$ ping hd-slave1
# should return:
#   Reply from 192.168.1.101
#   cntrl-c to break

hduser@hd-master:~$ ping hd-slave2
# should return:
#   Reply from 192.168.1.102
#   cntrl-c to break

hduser@hd-slave1:~$ ping hd-master
# should return:
#   Reply from 192.168.1.100
#   cntrl-c to break

hduser@hd-slave2:~$ ping hd-master
# should return:
#   Reply from 192.168.1.100
#   cntrl-c to break
{% endhighlight %}

Ensure all nodes can ssh into other nodes without password prompt.
{% highlight bash %}
hduser@hd-master:/home/tom$ ssh hd-slave1
# result:
hduser@hd-slave1:~$

hduser@hd-slave1:/home/tom$ ssh hd-master
# result:
hduser@hd-master:~$

hduser@hd-slave2:/home/tom$ ssh hd-master
# result:
hduser@hd-master:~$
{% endhighlight %}

## Format HDFS
Format the namenode and HDFS. If prompted to re-format, answer yes.
{% highlight bash %}
hduser@hd-master:~$ hadoop namenode -format
{% endhighlight %}

If a problem with the formatting procedure, compare the "clusterID= " in each of the following files (these paths can be found by looking in hdfs-site.xml).  The clusterID must be the same in each of these files across master and slaves. If clusterID is different between these files, copy the clusterID from the namenode VERSION file to the datanode VERSION file and save. If these are different the datanodes will not start.

Also ensure that the datanodeUuid is unique by slave, if not, make unique.
{% highlight bash %}
$ nano /usr/local/hadoop_store/hdfs/namenode/current/VERSION
$ nano /usr/local/hadoop_store/hdfs/datanode/current/VERSION
{% endhighlight %}

## Start Hadoop
{% highlight bash %}
# start Hadoop daemons
hduser@hd-master:~$ start-dfs.sh

# start MapReduce daemons: 
hduser@hd-master:~$ start-yarn.sh

# verify Hadoop daemons on Master:
hduser@hd-master:~$ jps
# should see: NameNode, SecondaryNameNode, Jps, ResourceManager

# verify Hadoop daemons on Slaves:
hduser@hd-slave1:~$ jps
hduser@hd-slave2:~$ jps
# should see for each: Jps, NodeManager, DataNode
{% endhighlight %}

## Start Spark
{% highlight bash %}
# start Spark master
hduser@hd-master-spk:~$ start-master.sh

# start Spark workers
hduser@hd-master-spk:~$ start-slaves.sh
{% endhighlight %}

## Verify active Hadoop and Spark subsystems 

Check Spark Master web UI:  http://hd-master:8080/

<figure><a href="/images/hd-master-02.png"><img src="/images/hd-master-02.png"></a>
<figcaption></figcaption>
</figure>


Check Hadoop web UI:  http://hd-master:50070/

<figure><a href="/images/hd-master-03.png"><img src="/images/hd-master-03.png"></a>
<figcaption></figcaption>
</figure>

<figure><a href="/images/hd-master-04.png"><img src="/images/hd-master-04.png"></a>
<figcaption></figcaption>
</figure>


Checking jps on each node shows Spark Master and Worker systems are now included.

<figure><a href="/images/hd-master-01.png"><img src="/images/hd-master-01.png"></a>
<figcaption></figcaption>
</figure>

## Test

Create text file and put on hdfs /myhdfs/mytext
{% highlight bash %}
# create text file
hduser@hd-master:~$  nano mytext

# paste in any text

# save

# create a new directory in HDFS
hduser@hd-master:~$  hadoop fs -mkdir /myhdfs

# copy the local file into HDFS
hduser@hd-master:~$  hadoop fs -put mytext /myhdfs/mytext
{% endhighlight %}


Start pyspark shell
{% highlight bash %}
# start pyspark shell
hduser@hd-master:~$ pyspark
{% endhighlight %}

Enter the following python code to perform a line count and word count of the text file created above. See if the results make sense based on the text in the file.
{% highlight python %}
>>> lines = sc.textFile('hdfs://hd-master:9000/myhdfs/mytext')
>>> lines_nonempty = lines.filter( lambda x: len(x) > 0 )
>>> lines_nonempty.count()
3
>>> words = lines_nonempty.flatMap(lambda x: x.split())
>>> wordcounts = words.map(lambda x: (x, 1)).reduceByKey(lambda x,y:x+y).map(lambda x:(x[1],x[0])).sortByKey(False)
>>> wordcounts.take(15)
[(20, u'the'), (13, u'in'), (9, u'and'), (9, u'a'), (8, u'was'), (5, u'of'), (4, u'to'), (4, u'Minneapolis'), (4, u'for'), (4, u'It')]

>>> quit()

{% endhighlight %}

Verify can access text document via WebHDFS:

<figure><a href="/images/hd-master-05.png"><img src="/images/hd-master-05.png"></a>
<figcaption></figcaption>
</figure>

|http://hd-master:50070/webhdfs/v1/myhdfs/mytext?op=OPEN|

Should get a prompt to "Save File"

## Create start and shutdown scripts
Need some wrappers to start and stop this whole thing.

Add /home/hduser/bin to system PATH
{% highlight bash %}
# edit user startup
hduser@hd-master:~$ sudo nano ~/.bashrc 

# append the following line:
export PATH="/home/hduser/bin:$PATH"

# save

# reset
hduser@hd-master:~$ source ~/.bashrc
{% endhighlight %}

Add a startup script.
{% highlight bash %}
# create bin directory
hduser@hd-master:~$ mkdir bin

# make bin current directory
hduser@hd-master:~$ cd bin

# create a file in bin
hduser@hd-master:~/bin$ sudo nano start-env.sh

# copy the following lines into the file:
    #!/bin/bash

    # start HDFS
    start-dfs.sh

    # start MapReduce
    start-yarn.sh

    # start Spark
    start-master.sh
    start-slaves.sh

# save

# make the file executable
hduser@hd-master:~/bin$ sudo chmod +x start-env.sh
{% endhighlight %}

Add a shutdown script.
{% highlight bash %}
# create a file in bin
hduser@hd-master:~/bin$ sudo nano stop-env.sh

# copy the following lines into the file:
    #!/bin/bash

    # stop Spark
    stop-slaves.sh
    stop-master.sh

    # stop MapReduce
    stop-yarn.sh

    # stop HDFS
    stop-dfs.sh

# save

# make the file executable
hduser@hd-master:~/bin$ sudo chmod +x stop-env.sh
{% endhighlight %}

This concludes this little project!

