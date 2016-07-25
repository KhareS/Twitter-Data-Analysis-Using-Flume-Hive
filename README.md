# Twitter Data Analysis with Apache Flume and Hive using CDH3
Store live streaming Tweeter data in HDFS using Apache flume, further load this data in Hive for analysis. Example use Cloudera Hadoop Distribution CHD3.

## Target System:
	1. Ubuntu v10.10
	2. Cloudera Hadoop Distribution CDH3 v0.3.7
	3. Hadoop v0.20.2-cdh3u0 in Pseudo-distribution mode
	4. Java v1.6.0_24
	5. Hive v0.7.0
	6. Flume v1.6.0

## Pre-requesites:
	1. All deamons are running.
	2. Java, Flume and Hive is installed and configured.
	3. metastore.db using Java 'Derby' in Embedded mode is configured.
	4. Knowledge of Hadoop HDFS, Hive and Flume.

## Issues:
	1. Avro block size is invalid or too large when using Flume and Twitter streaming
		http://stackoverflow.com/questions/30661478/unable-to-correctly-load-twitter-avro-data-into-hive-table

	2. 	TwitterAgent.sources.Twitter.type = org.apache.flume.source.twitter.TwitterSource
		"Twitter 1% Firehose Source
		This source is highly experimental. It connects to the 1% sample Twitter Firehose using 
	        streaming API and continuously downloads tweets, converts them to Avro format, and sends 
		Avro events to a downstream Flume sink"		

	3. 	JSONSerDe compatibility.

## Solutions:
	1. Use Cloudera JAR file flume-sources-1.0-SNAPSHOT.jar for Twitter Source
	
	2. Use Cloudera TwitterSource in flume agent
		
		<span style="background-color: #FFFF00">TwitterAgent.sources.Twitter.type =  com.cloudera.flume.source.TwitterSource</span>

		http://stackoverflow.com/questions/36053306/cloudera-5-4-2-avro-block-size-is-invalid-or-too-large-when-using-flume-and-twi/36189152#36189152
	
	3. Use Cloudera JSONSerDe hive-serdes-1.0-SNAPSHOT.jar
	
	4. Both JAR files are build and tested on Cludera Hadoop Distribution CDH3 v0.3.7, 
	   for other target systems, user can compile and built JAR files on target system using maven3, 
	   for details see <b>Annexure-A</b> and <b>Annexure-B</b>

		a. flume-sources-1.0-SNAPSHOT.jar, 
		b. hive-serdes-1.0-SNAPSHOT.jar 

	5. Further reading:
		a. https://blog.cloudera.com/blog/2012/09/analyzing-twitter-data-with-hadoop/
		b. http://blog.cloudera.com/blog/2012/10/analyzing-twitter-data-with-hadoop-part-2-gathering-data-with-flume/
		c. http://blog.cloudera.com/blog/2012/11/analyzing-twitter-data-with-hadoop-part-3-querying-semi-structured-data-with-hive/

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

* Download JAR file `flume-sources-1.0-SNAPSHOT.jar` available under [/lib/](../tree/master/lib) folder to `$HOME/Desktop/hadoop-Use-Cases/twitter-Analysis/`
 
* create plugind.d & other sub-directories
	
		/usr/lib/flume-ng/plugins.d/twitter-streaming/lib/
		/var/lib/flume-ng/plugins.d/twitter-streaming/lib

* Copy `flume-sources-1.0-SNAPSHOT.jar` to following directories

		$ cd $HOME/Desktop/hadoop-Use-Cases/twitter-Analysis 
		$ sudo cp flume-sources-1.0-SNAPSHOT.jar /usr/lib/flume-ng/plugins.d/twitter-streaming/lib/
		$ sudo cp flume-sources-1.0-SNAPSHOT.jar /var/lib/flume-ng/plugins.d/twitter-streaming/lib/

* See **Annexure-A**; If want to built & Compile `flume-sources-1.0-SNAPSHOT.jar`
	




