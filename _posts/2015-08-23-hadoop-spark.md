---
layout: post
title: Install Spark 1.5 with Hadoop 2.6
description: How to install Spark 1.5 alongside a single node of Hadoop 2.6 running on Ubuntu 14.04
tags: [hadoop, spark, ubuntu, virtualbox]
comments: True
---
I am installing Spark 1.5 today on a new single cluster of Hadoop 2.6, created from a previous post.

* TOC
{:toc}

## Create a new VM
Clone previous virtual machine (VM) **hd-single** to new VM **hd-spark** using the VirtualBox administration GUI.

Start the new VM **hd-spark** and login as administrator (user **tom** in my case).

Change hostname.
{% highlight bash %}
# edit file
tom@hd-single:~$ sudo nano /etc/hostname

# change hd-single to hd-spark:
  hd-spark

# save
{% endhighlight %}

Change hosts table.
{% highlight bash %}
# edit file
tom@hd-single:~$ sudo nano /etc/hosts

# clear the hosts file and make contents:
  127.0.0.1   hd-spark
  127.0.0.1   localhost

# save file

# exit terminal
tom@hd-single:~$ exit
{% endhighlight %}

Open a terminal and change SSH authorized keys:
{% highlight bash %}
# login as hduser
tom@hd-spark:~$ su hduser

# change hduser@hd-single to hduser@hd-spark
hduser@hd-spark:/home/tom$ cd
hduser@hd-spark:~$ cd .ssh
hduser@hd-spark:~/.ssh$ nano authorized_keys

# change 'hduser@hd-single' to 'hduser@hd-spark'

# save

hduser@hd-spark:~/.ssh$ cd
{% endhighlight %}

## Install Spark

{% highlight bash %}
# download Spark
hduser@hd-spark:~$ wget http://d3kbcqa49mib13.cloudfront.net/spark-1.5.0-bin-hadoop2.6.tgz

# untar (unzip)
hduser@hd-spark:~$ tar -xvf spark-1.5.0-bin-hadoop2.6.tgz

# create Spark directory
hduser@hd-spark:~$ sudo mkdir /usr/local/spark

# change working directory
hduser@hd-spark:~$ cd spark-1.5.0-bin-hadoop2.6

# move the contents of Spark installation to the /usr/local/spark directory:
hduser@hd-spark:~/spark-1.5.0-bin-hadoop2.6$ sudo mv * /usr/local/spark

# change owner of the spark directory
hduser@hd-spark:~/spark-1.5.0-bin-hadoop2.6$ sudo chown -R hduser:hadoop /usr/local/spark

# edit ~/.bashrc so Spark environment
hduser@hd-spark:~$ sudo nano ~/.bashrc

# add the following lines:
  # SPARK VARIABLES BEGIN
    export SPARK_HOME=/usr/local/spark
    export PATH=$PATH:$SPARK_HOME/bin
    export PATH=$PATH:$SPARK_HOME/sbin
  # SPARK VARIABLES END

# save ~/.bashrc

# refresh environment
hduser@hdp-single-spk:~$ source ~/.bashrc

# install is complete
{% endhighlight %}

## Start Spark and verify
{% highlight bash %}
# start Spark master
hduser@hd-spark:~$ start-master.sh

# start Spark workers
hduser@hd-spark:~$ start-slaves.sh
{% endhighlight %}

Open a browser and navigate to Spark Administration.

|Spark Admin|http://hd-spark:8080/|
|URL:|spark://hd-spark:7077|
|REST URL:|spark://hd-spark:6066 (cluster mode)|

## Start Hadoop and verify
{% highlight bash %}
# start Hadoop daemons
hduser@hd-master:~$ start-dfs.sh

# start MapReduce daemons: 
hduser@hd-master:~$ start-yarn.sh

{% endhighlight %}

Check the cluster status. Open a browser and navigate to the following URLs:

|http://hd-spark:50070/|web UI of the NameNode daemon|

Spark Administration.
<figure><a href="/images/hd-spark-01.png"><img src="/images/hd-spark-01.png"></a>
<figcaption></figcaption>
</figure>

NameNode Status.
<figure><a href="/images/hd-spark-02.png"><img src="/images/hd-spark-02.png"></a>
<figcaption></figcaption>
</figure>

DataNode Status.
<figure><a href="/images/hd-spark-03.png"><img src="/images/hd-spark-03.png"></a>
<figcaption></figcaption>
</figure>

## Test Spark context

Create text file and put on hdfs /myhdfs/mytext
{% highlight bash %}
# create text file
hduser@hd-spark:~$  nano mytext

# paste in any text

# save

# create a new directory in HDFS
hduser@hd-spark:~$  hadoop fs -mkdir /myhdfs

# copy the local file into HDFS
hduser@hd-spark:~$  hadoop fs -put mytext /myhdfs/mytext
{% endhighlight %}

Start pyspark shell.
{% highlight bash %}
# start pyspark shell
hduser@hd-spark:~$ pyspark
{% endhighlight %}

Enter the following python code to perform a line count and word count of the text file created above. See if the results make sense based on the text in the file.
{% highlight python %}
>>> lines = sc.textFile('hdfs://hd-spark:9000/myhdfs/mytext')
>>> lines_nonempty = lines.filter( lambda x: len(x) > 0 )
>>> lines_nonempty.count()
3
>>> words = lines_nonempty.flatMap(lambda x: x.split())
>>> wordcounts = words.map(lambda x: (x, 1)).reduceByKey(lambda x,y:x+y).map(lambda x:(x[1],x[0])).sortByKey(False)
>>> wordcounts.take(15)
[(20, u'the'), (13, u'in'), (9, u'and'), (9, u'a'), (8, u'was'), (5, u'of'), (4, u'to'), (4, u'Minneapolis'), (4, u'for'), (4, u'It')]

>>> quit()

{% endhighlight %}

That is it! Spark is ready to use.



