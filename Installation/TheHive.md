# <img align="center" src="https://seeklogo.com/images/E/elastic-beats-logo-02512BDFD2-seeklogo.com.png" height="40px" width="30px">&nbsp; The Hive
For this lab, We'll be running Version: 4.1.24-1 of The Hive. For more information on The hive and see the offical documentation [link](https://docs.thehive-project.org/thehive/)
<br>

## Introduction
The Hive is a open source security incidient response flatform that will be our central point of handling and mediating alerts coming through from elastic. Its intergration with MISP and Cortex make it a good choice for us to use as a free software.

## <div id="installation">💻 Installation
The hive contains some pre-requirements that the application uses to run. Thse are Java and cassandra which we'll need to install first to get started.

### Java
TheHive only supports Java 8 running on its primary node. To download/install java 8, run the following command:
```bash
apt-get install -y openjdk-8-jre-headless
echo JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64" >> /etc/environment
export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
```
### Cassandra 
TheHive required version 3.11.x of Cassandra for it to run. To download/install Cassandra, run the following command:
```bash
curl -fsSL https://www.apache.org/dist/cassandra/KEYS | sudo apt-key add -
echo "deb http://www.apache.org/dist/cassandra/debian 311x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
sudo apt update
sudo apt install cassandra
```

### TheHive
To donwload/install version 4.X of TheHive, run the following command:
```bash
curl https://raw.githubusercontent.com/TheHive-Project/TheHive/master/PGP-PUBLIC-KEY | sudo apt-key add -
echo 'deb https://deb.thehive-project.org release main' | sudo tee -a /etc/apt/sources.list.d/thehive-project.list
sudo apt-get update
sudo apt-get install thehive4
```

<br><br>

## <div id="configurations">⚙️ Configurations
There are some configurations changes we need to make to get TheHive up and running. Thse include setting up Cassandra to run on our localhost and adding some security. TheHive also require space for indexing and storage that will need to be made for TheHive to run.

### Cassandra 
First thing is changing the cluster name, run the following command to get into the cqlsh console where we can make the changes:
```bash
cqlsh localhost 9042
```
Once in the cqlsh console, run the following command to change the cluster name:
```bash
cqlsh> UPDATE system.local SET cluster_name = 'thp' where key='local';
```
Exiting out of the cqlsh console and run the following command to flush the memtable data to disk:
```bash
nodetool flush
```
<br>

Next up we need to go to the `/etc/cassandra/cassandra.yaml` file to configure cassandra further.
```yml
cluster_name: 'thp'
authenticator: PasswordAuthenticator
authorizer: CassandraAuthorizer
```
These changes will add credentials as mandatory to connect to our cassandra database. Next 


## <div id="startup">🚀 Startup

## Conclusion 
