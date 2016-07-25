# Twitter Data Analysis with Apache Flume and Hive using CDH3
Store live streaming Tweeter data in HDFS using Apache flume, further load this data in Hive for analysis. </br>Example use Cloudera Hadoop Distribution CHD3.

## Target System:
	1. Ubuntu v10.10
	2. Cloudera Hadoop Distribution CDH3 v0.3.7
	3. Hadoop v0.20.2-cdh3u0 in Pseudo-distribution mode
	4. Java v1.6.0_24
	5. Hive v0.7.0
	6. Flume v1.6.0; location (/usr/lib/flume-ng/apache-flume-1.6.0-bin/)

## Pre-requesites:
	1. All deamons are running.
	2. Java, Flume and Hive is installed and configured.
	3. metastore.db using Java 'Derby' in Embedded mode is configured.
	4. Knowledge of Hadoop HDFS, Hive and Flume.

## Issues:
  1. [Avro block size is invalid or too large](http://www.stackoverflow.com/questions/30661478/unable-to-correctly-load-twitter-avro-data-into-hive-table) when using Flume and Twitter streaming

  2. `TwitterAgent.sources.Twitter.type = org.apache.flume.source.twitter.TwitterSource` </br>
	 "Twitter 1% Firehose Source</br>
	 This source is highly experimental. It connects to the 1% sample Twitter Firehose using </br>
	 streaming API and continuously downloads tweets, converts them to Avro format, and sends </br> 
	 Avro events to a downstream Flume sink"		

  3. JSONSerDe compatibility.



## Solutions:
  1. Use Cloudera JAR file `flume-sources-1.0-SNAPSHOT.jar` for Twitter Source
	
  2. Use [Cloudera TwitterSource](http://stackoverflow.com/questions/36053306/cloudera-5-4-2-avro-block-size-is-invalid-or-too-large-when-using-flume-and-twi/36189152#36189152) in flume agent
		
	 **`TwitterAgent.sources.Twitter.type =  com.cloudera.flume.source.TwitterSource`**

  3. Use Cloudera `JSONSerDe hive-serdes-1.0-SNAPSHOT.jar`
	
  4. Both JAR files are build and tested on Cludera Hadoop Distribution CDH3 v0.3.7, </br>
	 for other target systems, user can compile and built JAR files on target system using maven3, </br>
	 for details see **Annexure-A** and **Annexure-B** 

		a. flume-sources-1.0-SNAPSHOT.jar, 
		b. hive-serdes-1.0-SNAPSHOT.jar 

  5. Further reading: </br>
	a. [How-to: Analyze Twitter Data with Apache Hadoop](https://blog.cloudera.com/blog/2012/09/analyzing-twitter-data-with-hadoop/) </br>
	b. [Analyzing Twitter Data with Apache Hadoop, Part 2: Gathering Data with Flume](http://blog.cloudera.com/blog/2012/10/analyzing-twitter-data-with-hadoop-part-2-gathering-data-with-flume/) </br>
	c. [Analyzing Twitter Data with Apache Hadoop, Part 3: Querying Semi-structured Data with Apache Hive](http://blog.cloudera.com/blog/2012/11/analyzing-twitter-data-with-hadoop-part-3-querying-semi-structured-data-with-hive/) 

  6. Follow each STEP one by one.

## Note:
	1. Code is tested on Cloudera Hadoop Distribution CDH3. 
	2. Pre build JAR files are available at /lib/ folder.
		a. flume-sources-1.0-SNAPSHOT.jar
		b. hive-serdes-1.0-SNAPSHOT.jar


## 1. Setup Twitter account:

	Setup Twitter application to get consumerKey and accessToken details.

* Login/Open [Twitter](https://www.twitter.com/) account.
* Click on [create app](https://apps.twitter.com/app) link.
* Fill necessary details.
* Enter full web site URL. Last forward slash (/) is required; otherwise it will not validate. Example;
	<http://www.yahoo.com/>
* Accept the agreement and click on ‘create your Twitter application’.
* Go to ‘Keys and Access Token’ tab.
* Copy the consumer key and the consumer secret.
* Scroll down further and click on ‘create my access token’.
* Copy the Access Token and Access token Secret.
* Note down all four key at some place, will need this in Flume config file.


## 2. Create subdirectories and copy JAR file:

	Create following directories and copy JAR file available under /lib/ folder.
	
* $HOME = /home/loggedin_user/
	
		$ cd $HOME/Desktop/
		$ mkdir hadoop-Use-Cases
		$ cd hadoop-Use-Cases/	
		$ mkdir  twitter-Analysis
		$ cd twitter-Analysis/

* Download JAR file `flume-sources-1.0-SNAPSHOT.jar` available under [/lib/](https://github.com/KhareS/Twitter-Data-Analysis-Using-Flume-Hive/tree/master/lib) folder to `$HOME/Desktop/hadoop-Use-Cases/twitter-Analysis/`
 
* create plugind.d & other sub-directories
	
		/usr/lib/flume-ng/plugins.d/twitter-streaming/lib/
		/var/lib/flume-ng/plugins.d/twitter-streaming/lib

* Copy `flume-sources-1.0-SNAPSHOT.jar` to following directories

		$ cd $HOME/Desktop/hadoop-Use-Cases/twitter-Analysis 
		$ sudo cp flume-sources-1.0-SNAPSHOT.jar /usr/lib/flume-ng/plugins.d/twitter-streaming/lib/
		$ sudo cp flume-sources-1.0-SNAPSHOT.jar /var/lib/flume-ng/plugins.d/twitter-streaming/lib/

* See **Annexure-A**; If want to built `flume-sources-1.0-SNAPSHOT.jar`
	

## 3. Setting up Flume agent:

	Create `flume-twitter-analysis-conf.properties` file and use consumerKey & accessToken details.
	
* Create HDFS directory
	
		$ hadoop fs -mkdir /user/cloudera/flume/tweetsinput

* Create configuration file for Flume agent. </br> 
  Name this file `flume-twitter-analysis-conf.properties` and save at: </br>
  $HOME/Desktop/hadoop-Use-Cases/twitter-Analysis/
	
		$ cd $HOME/Desktop/hadoop-Use-Cases/twitter-Analysis/
		$ gedit flume-twitter-analysis-conf.properties

* Paste following code in config file `flume-twitter-analysis-conf.properties` and Save.
	
		TwitterAgent.sources = Twitter 
		TwitterAgent.channels = MemChannel 
		TwitterAgent.sinks = HDFS
  
		# Use CLoudera Twitter Source;
		# place your consumerKey and accessToken details here
		# Describing/Configuring the source
		TwitterAgent.sources.Twitter.type = com.cloudera.flume.source.TwitterSource
		TwitterAgent.sources.Twitter.consumerKey=
		TwitterAgent.sources.Twitter.consumerSecret=
		TwitterAgent.sources.Twitter.accessToken=
		TwitterAgent.sources.Twitter.accessTokenSecret=
		TwitterAgent.sources.Twitter.maxBatchSize = 1000
		TwitterAgent.sources.Twitter.maxBatchDurationMillis = 1000
		TwitterAgent.sources.Twitter.keywords=hadoop, big data, analytics, bigdata, cloudera, data science, data scientist, business intelligence, mapreduce, data warehouse, data warehousing, mahout, hbase, nosql, newsql, businessintelligence, cloudcomputing

		# Use a channel which buffers events in memory
		TwitterAgent.channels.MemChannel.type=memory
		TwitterAgent.channels.MemChannel.capacity=100
		TwitterAgent.channels.MemChannel.transactionCapacity=100

		# Describing/Configuring the sink 
		TwitterAgent.sinks.HDFS.channel=MemChannel
		TwitterAgent.sinks.HDFS.type=hdfs
		TwitterAgent.sinks.HDFS.hdfs.path=/user/cloudera/flume/tweetsinput
		TwitterAgent.sinks.HDFS.hdfs.fileType=DataStream
		TwitterAgent.sinks.HDFS.hdfs.writeformat=Text
		TwitterAgent.sinks.HDFS.hdfs.batchSize=100
		TwitterAgent.sinks.HDFS.hdfs.rollSize=0
		TwitterAgent.sinks.HDFS.hdfs.rollCount=1000
		TwitterAgent.sinks.HDFS.hdfs.rollInterval=600

		# Bind the source and sink to the channel
		TwitterAgent.sources.Twitter.channels = MemChannel
		TwitterAgent.sinks.HDFS.channel = MemChannel


## 4. Copy Flume agent in flume config directory:

* Navigate to source directory and copy Flume agent 
 
		$ cd $HOME/Desktop/hadoop-Use-Cases/twitter-Analysis/
		$ sudo cp flume-twitter-analysis-conf.properties /usr/lib/flume-ng/apache-flume-1.6.0-bin/conf/

* Test file presence
 
		$ cd /usr/lib/flume-ng/apache-flume-1.6.0-bin/conf/
		$ ls -l


## 5. Start Flume agent:

* Command to start Flume Agent 
 
		$ /usr/lib/flume-ng/apache-flume-1.6.0-bin/bin/flume-ng agent -n TwitterAgent -c conf -f /usr/lib/flume-ng/apache-flume-1.6.0-bin/conf/flume-twitter-analysis-conf.properties


* Command to start Flume Agent with detailed debug information 
 
		$ /usr/lib/flume-ng/apache-flume-1.6.0-bin/bin/flume-ng agent -n TwitterAgent -c conf -f /usr/lib/flume-ng/apache-flume-1.6.0-bin/conf/flume-twitter-analysis-conf.properties -Dflume.root.logger=DEBUG,console

* To Stop streaming of data, press
		
		$ Ctrl + c


## 6. Download Cloudera JSONSerDe file:

* Download JAR file `hive-serdes-1.0-SNAPSHOT.jar` available under  [/lib/](https://github.com/KhareS/Twitter-Data-Analysis-Using-Flume-Hive/tree/master/lib) folder to $HOME/Desktop/hadoop-Use-Cases/twitter-Analysis/ 
 
* Check JSONSerDe file 

		$ cd $HOME/Desktop/hadoop-Use-Cases/twitter-Analysis 

* See **Annexure-B**; If want to built `hive-serdes-1.0-SNAPSHOT.jar`



## 7. Add JSONSerDe file and create table structure in Hive:

* Start Hive CLI 

		$ sudo hive 
		hive>		

* Add JSONSerDe JAR file 'hive-serdes-1.0-SNAPSHOT.jar`  location 

		hive> ADD JAR $HOME/Desktop/hadoop-Use-Cases/twitter-Analysis/hive-serdes-1.0-SNAPSHOT.jar;

* create new databse
 
		hive> create database twitter-Analysis;

* use database

		hive> use twitter-Analysis;
		
* Create Table to store JSON tweets into Hive tables, without using Partition. </br> 
  Stored as internal table at:- /user/hive/warehouse/twitter-Analysis.db/tweets </br>

		CREATE TABLE tweets (
  		  id BIGINT,
  		  created_at STRING,
  		  source STRING,
  		  favorited BOOLEAN,
  		  retweeted_status STRUCT<
     		    text:STRING,
    		    user:STRUCT<screen_name:STRING,name:STRING>,
    		    retweet_count:INT>,
  		  entities STRUCT<
    		    urls:ARRAY<STRUCT<expanded_url:STRING>>,
    		    user_mentions:ARRAY<STRUCT<screen_name:STRING,name:STRING>>,
    		    hashtags:ARRAY<STRUCT<text:STRING>>>,
  		  text STRING,
  	          user STRUCT<
    		    screen_name:STRING,
    		    name:STRING,
    		    friends_count:INT,
    		    followers_count:INT,
    		    statuses_count:INT,
    		    verified:BOOLEAN,
    		    utc_offset:INT,
    		    time_zone:STRING>,
  		    in_reply_to_screen_name STRING
		   ) 
		ROW FORMAT SERDE 'com.cloudera.hive.serde.JSONSerDe';
		

* Describe table
		
		hive> describe formatted tweets;


## 8. Load data into Hive tables:

* Copy flume data from HDFS to local disk and then into Hive tables. Get Flume data file names. Open new terminal
 
		$ hadoop fs -lsr /user/cloudera/flume/tweetsinput/

* Copy Flume data from HDFS		
	
		$ cd $HOME/Desktop/hadoop-Use-Cases/twitter-Analysis	
		$ mkdir  rawTweets
		$ cd rawTweets/
		$ hadoop fs -get /user/cloudera/flume/tweetsinput/*  $HOME/Desktop/hadoop-Use-Cases/twitter-Analysis/rawTweets/	

* Load Data into Hive tables, your file name may have a different extension like; FlumeData.xxxxx		
		
		hive> LOAD DATA LOCAL INPATH '$HOME/Desktop/hadoop-Use-Cases/twitter-Analysis/rawTweets/FlumeData.1468333621171' INTO TABLE tweets;

		hive> LOAD DATA LOCAL INPATH '$HOME/Desktop/hadoop-Use-Cases/twitter-Analysis/rawTweets/FlumeData.1468333621172' INTO TABLE tweets;


## 9. Run queries:

* Select total count 		
	
		hive> SELECT count(*) FROM tweets;

		Result in:
		

* How records are look like? Check one record.

		hive> SELECT * FROM tweets LIMIT 1;
		
		Result in:
		752871872170106881	Tue Jul 12 14:26:56 +0000 2016	<a href="http://twitter.com/download/android" rel="nofollow">Twitter for Android</a>	false	{"text":null,"user":null,"retweet_count":null}	{"urls":[],"user_mentions":[{"screen_name":"Amarte_Kong","name":"빈슈"}],"hashtags":[]}	@Amarte_Kong 맨션 수에 따라 올라가는 것..? ㅋㅋㅌㅋㅋ근데 왜때뮤네 빈슈가 없눈거야ㅠㅠ 트이터 일해라ㅠㅠ	{"screen_name":"Ha_an10","name":"메인이벤트❀하안❀","friends_count":218,"followers_count":892,"statuses_count":25758,"verified":false,"utc_offset":null,"time_zone":null}	Amarte_Kong Time taken: 0.383 seconds



		
---



### Annexure-A:

If using other target system, please built **`flume-sources-1.0-SNAPSHOT.jar`** using maven3 

* Download master file

		$ wget https://github.com/cloudera/cdh-twitter-example/archive/master.zip 

* Unzip

		$ unzip -o master.zip

* Install Maven if not installed. [How to install Maven on Ubuntu?](http://stackoverflow.com/questions/15630055/how-to-install-maven-3-on-ubuntu-15-10-15-04-14-10-14-04-lts-13-10-13-04-12-10-1)

* Navigate to folder

		$ cd cdh-twitter-example-master/flume-sources

* Build package; This will generate a file called `flume-sources-1.0-SNAPSHOT.jar` in the **target** directory

		$ mvn package

* Check newly build JAR file

		$ cd target/

* Go to STEP-02: (Create subdirectories and copy JAR file)


### Annexure-B:

If using other target system, please built **`hive-serdes-1.0-SNAPSHOT.jar`** using maven3 

* Download master file

		$ wget https://github.com/cloudera/cdh-twitter-example/archive/master.zip 

* Unzip

		$ unzip -o master.zip

* Navigate to folder

		$ cd cdh-twitter-example-master/hive-serdes

* Build package; This will generate a file called `hive-serdes-1.0-SNAPSHOT.jar` in the **target** directory

		$ mvn package

* Check newly build JAR file

		$ cd target/

* Go to STEP-06: (Create subdirectories and copy JAR file)


