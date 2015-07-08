YCSB is a framework for benchmarking systems. By itself, it is not particularly useful; only when you add code to interface with a data serving system is it useful. The current YCSB release (0.2.0) contains code to interface with the following systems:

- [Accumulo](https://accumulo.apache.org/)
- [Cassandra](http://cassandra.apache.org/)
- [Couchbase](http://www.couchbase.com/)
- [DynamoDB](http://aws.amazon.com/dynamodb/)
- [ElasticSearch](https://www.elastic.co/products/elasticsearch)
- [VMware vFabric GemFire](http://www.vmware.com/products/application-platform/vfabric-gemfire/overview.html)
- [HBase](http://hbase.apache.org/)
- [HyperTable](http://www.hypertable.com/)
- [Infinispan](http://www.jboss.org/infinispan)
- [JDBC](http://www.oracle.com/technetwork/java/javase/jdbc/index.html)
- [MongoDB](http://www.mongodb.org/)
- [OrientDB](http://www.orientdb.org/)
- [Redis](http://redis.io/)
- [Tarantool](http://tarantool.org/)


It is straightforward to interface with other database systems - see [[Adding a Database]].

## 1. Obtain YCSB

If you'll be running on Windows, please start by referencing [our prerequisites for Windows](https://github.com/brianfrankcooper/YCSB/wiki/Prerequisites-for-Windows).

Download the latest version:

    curl -O https://github.com/brianfrankcooper/YCSB/releases/download/0.2.0/ycsb-0.2.0.tar.gz
    tar xfvz ycsb-0.2.0.tar.gz
    cd ycsb-0.2.0

Or clone the git repository and build:

    git clone git://github.com/brianfrankcooper/YCSB.git
    cd YCSB
    mvn clean package

Systems may have additional requirements for running clients.  For example, HBase requires the client be able to contact Zookeeper.  See <A HREF="http://hadoop.apache.org/hbase/docs/r0.20.3/api/org/apache/hadoop/hbase/client/package-summary.html#package_description">HBase 0.20.3 Client Package Description</A> for HBase-specific instructions. Some details are listed in [[Using the Database Libraries]].

You will be using the `ycsb` command to interact with YCSB. Run:

    ./bin/ycsb

to see the usage. 

## 2. Now, run a workload

See [[Running a Workload]].