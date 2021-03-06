# Spark Development Environment for XUbuntu 14.10
This environment is composed on the following elements:
 * the main development will be performed in a VirtualBox VM running XUbuntu 14.10
    - the local versions of some services, like HBase and Kafka, will run in XUbuntu, as these services are difficul to run in windows
    - due to problems communicating the Windows host with the guest XUbuntu, finally development with Eclipse and Anaconda Spyder will also be performed in the VM. Anyway a Linux environment is more confortable for developing

 * a EMR cluster launched by some Fabric scripts from the XUbuntu VM will be used to run the jobs in a cluster, for the final acceptance and performance tests

## Components and versions:
    * Spark 1.3
    * Scala 2.10: required by Spark
    * Python 2.7: required by Spark
    * Apache Kafka kafka_2.10_0.8.1.1: required by Spark
 
## Network Setup for VirtualBox
First of all install the ssh server (package openssh-server) in the VM. Then following http://stackoverflow.com/questions/5906441/how-to-ssh-to-a-virtualbox-guest-externally-through-a-host I take the option of using the NAT network configuration for VirtualBox, and then in Configuration -> Networking -> Port forwarding we add a new rule with protocol = TCP, host port = 3022 and guess port = 22. This forwards petitions to host:3022 to the VM:22. Hence we can connect to the VM by ssh from the Windows host with:

```bash
C:\Users\bc24u_000\Sistemas>ssh -p 3022 juanrh@localhost
The authenticity of host '[localhost]:3022 ([127.0.0.1]:3022)' can't be established.
...
juanrh@localhost's password:
Welcome to Ubuntu 14.10 (GNU/Linux 3.16.0-33-generic x86_64)
```

This is enough as most of the time we'll use the VM directly, and ssh is enough for moving data from the host to the guest, and viceversa

## Scripts for the development environment
Fabric is used for automation
 * Run `install_prerequisites.sh` to setup Fabric
 * Run `install_devenv.sh` to install the development environment

 For now all the Fabric tasks are defined in the single file `fabfile.py`

## Scala Environment for Spark
### Version of Scala
Spark is compiled for Scala 2.10, but 1) Scala IDE 4 is distributed with Scala 2.11, and 2) Scala IDE can only
have a single version of Scala installed (http://scala-ide.org/docs/current-user-doc/faq/index.html "Currently,
it is not possible to install more than one Scala IDE plugin within the same Eclipse installation"), and 3)
we cannot run a binary for Scala 2.10 in the Scala 2.11 runtime (https://groups.google.com/forum/#!topic/scala-user/mUZSYgCLigA).
Hence we should use Scala IDE 3.0.3 (http://scala-ide.org/download/prev-stable.html careful, Scala IDE 3.0.4 is
for Scala 2.11) which is the last version of the plugin for Scala 2.10. This works for Eclipse Kepler

### Configure Scala IDE
- Import project as a Maven project
- FIX for Scala IDE 4: this is not needed for Scala IDE 3.0.3, but for Scala IDE 4 we cannot execute the
code because the runtime is Scala 2.11. In project properties -> Scala Compiler select "use project settings" and "Scala Installation" to
"Fixed Scala Installation: 2.10.4 (built-in)" 

### Configure Eclipse for Sun Java 7 
In Ubuntu, after installing the sun JDK with the corresponding task of the fabfile in this directory we have to modify eclipse.ini as indicated [here](https://wiki.eclipse.org/Eclipse.ini) in order to use that JDK. For that we use `-vm`, that needs to occour before `-vmargs`, for example:

```bash
....
--launcher.appendVmargs
-vm 
/opt/java7/jdk1.7.0_75/bin/java
-vmargs
-Dosgi.requiredJavaVersion=1.6
...
```

## Kafka maintenance for development in local mode

### Deleting a Kafka topic
Deleting a topic is not recommended, this is more for development purposes, in production the idea is letting the TTL for the topic arrive to Kafka.
For Kafka 0.8.1.1 we can use http://stackoverflow.com/questions/24287900/delete-topic-in-kafka-0-8-1-1 we can do it:

 * From the command line utils that come with Kafka:
```bash
bin/kafka-run-class.sh kafka.admin.DeleteTopicCommand --zookeeper localhost:2181 --topic test
```
 
 * With the Java / Scala API
```java
ZkClient zkClient = new ZkClient("localhost:2181", 10000);
zkClient.deleteRecursive(ZkUtils.getTopicPath("test2"));
```
 
### Purging a Kafka Installation
For development, as seen in http://stackoverflow.com/questions/16284399/purge-kafka-queue we can:
 * Stop the Apache Kafka daemon
 * Delete the topic data folder: rm -rf /tmp/kafka-logs/MyTopic-0
 * Delete the topic metadata: zkCli.sh then rmr /brokers/MyTopic
 * Start the Apache Kafka daemon

