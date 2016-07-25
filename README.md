# Twitter-Data-Analysis-Using-Flume-Hive
Store live streaming Tweeter data in HDFS using Apache flume and load this data in Hive for analysis. Example use Cloudera Hadoop Distribution CHD3.

Target System:
	01. Ubuntu v10.10
	02. CLoudera Hadoop Distribution CDH3 v0.3.7
	02. Hadoop v0.20.2-cdh3u0 in Pseudo-distribution mode
	03. Java v1.6.0_24
	04. Hive v0.7.0
	05. Flume v1.6.0

Note:
	01. Code is tested only on Cloudera Hadoop Distribution CDH3. 
	02. "flume-sources-1.0-SNAPSHOT.jar", "hive-serdes-1.0-SNAPSHOT.jar" pre build JAR are attached.

Pre-requesites:
	01. All deamons are running.
	02. Java, Flume and Hive is installed & properly configured.
	03. metastore.db using Java 'Derby' in Embedded mode is configured.
	04. Knowledge of Hadoop HDFS, Hive and Flume.

Issues:
	01. Avro block size is invalid or too large when using Flume and Twitter streaming
		Java -jar avro-tools-1.7.7.jar tojson FlumeData.1468209072594
		http://stackoverflow.com/questions/30661478/unable-to-correctly-load-twitter-avro-data-into-hive-table

	02. TwitterAgent.sources.Twitter.type = org.apache.flume.source.twitter.TwitterSource
		Twitter 1% Firehose Source
		This source is highly experimental. It connects to the 1% sample Twitter Firehose using streaming API and continuously downloads tweets, 
		converts them to Avro format, and sends Avro events to a downstream Flume sink
		
	03. JSONSerDe compatibility.
