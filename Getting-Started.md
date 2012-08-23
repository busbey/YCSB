YCSB is a framework for benchmarking systems. By itself, it is not particularly useful; only when you add code to interface with a data serving system is it useful. The current YCSB release (0.1.4) contains code to interface with the following systems:

- [Cassandra](http://cassandra.apache.org/)
- [DynamoDB](http://aws.amazon.com/dynamodb/)
- [VMware vFabric GemFire](http://www.vmware.com/products/application-platform/vfabric-gemfire/overview.html)
- [GigaSpaces XAP](http://www.gigaspaces.com/xap)
- [HBase](http://hbase.apache.org/)
- [Infinispan](http://www.jboss.org/infinispan)
- [JDBC](http://www.oracle.com/technetwork/java/javase/jdbc/index.html)
- [MapKeeper](https://github.com/m1ch1/mapkeeper)
- [MongoDB](http://www.mongodb.org/)
- [Oracle NoSQL Database](http://www.oracle.com/technetwork/database/nosqldb/overview/index.html)
- [Redis](http://redis.io/)
- [Voldemort](http://project-voldemort.com/)

It is straightforward to interface with other database systems - see [[Adding a Database]].

## 1. Obtain YCSB

Download the latest version:

    wget https://github.com/downloads/brianfrankcooper/YCSB/ycsb-0.1.4.tar.gz
    tar xfvz ycsb-0.1.4.tar.gz
    cd ycsb-0.1.4

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