<h1 align=center><img align="center" src="https://www.elastic.co/apple-icon-57x57.png" height="45px" width="45px">&nbsp; Fleet Server</h1>

## :books: Table Of Content
 - [Installation](#installation)
   - [Java](#java)
   - [Elasticsearch](#elasticsearch)
   - [cortex](#cortex)
 - [Configurations](#configurations)
   - [Elasticsearch](#elasticsearch)
   - [cortex](#cortex)
   - [Analyzers & Responders](#analyzers--responders) 
 - [Startup](#startup)
   - [Elasticsearch](#elasticsearch)
   - [cortex](#cortex)
   - [Account setup](#account-setup)
   - [Analyzers & Responders](#analyzers--responders)
   - [Security](#security)
   - [Nginx Install](#nginx-install)
   - [Self-Signed Certificates](#self-signed-certificates)
   - [Nginx Configuration](#nginx-configuration)
  
<br>

## <img align="center" src="https://files.softicons.com/download/social-media-icons/free-social-media-icons-by-uiconstock/png/512x512/AWS-Icon.png" height="33px" width="33px">&nbsp;  AWS
To kick off, the first step is to launch a virtual machine (VM) to host our Cortex services. In this guide, I'll be using an AWS EC2 Ubuntu instance for this purpose. However, feel free to choose any cloud or on-premises service that suits your preferences. If you're unfamiliar with setting up a VM on AWS, you can follow a step-by-step walkthrough provided [here](./aws).

<br>

## <div id="installation">💻 Fleet Sever Setup
Open up your elastic webpage and navage down to Management > Fleet > Settings.

### Output 
Since we are running a single node cluster we can use our Elasticsearch service that we already have running on our host as out output. 
While in fleet settings, scroll down to output and click the pencil icon to edit the defualt elasticsearch configurations.
1. Change the ***Name*** to Elasticsearch
2. Set ***Type*** to Elasticsearch
3. Host can be set to Https://localhost:9000
4. ***Advance YAML Configurations*** enter the following
   ```yml
   ssl.certificate_authorities:
      - //CA Certificate in Raw Form 
   ```
5. Once complete click the ***Save and Apply Settings***
<br>

### Add Fleet Server
To add our fleet server, while still in the fleet settings click the ***Add Fleet Server*** button which will open up the configuration for your fleet server you want to add.
1. Click on ***Advance*** to use the advance configurations.
2. Under Select a policy for Fleet Server tick ***Collect System logs and metrics*** and then clik ***Create Policy*** to create your fleet server policy.
3. Set ***Choose a deployment mode for security*** to ***Production***.
4. Now under Add your Fleet Server Host set ***Name*** to Fleet and ***URL*** to https://<IP-FleetServer>:8220 then click ***Add host***.
5. Click on ***Generate Service Token*** to generate your service token.
6. Elastic should provide your the set of command to run attach your fleet server to your elastic host. The command will need to be modified slightly before running on your host.
   ```bash
   curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.11.3-linux-x86_64.tar.gz
   tar xzvf elastic-agent-8.11.3-linux-x86_64.tar.gz
   cd elastic-agent-8.11.3-linux-x86_64
   sudo ./elastic-agent install \
     --fleet-server-es=https://172.31.8.55:9200 \
     --fleet-server-service-token=<SERVICE_TOKEN> \
     --fleet-server-policy=fleet-server-policy \
     --certificate-authorities=<PATH_TO_CA> \
     --fleet-server-es-ca=<PATH_TO_CA_ES_CERT>
     --fleet-server-cert=<PATH_TO_FLEET_SERVER_CERT> \
     --fleet-server-cert-key=<PATH_TO_FLEET_SERVER_CERT_KEY> \
     --fleet-server-port=8220
   ```
7. Store this bash command in file for now to make it easier to edit later, `elastic-fleet.sh`.
<br>

### Self-Signed Certificates
Since we're running our elastic instance over TLS to ensure secure communication, we'll need to make up a set of certificates for our fleet server as well.
1. While on our elastic host, run the following command to create a set to certificates for our fleet host:
   ```bash
   usr/share/elasticsearch/bin/elasticsearch-certutil cert --out /etc/elasticsearch/certs/fleet.zip --name fleet --ca-cert /etc/elasticsearch/certs/ca/ca.crt --ca-key /etc/elasticsearch/certs/ca/ca.key --ip 
   <YOUR_HOST_IP> --pem
   
   # cert - generate X.509 certificates and keys
   # --out - Output path
   # --name - Name of our certificate 
   # --ca-cert - Path to our ca.crt
   # --ca-key - Path to our ca.key
   # --ip - Ip of host that will be using the certificate
   # --DNS - DNS name of host that will be using the certificate (Do not need in this instance as we have a IP)
   # --pem - Is the file format to make a our certificate and private key
   ```
2. Unzip your `fleet.zip` file using the following command:
   ```bash
   unzip fleet.zip
   ```
3. We also need to enclude our CA certificate in our folder as well before we transfer it.
   ```bash
   cp -r source_directory destination_dirctory
   ```
4. Now that we have our fleet server's certificates made we'll need to transfer them over to our fleet host from our elastic host. For this we'll be using the Secure Copy Protocol
   ```bash
   scp \ # Secure Copy Protocol is the most command and secure way of transfering file on linux host
   -i /path/to/private_key.pem \ # My host use private keys for auth 
   -r CA_FLEET_FOLDER \ # File that you want tranfered
   username@remote_host_ip:/path/to/destination # Host you want to tranfer the file to and the destination in the host 
   ```
<br>

### Update Machine
Ensure that before we run our installation script that our host is up to date:
```bash
Sudo apt update && sudo apt upgrade -y
```
<br>

### Run Install Script
Going back to our fleet server host, navagate over where you stored `elastic-fleet.sh` file. We can now add in the details to our Certificates we just transfered over to our host.
Once completed we can now run our script `./elastic-fleet.sh` which will install elastic-agent and enroll the host as our fleet server in our elastic instance.
Going back to our elastic page, we should now see our fleet server now popup as one of our agents.


