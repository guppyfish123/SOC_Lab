# <img align="center" src="https://www.elastic.co/apple-icon-57x57.png" height="45px" width="45px">&nbsp;&nbsp; Elastic Kibana 
The Kibana suite offers a comprehensive platform for data exploration and visualization. Kibana serves as the visualization layer, enabling users to effortlessly shape and navigate their data through intuitive interfaces, creating diverse visualizations, charts, and dashboards. Elasticsearch's robust search capabilities, combined with Kibana's visualization tools, empower users to extract meaningful insights, monitor data in real-time, and scale horizontally to accommodate growing data volumes. Together, Elasticsearch and Kibana provide a powerful solution for streamlining data management and gain actionable insights from their datasets.
<br><br>
   
## :books: Table Of Content
- [ElasticSearch](#elasticsearch)
   - [Installation](#installation)
   - [Configurations](#configurations)
   - [Startup](#startup)
- [Kibana](#kibana)
   - [Installation](#installation)
   - [Configurations](#configurations)
   - [Startup](#startup)
<br>

## <img align="center" src="https://files.softicons.com/download/social-media-icons/free-social-media-icons-by-uiconstock/png/512x512/AWS-Icon.png" height="33px" width="33px">&nbsp;  AWS
To kick off the Lab we first require a virtual machine (VM) to host our Cortex services. In this lab, I'll be utilizing AWS's EC2 Instances Service to run my VMs. However, feel free to choose any cloud or on-premises service that suits your preferences and budget. If you're unfamiliar with setting up a VM in AWS, you can follow a step-by-step walkthrough provided [here](./aws).
<br>

### Security groups
In AWS, managing security groups is crucial to controlling traffic coming into and from our host. Adhering to the principle of least privilege ensures that only the necessary ports are open that the host required in order to operation with. For detailed instructions on configuring security groups in AWS, refer to [GUIDE](./aws).
<br><br>

# <img id="elasticsearch" src="https://static-00.iconduck.com/assets.00/elasticsearch-icon-1839x2048-s0i8mk51.png" height="30px" width="30px">&nbsp; ElasticSearch
For this lab, we'll be using ElasticSearch Version 8.11, the current version as of 2023. Refer to the official ElasticSearch Documentation for more information [Link](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html).
<br>

## <div id="installation">💻 Installation
### Update Machine
Before diving into ElasticSearch installation, let's ensure our host is up to date:
```bash
Sudo apt update && sudo apt upgrade -y
```
<br>

### ElasticSearch Installation Steps
Now, let's proceed with installing ElasticSearch:
```bash
# Download and install the public signing key
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

# Install the Elasticsearch Debian package
sudo apt-get update && sudo apt-get install elasticsearch
```
With these commands, ElasticSearch is now up and running.
<br><br>

## <div id="configurations">⚙️ Configurations
Using the following command to open up the elasticsearch configurations file to make the follow changes to configure elasticsearch to run on our host:
```bash
sudo vim /etc/elasticsearch/elasticsearch.yml
```
> [!NOTE]
> We'll be using Vim as our text editor in this lab. If you prefer another text editor like Nano, feel free to use it. For a quick Vim tutorial, check [link](./vim.md)

<br>

Remove the # from the beginning of lines to uncomment and set the following parameters:
  - luster.name: {Name of your elastic cluster}
  - network.host: {Set to either a allocated IP, DNS name, or, for this instance, we'll set it to our VM's assigned IP by setting it to 0.0.0.0}
  - http.port: {Change from default 9200 for security reasons}
> [!IMPORTANT]
> Avoid using default ports to enhance security.
  - http.host: 0.0.0.0
<br>

### Self Signed Certificates 
To secure our Elastic Host and encrypt traffic over HTTPS, we'll set up Self-Signed Certificates using elasticsearch-certutil tool found in the `/usr/share/elasticsearch/bin` directory.

1. To make a CA certificate and key for our services, use command:
```bash
./elasticsearch-certutil ca --pem --out /etc/elasticsearch/certs/ca.zip

# ca - Generates a new local certificate authority
# --pem - Is the file format to make a our certificate and private key
# --out - Is our output path
```
2. Unzip the file:
```bash
sudo apt install zip
unzip ca.zip
```
3. Make our ElasticSearch host certificate:
```bash
./elasticsearch-certutil cert --out /etc/elasticsearch/certs/fleet.zip --name elastic --ca-cert /etc/elasticsearch/certs/ca/ca.crt --ca-key /etc/elasticsearch/certs/ca/ca.key --ip 172.31.13.161 --pem

# cert - generate X.509 certificates and keys
# --out - Output path
# --name - Name of our certificate 
# --ca-cert - Path to our ca.crt
# --ca-key - Path to our ca.key
# --ip - Ip of host that will be using the certificate
# --DNS - DNS name of host that will be using the certificate (Do not need in this instance as we have a IP)
# --pem - Is the file format to make a our certificate and private key
```
4.Unzip the file:
```bash
unzip elastic.zip
```
5. Change Permission to certs directory
```bash
sudo chown -R /etc/elastisearch/certs
```
<br>

Now, add the certificates to the `elasticsearch.yml` configuration file to enabled SSL encryption:
```yml
xpack.security.enabled: true
xpack.security.enrollment.enabled: true
xpack.security.http.ssl.enabled: true
xpack.security.transport.ssl.enabled: true
```
Add in the Self Signed Certificates:
```yml
xpack.security.http.ssl:
  enabled: true
  certificate: certs/elastic/elastic.crt
  key: certs/elastic/elastic.key
  certificate_authorities: certs/ca/ca.crt
```
Remove any truststore.path or keystore.path variables from the xpack.security.http.ssl parameter, as they are no longer needed. The xpack.security.transport.ssl parameters can be left as default since Elasticsearch generates its own certificates for this.
<br><br>

## <div id="startup">🚀 Startup
To configure Elastic so that it starts automatically on boot, run the following command:
```bash
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch
```
Elasticsearch can now be started using the following:
```bash
sudo systemctl start elasticsearch
```
> [!NOTE]
> This may take a minute to start up elastic
To check that elastic is running and startup hasn't failed, use the following command:
```bash
sudo systemctl status elasticsearch
```
In the event that Elasticsearch fails to start, use the following commands to look at the logs to determine the reason:
```bash
journalctl -u elasticsearch -n 50

# -n is the number of line you want to print out
```
To stop Elasticsearch at any point while running, use the following command:
```bash
sudo systemctl stop elasticsearch
```

<br><br><br>

# <img id="kibana" src="https://static-00.iconduck.com/assets.00/kibana-icon-1537x2048-476gnmfc.png" height="30px" width="30px">&nbsp; Kibana
For this lab, we'll be using kibana Version 8.11, the current version as of 2023. Refer to the official kibana Documentation for more information [Link](https://www.elastic.co/guide/en/kibana/current/install.html).
> [!NOTE]
> Ensure that you are using the same version of kibana as you are elasticsearch

## <div id="installation">💻 Installation 
To install Kibana, use the following command:
```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt-get update && sudo apt-get install kibana
```

After installation, Kibana should output setup configs to the terminal. Note them down as they contain login credentials that you'll need later on.
<br><br>

## <div id="configurations">⚙️ Configurations
Open the configuration file to kibana using the following command:
```bash
sudo vim /etc/kibana/kibana.yml
```

Remove the # from the beginning of lines to uncomment and set the following parameters:
   - server.port: {5601}
> [!IMPORTANT]
> Avoid using default ports to enhance security.

   - server.host: {"0.0.0.0"}
   - elasticsearch.hosts: {https:\\*Private_IP_Of_Elastic_Host*:9200"}
<br>

### Self Signed Certificates 
As by Defualt kibana doesn't encrypt its traffic over TLS and uses the standard HTTP protocol, using Self Signed Certificates will enable the encryption of traffic for our host.
To Store our Certs to be used by kibana make a folder called `Certs` in the `/etc/kibana/` directory.
<br><br>
1. Self Signed Certificates for Kibana:
```bash
./elasticsearch-certutil cert --out /etc/kibana/certs/kibana.zip --name kibana --ca-cert /etc/elasticsearch/certs/ca/ca.crt --ca-key /etc/elasticsearch/certs/ca/ca.key --ip 172.31.13.161 --pem

# cert - generate X.509 certificates and keys
# --out - Output path
# --name - Name of our certificate 
# --ca-cert - Path to our ca.crt
# --ca-key - Path to our ca.key
# --ip - Ip of host that will be using the certificate
# --DNS - DNS name of host that will be using the certificate (Do not need in this instance as we have a IP)
# --pem - Is the file format to make a our certificate and private key
```
2.Unzip the file:
```bash
unzip elastic.zip
```
3. Change Permission to the `certs` directory
```bash
sudo chown -R /etc/kibana/certs
```
<br>

To enabled SSL encryption in Kibana open back up the kibana configuration file in `/etc/kibana/kibana.yml` and make the following changes:
```yml
# =================== System: Kibana Server (Optional) ===================
server.ssl.enabled: true
server.ssl.certificateAuthorities: ["/etc/kibana/certs/ca.crt"]
server.ssl.certificate: /etc/kibana/certs/kibana/kibana.crt
server.ssl.key: /etc/kibana/certs/kibana/kibana.key
```
<br>

```yml
# =================== System: Elasticsearch (Optional) ===================

elasticsearch.ssl.certificateAuthorities: [ "/etc/kibana/certs/ca.crt" ]
elasticsearch.ssl.verificationMode: full
```
<br>

### Security 
Kibana also requires both the `xpack.encryptedSavedObjects.encryptionKey` and `elasticsearch.serviceAccountToken` to be configured in the in the `/etc/kibana/kibana.yml` file.
To setup a `xpack.encryptedSavedObjects.encryptionKey:` which will be required for encryption purposes in order to use connectors in kibana. The encryption key has to be a minium of 32 bytes long in order to ve valid and can be generated using the following command:
```bash
openssl rand -hex 16
```
<br>

To setup a ***serviceAccountToken*** which can be used to authenticate with elasticsearch instead of a generic username and password which can be harder to manage in this case.
ElasticSearch has a tool to assist in making this tokem which can be found in your `/usr/share/elasticsearch/bin` directory.
To create a service account token use the following command
```bash
 ./elasticsearch-service-tokens create elastic/kibana kibana_token
```
<br>

### Storing Keys
Instead of storing these keys in plain text in your config file, it's always best practice to keep them encrypted in case your machine is compromised. Kibana has a built-in key vault for storing these types of sensitive keys and credentials. To set this up, go to the /usr/share/kibana/bin directory and run the following command:
```bash
./kibana-keystore add elasticsearch.serviceAccountToken
```
You'll then be prompted to enter your value for the parameter. Repeat the same process for the xpack.encryptedSavedObjects.encryptionKey parameter:r.
```bash
./kibana-keystore add xpack.encryptedSavedObjects.encryptionKey
```
This will add a new file to your /etc/kibana directory, kibana.keystore, which will have these parameters kept encrypted and apply them automatically to Kibana on startup.
<br>

Finally, ensure that all the files in /etc/kibana can be accessed by the Kibana user. You can check this by running the following command:
```bash
ls -alh
```
This command will show you the permissions for every file in the directory. To add Kibana as the owner of these files, run the following command:
```bash
chown -R kibana:kibana ./
```
<br><br>

##  <div id="startup">🚀 Startup
To configure Kibana so that it starts automatically on boot, run the following command:
```bash
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable kibana
```
Kibana can now be started using the following:
```bash
sudo systemctl start kibana
```

<br>
To check that kibana is running and startup hasn't failed, use the following command:
```bash
sudo systemctl status kibana
```

In the event that kibana fails to start, use the following commands to look at the logs to determine the reason:
```bash
journalctl -u kibana -n 50

# -n is the number of line you want to print out
```

To stop kibana at any point while running, use the following command:
```bash
sudo systemctl stop kibana
```
<br><br>
## Conclusion
Congratulations! With both Elasticsearch and Kibana successfully deployed, you're now ready to explore the power of your Elastic SOC Lab. Access Elasticsearch at https://*public-ip*:6900 and Kibana at https://*public-ip*:5601.




