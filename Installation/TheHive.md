# <img align="center" src="https://thehive-project.org/assets/ico/favicon.ico" height="45px" width="45px">&nbsp; The Hive
For this lab, We'll be running Version: 4.1.24-1 of The Hive. For more information on The hive and see the offical documentation [link](https://docs.thehive-project.org/thehive/)
<br><br>

## :books: Table Of Content
 - [Installation](#installation)
   - [Java](#java)
   - [Cassandra](#cassandra)
   - [TheHive](#thehive)
 - [Configurations](#configurations)
   - [Cassandra](#cassandra)
   - [Indexing Engine](#indexing-engine)
   - [TheHive](#thehive)
   - [File Storage](#file-storage)
   - [Security](#security)
 - [Startup](#startup)
   - [Cassandra](#cassandra)
   - [TheHive](#the-hive)

<br>

## <img align="center" src="https://files.softicons.com/download/social-media-icons/free-social-media-icons-by-uiconstock/png/512x512/AWS-Icon.png" height="33px" width="33px">&nbsp;  AWS
To kick off, the first item to action is to launch a virtual machine (VM) to host our Hive services. In this guide, I'll be using an AWS EC2 Ubuntu instance for this purpose. However, feel free to choose any cloud or on-premises service that suits your preferences. If you're unfamiliar with setting up a VM in AWS, you can follow a step-by-step walkthrough provided [here](./aws).
<br>

## <div id="installation">💻 Installation
### Update Machine
Before diving into Cortex installation, let's ensure our machine is up to date:
```bash
Sudo apt update && sudo apt upgrade -y
```
<br>

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
echo "deb http://archive.apache.org/dist/cassandra/debian 311x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
sudo apt update
sudo apt install cassandra
```

### TheHive
To download/install version 4.X of TheHive, run the following command:
```bash
curl https://raw.githubusercontent.com/TheHive-Project/TheHive/master/PGP-PUBLIC-KEY | sudo apt-key add -
echo 'deb https://deb.thehive-project.org release main' | sudo tee -a /etc/apt/sources.list.d/thehive-project.list
sudo apt-get update
sudo apt-get install thehive4
```

<br><br>

## <div id="configurations">⚙️ Configurations

To get TheHive up and running, several configuration changes are required. These include configuring Cassandra to run on our localhost and implementing security measures. TheHive also demands sufficient space for indexing and storage, which needs to be provisioned for its proper functioning.

### Cassandra

Begin by changing the cluster name. Access the cqlsh console using the following command:

```bash
cqlsh localhost 9042
```
Exit the cqlsh console and flush the memtable data to disk:
```bash
cqlsh> UPDATE system.local SET cluster_name = 'thp' where key='local';
```
Exiting out of the cqlsh console and run the following command to flush the memtable data to disk:
```bash
nodetool flush
```
<br>

Next, navigate to the /etc/cassandra/cassandra.yaml file to make further configurations:
```yml
cluster_name: 'thp'
authenticator: PasswordAuthenticator
authorizer: CassandraAuthorizer
```
Leave all other parameters as default for now. These changes enforce credentials as mandatory for connecting to our Cassandra database, enhancing security.
<br>

Now, set up a new user to avoid using the default credentials.
1. Connect to cqlsh using the following command:
  ```bash
  cqlsh -u cassandra -p cassandra
  ```
2. Now that we are in as the super user we can make a new user by running the following command while in the cqlsh console:
  ```bash
  #Replace `username` & `password` with your own
  CREATE ROLE username with SUPERUSER = true AND LOGIN = true and PASSWORD = 'password';
  ```
3. Exit cqlsh and connec to the cqlsh again but this time with your new credentials that have just made:
  ```bash
  cqlsh -u username -p password
  ```
4. To confirm that we have made our new account as a superuser and to view all other account, run:
  ```bash
  LIST ROLES;
  ```
5. Delete the default cassandra account with the following command:
  ```bash
  DROP ROLE cassandra;
  ```
6. Verify that the cassandra account has now been deleted
  ```bash
  LIST ROLES;
  ```
<br>

### Indexing Engine
TheHive relies on an indexing engine for its operation. For standalone servers, such as in this lab, TheHive is equipped with a built-in local Lucene engine that we'll utilize.
Create a dedicated folder for TheHive to host its indexes with the following commands:
```bash
# Create a new directory called index
mkdir /opt/thp/thehive/index

# Grant permissions to the folder for TheHive to access
chown thehive:thehive -R /opt/thp/thehive/index
```
<br>

### File Storage
For proper functionality, TheHive requires a designated storage location to store files uploaded for task logs or observables. In the case of a standalone server, like the one in this lab, we can utilize the local system for file storage.
To achieve this, create a dedicated directory for TheHive to manage these files using the following commands:
```bash
# Create a new directory called files
mkdir -p /opt/thp/thehive/files

# Grant permissions to the folder for TheHive to access
chown -R thehive:thehive /opt/thp/thehive/files
```
<br>

### The Hive
With all the directory made now and our Cassandra database up and running, we just need to connect it all to TheHive now.

**Cassandra**<br>
We can connect our Cassandra database to TheHive in the configuration file `/etc/thehive/application.conf` to be the following 
```conf
db {
  provider: janusgraph
  janusgraph {
    storage {
      backend: cql
      hostname: ["127.0.0.1"] # seed node ip addresses
      #username: "<cassandra_username>"       # login to connect to database 
      #password: "<cassandra_passowrd"
      cql {
        [...]
      }
    }
  }
}
```
<br><br>

### Security
Leaving passwords in plain text is considered bad practice, and for improved security, we can implement a Key Vault system. In this scenario, we will use AWS's Secrets Manager and the aws-cli to retrieve secrets securely during each session. A detailed tutorial on setting up AWS Secret Manager with your EC2 Instance is available [here](./aws).
<br><br>
Once your secret is set up, use the following command to install awscli:
```bash
sudo apt-get install awscli
```
To configure AWS on your host, run the following command:
```bash
aws configurations
# Just leave all input blank and enter through them all
```
<br>

Now, to retrieve the secret you have set up from AWS Secret Manager, use the following command:
```bash
# Replace your_secretID & your_region with your own 
aws secretsmanager get-secret-value --secret-id your_secretID --region your_region
```
The output will look similar to this JSON structure:
```json
{
    "ARN": "arn:aws:secretsmanager:ap-southeast-2342:234234:secret:secret-2342",
    "Name": "secret-name",
    "VersionId": "234467-7f9d-4361-8517-fs98234290",
    "SecretString": "{\"username\":\"your_username\",\"password\":\"your_password\"}",
    "VersionStages": [
        "AWSCURRENT"
    ],
}
```
<br>

To extract the username and password for use in TheHive's configuration file, create a `.env` file in the `/etc/thehive` directory with the following content:
```bash
# Replace your_secretID & your_region with your own 
CASSANDRA_USER=$(aws secretsmanager get-secret-value --secret-id your_secretID --region your_region | jq -r '.SecretString | fromjson | .username')
CASSANDRA_PASSWORD=$(aws secretsmanager get-secret-value --secret-id your_secretID --region your_region | jq -r '.SecretString | fromjson | .password')
```
Assign these variables in the `/etc/thehive/application.conf` file as follows:
```conf
db {
  provider: janusgraph
  janusgraph {
    storage {
      backend: cql
      hostname: ["127.0.0.1"] # seed node ip addresses
      #username: "${CASSANDRA_USER}"   
      #password: "${CASSANDRA_PASSWORD}"
      cql {
        [...]
      }
    }
  }
}
```
<br><br>

**Indexes**<br>
To configure our index search on our `/etc/thehive/application.conf` with our Lucene engine directory that we made, update the following:
```conf
db {
  provider: janusgraph
  janusgraph {
    storage {
      [..]
    }
    ## Index configuration
    index.search {
      backend : lucene
      directory:  /opt/thp/thehive/index
    }
  }
}
```

**Filesystem**<br>
Lastly to add our local file system directoty we made for TheHive by changing the following lines:
```
## Storage configuration
storage {
provider = localfs
localfs.location = /opt/thp/thehive/files
}
```



<br><br>

## <div id="startup">🚀 Startup
###Cassandra
Cassandra should already be running from when we installed it. We need to restart the service to apply to changes that we made earlier.
```bash
sudo systemctl restart cassandra 
```
Now to confirm that the Cassandra service is running use the following command.
```bash
sudo systemctl status cassandra
```
In the even that Cassandra fails to start, you can check the logs for any error messages with:
```bash
journalctl -u cassandra -n 100
```
To make the cassandra service start on boot, run the following:
```bash
sudo systemctl daemon-reload
sudo systemctl enable cassandra
```
<br>

### The Hive
The Hive should now be ready to start with everything configured and Cassandra up and running. Use the following command to start TheHive:
```bash
sudo systemctl start thehive
```
To check that TheHive is up and running you can use:
```bash
sudo systemctl status thehive
```
In the even that thehive fails to start, you can check the logs for any error messages with:
```bash
journalctl -u thehive -n 100
# Or
vim /var/log/thehive/application.log
```
To make the thehive service start on boot, run the following:
```bash
sudo systemctl daemon-reload
sudo systemctl enable thehive
```
<br>
TheHive should now be reachable on your host `http://YOUR_SERVER_ADDRESS:9000/` with a login page. The default admin user is admin@thehive.local with password secret. Which should be change once you can login.

