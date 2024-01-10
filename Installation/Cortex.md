# <img align="center" src="https://thehive-project.github.io/Cortex-Analyzers/images/cortex-logo.png" height="45px" width="45px">&nbsp; Cortex

## <div id="installation">💻 Installation
Cortex require certain package so that the application can run, install the following packages
```bash
apt install wget gnupg apt-transport-https git ca-certificates ca-certificates-java curl  software-properties-common python3-pip lsb_release
```

### java
```bash
apt install -y openjdk-11-jre-headless
echo JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64" >> /etc/environment
export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
```
<br>

### Elasticsearch 
```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch |  sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" |  sudo tee /etc/apt/sources.list.d/elastic-7.x.list 
sudo apt install elasticsearch
```
<br>

### cortex
```bash
wget -O- "https://raw.githubusercontent.com/TheHive-Project/Cortex/master/PGP-PUBLIC-KEY"  | sudo apt-key add -
wget -qO- https://raw.githubusercontent.com/TheHive-Project/Cortex/master/PGP-PUBLIC-KEY |  sudo gpg --dearmor -o /usr/share/keyrings/thehive-project.gpg
echo 'deb https://deb.thehive-project.org release main' | sudo tee -a /etc/apt/sources.list.d/thehive-project.list
apt install cortex
```
<br><br>

## <div id="configurations">⚙️ Configurations
### Elastic
To configure elasticsearch to run on our localhost so that cortex can connect to its services, we need to edit the configuration file `/etc/elasticsearch/elasticsearch.yml` as follows:
```yml
http.host: 127.0.0.1
transport.host: 127.0.0.1
cluster.name: hive
thread_pool.search.queue_size: 100000
path.logs: "/var/log/elasticsearch"
path.data: "/var/lib/elasticsearch"
xpack.security.enabled: false
script.allowed_types: "inline,stored"
```

### Cortex
In order to connect our elasticsearch to our elasticsearch service, we'll need to make some changes to the configuration `/etc/cortex/application.conf` file as follows:
```conf
## ElasticSearch
search {
  # Name of the index
  index = cortex
  # ElasticSearch instance address.
  uri = "http://127.0.0.1:9200"
}
```


## <div id="startup">🚀 Startup
