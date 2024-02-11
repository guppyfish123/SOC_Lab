# <img align="center" src="https://www.elastic.co/apple-icon-57x57.png" height="45px" width="45px">&nbsp; Fleet Server
## :books: Table Of Content
 - [Fleet Sever Setup](#fleet-sever-setup)
   - [Output](#output)
   - [Add Fleet Server](add-fleet-server)
   - [Self-Signed Certificates](#self-signed-certificates)
   - [Update Machine](#update-machine)
   - [Run Installation Script](#run-installation-script)
 - [Window Agent setup](#window-agent-setup)
   - [Agent Policy](#agent-policy)
   - [Add Intergration](#add-intergration)
   - [Add Windows Agent](#add-windows-agent)
   - [Self Signed Certificate](#self-signed-certificate)
   - [Install Agent](#install-agent)

<br>

## <img align="center" src="https://files.softicons.com/download/social-media-icons/free-social-media-icons-by-uiconstock/png/512x512/AWS-Icon.png" height="33px" width="33px">&nbsp;  AWS
To kick off, the first step is to launch a virtual machine (VM) to host our Cortex services. In this guide, I'll be using an AWS EC2 Ubuntu instance for this purpose. However, feel free to choose any cloud or on-premises service that suits your preferences. If you're unfamiliar with setting up a VM on AWS, you can follow a step-by-step walkthrough provided [here](./aws).

<br>

## <div id="fleet-sever-setup">💻 Fleet Sever Setup
Open up your elastic webpage and navigate down to Management > Fleet > Settings.

### Output 
Since we are running a single node cluster we can use our Elasticsearch service that we already have running on our host as out output. 
While in fleet settings, scroll down to output and click the pencil icon to edit the default elasticsearch configurations.
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
2. Under Select a policy for Fleet Server tick ***Collect System logs and metrics*** and then click ***Create Policy*** to create your fleet server policy.
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
3. We also need to include our CA certificate in our folder as well before we transfer it.
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

### Run Installation Script
Going back to our fleet server host, navigate over where you stored `elastic-fleet.sh` file. We can now add in the details to our Certificates we just transferred over to our host.
Once completed we can now run our script `./elastic-fleet.sh` which will install elastic-agent and enrol the host as our fleet server in our elastic instance.
Going back to our elastic page, we should now see our fleet server now popup as one of our agents.

<br><br>

## <div id="window-agent-setup">⚙️ Window Agent Setup 
Now that we have our Elastic Fleet server up and running we can start to integrating our endpoints (agents).
<br>

### Agent Policy 
A policy is terms of Elastic Fleet is a collection of configurations and settings for the type of log data you want to collect from your agents. Making it easier for integrating large volumes of endpoints that you want to have configured the same.
<br>
1. To make our first fleet policy, open up your elastic web page and navigate to your ***Fleet*** management tab. Under ***Agent Policies*** click on the ***Create Agent Policy*** button.
2. Name your policy ***Windows-Agent-Policy*** and tick ***Collect System Logs and Metrics***
3. Under Advance options add a description of ***Core Windows Agent Policy***
4. ***Default Namespace*** set to ***Windows*** and remove ***default***
5. Tick both ***Collect Agent Logs*** & ***Collect Agent Metrics*** under ***Agent Monitoring***
6. Enable ***Agent Tamper Protection***
7. Leave all other options as default and click ***Create Agent Policy***
<br>

### Add Integration 
Opening up your newly created Agent Policy we will now need to add some integrations to our policy depending on what exactly we want to capture and report on our endpoints.
As we are integration a windows endpoint these are the integration options we'll be using:
- Elastic Defend
- Windows
The default configurations for both of these integrations will do for now and can be changed later if needed.
<br>

### Add Windows Agent
After creating our windows policy we can now add our windows endpoint as an agent of our elastic fleet. 
1. In the ***Agent*** tab in ***Fleet***, click ***Add Agent***
2. Select the policy that we just made under ***What type of host are you adding?***
3. Set ***Enrol in Fleet?*** as ***Enrol in Fleet***
4. Under ***Install Elastic Agent on your host*** elastic should have a preconfigured command to enrol you windows endpoint to fleet.
 ```bash
 curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.11.3-linux-x86_64.tar.gz
 tar xzvf elastic-agent-8.11.3-linux-x86_64.tar.gz
 cd elastic-agent-8.11.3-linux-x86_64
 sudo ./elastic-agent install --url=https://172.31.13.161:8220 \
 --enrollment-token=ejRYQXpvd0JiTmhaSmhueGxONkc6WTg2LVUwN0NScHVGcnBudEx0bWEwUQ== \
 --certificate-authorities=<PATH_TO_CA> \
 --fleet-server-es-ca=<PATH_TO_CA_ES_CERT>
 ```
5. Over on your windows host make a file called `fleet-agent-install.sh` and paste the code into the file to be edited for later
<br>

### Self Signed Certificate
As we are running our elastic instance over self signed certificates to secure lab. We'll need to copy over a instance of our CA certificate onto our window host to use.
<br>

### Install Agent 
Now that we have our CA certificate on our windows host we can edit our `fleet-agent-install.sh` file with our location of our certificate. 
Once completed we can run our script to install/setup our windows agent with our fleet server `./fleet-agent-install.sh`.
Back over on our elastic webpage we should now be able to see our new window agent on our fleet ***agent*** listings.

