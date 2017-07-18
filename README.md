#NYC-Taxi Demo
##About
The nyc-taxi demo is a demonstration of some of the features of the Geowave service. It takes in a set of information from the NYCTLC (New York City Taxi & Limousine Commission) and outputs how long a taxi trip from one chosen point on the map to another would be based on the data set.

The NYCTLC trip data can be find [here.](http://www.nyc.gov/html/tlc/html/about/trip_record_data.shtml)

This is a subset of the demo-able application that can be found in full at http://github.com/radiantbluetechnologies/geowave-demos
This portion of the application was used for research purposes.


##Installing the Project
After pulling down the repository, cd into the **/nyc-taxi/** directory and run 
```
$ mvn package
```
This will resolve all of the dependencies and build the target/geowave-format-0.9.5-SNAPSHOT.jar. 

##Ingesting Data
Assuming that the appropriate software stack (Hadoop, Zookeeper, and Accumulo and/or HBase) has been successfully setup, the following steps will show how to ingest a set of data.

To install the Geowave command line, follow the instructions [here](https://locationtech.github.io/geowave/documentation.html#installation-from-rpm) to install Geowave 0.9.5.

Next, take the **geowave-format-0.9.5-SNAPSHOT.jar** file that you've built in the **target/** directory and drop it into the **/usr/local/geowave/tools/plugins/** directory that should have just been created on the RPM install. This will allow the service to properly ingest the nyctlc data. 

To actually ingest the data, first, create a directory in which you can keep all data that you wish to ingest. something such as **~/ingest/** is fine. Put the .csv file(s) in there.

The next step is to configure a backend datastore and an index so that the data can be read and interpreted. Configure a store by using the `geowave config store` command such as this example configuring a connection to accumulo:
```
$ geowave config addstore -t accumulo nycstore \ 
      --gwNamespace ANY_NAME_USED_AS_TABLE_PREFIX \ 
      --zookeeper ZOOKEEPER_HOSTNAME:2181 \
      --instance ACCUMULO_INSTANCE_NAME \
      --user USERNAME \
      --password PASSWORD (or ignore password and it will prompt on stdin)
```
* When entering in the gwNamespace name ensure that the name does not have a hyphen '-' in it.

Next, create an index using the following command:
```
$ geowave config addindex -t INDEX_TYPE ANY_NAME
```
There is a `spatial` index type, and `spatial_temporal` index type that exist as part of the core geowave project.  This project adds `multigeo-multitime`, `multigeo-timerange` (which uses a tiering approach for the time range), and `multigeo-timerange-xz` (which uses an XZ-order space filling curve to handle the time range).  An example of configuring an index with one of these index types is:
```
$ geowave config addindex -t multigeo-multitime my_mgmt
```
Finally, run the ingest call on the data with:
```
$ geowave ingest localtogw ./ingest nycstore my_mgmt \ 
	  -f nyctlc
```

At this point the data should be ingested.

To run queries to measure the performance, there is a `query` command that this project provides as a plugin to geowave and will be avaiable through the `geowave` command. An example of running a query experiment is:
```
$ geowave nyctlc query my_mgmt \ 
	  -pw -73.575 -pe -74.375  -ps 40.3 \
	  -pn 41.1 -dw -73.931 -de -73.921 \
	  -ds 40.826 -dn 40.835 -pst 2009-04-05-00:00:00 \
	  -pet 2009-11-04-23:59:59 -dst 2009-04-05-00:00:00 \
	  -det 2009-11-04-23:59:59
```

