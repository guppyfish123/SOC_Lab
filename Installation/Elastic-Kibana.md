<h1 align=center><img align="center" align=center src="https://www.elastic.co/apple-icon-57x57.png" height="45px" width="45px">&nbsp;&nbsp;ElasticSearch & Kibana Installation/Setup</h1>

## AWS
To get Elastic up and running, our first step is to set up a server, and AWS is the go-to platform for VMs. While you're free to choose any platform, AWS is highly recommended due to its popularity and reliability. If you're looking for cost-effective options, check out these Cheap VM Hostings. Keep in mind that this might be the only part of the lab where some expenses could be involved.
[Cheap VM Hostings](https://webhostingadvices.com/19-cheap-vm-hosting/).
If you lack experience with major cloud providers (AWS, Azure, Google), this is an excellent opportunity to dive into one of them.
<br><br>
### AWS Setup Instructions
1. Navigate to the EC2 Dashboard on AWS and select `Launch Instance`.
2. Give your VM a meaningful name under `Name and tags`. For example, I'll name mine Elastic_Ubuntu.

> [!TIP]
> Stick to a Naming Convention for clarity and future reference.

3. Under `Application and OS Images`, choose Ubuntu as the OS and leave other configurations as default.
4. Move to `Instance type` to specify your specs. While Elastic can run on T2.Medium, T2.Large is recommended for smoother performance.
5. Set up a key for login under `Key pair`. Click Create new key pair.
    - Provide a `Key pair name`, keeping it consistent with your server name.
    - `Key pair type` can be left as default (RSA).
    - Choose the `Private key file format` depending on your preference (e.g., .ppk for Putty).
    - After configuring, click `Create Key Pair`, and move the downloaded Private key to your project folder.<br><br>

> [!CAUTION]
> Double-check your private key installation, as it cannot be regenerated once the instance is created.

6. In `Network settings`, leave the default settings for Create Security Group and Allow SSH traffic from selected. Later, we'll modify these to reach the Elastic web page, avoiding unnecessary open ports.
7. In `Configure storage`, set the storage to 15 GiBs, with the option to add more volumes later if needed.
8. `Advanced details` can be left as default.
9. Review your summary and ensure under `Number of instances`, only 1 instance is being created.
10. Deploy your instance by clicking `Launch Instance`.<br><br>

Your VM is created, and while it might take some time to complete, head back to your instance dashboard to find the details, including your Private & public IP needed for later.

To connect to your machine via AWS, use the Connect button on the top right. If it's greyed out, your instance might not be ready or started. In the Connect to instance tab, focus on EC2 Instance Connect for AWS connection or SSH Client for connecting locally through SSH.
<br><br>
### Putty
For Windows users, connecting to your VM using Putty is a straightforward process. Putty is a commonly used software for SSH connections. Follow this quick walkthrough:

Download: [Putty](https://www.putty.org/)

1. Open up the Putty Application on your machine
2. Putty will load into the sesion detail tab, here you will enter in your Public IPv4 DNS to your instance into the `Host Name (or IP Address)` field. If your following along in the `Connect to instance` SSH Client Tab then your Public IPv4 DNS to your instance is provided in listing 4. `Port` should be set to 22 and `Connection Type` as SSH though this should be set automatically as default.
3. Next we need to add the user we are wanting to connect to, which can be done in the `Data` tab on the left under `Connection`. Set the `Auto-Login username` to Ubuntu, which is the default user that is create when you launch a Ubuntu AWS Instance.
4. Lastly we need to apply our Private key that we downloaded when we made our insatnce. Expand the `SSH` and `Auth` tabs on the left hand side till you see the `Credentials` tab. In the credentials tab we want to select our Private key that we have on file by hitting `Browse` under the `Private key file for authentication` field.
5. Finally we can connect to our VM by hittin the `Open` button on the bottom right on the window

You'll most likily be greeted with a pop-up warning which you can just accept. Though beside from that you have now connected to your AWS VM. 
<br><br>

## ElasticSearch Installation
Setting up ElasticSearch can be a bit intricate, and the process may vary depending on the version you're working with. For this lab, we'll be using Elastic Version 8.11, the current version as of 2023. Refer to the official ElasticSearch Guide for more details [Link](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html).
<br>
### Update Machine
Before diving into ElasticSearch installation, let's ensure our machine is up to date:
```
Sudo apt update && sudo apt upgrade -y
```

### ElasticSearch Installation Steps
Now, let's proceed with installing ElasticSearch:
```
# Download and install the public signing key
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

# Install the Elasticsearch Debian package
sudo apt-get update && sudo apt-get install elasticsearch
```
With these commands, ElasticSearch is now up and running.
<br><br>
## Elastic Setup/Configs
ElasticSearch's configurations can be complex, depending on your environment and requirements. We'll adjust settings to enhance the security of our Elastic instance. Configuration settings are stored in the elasticsearch.yml file. Open the file using Vim:
```
sudo vim /etc/elasticsearch/elasticsearch.yml
```
> [!NOTE]
> We'll be using Vim as our text editor in this lab. If you prefer another text editor like Nano, feel free to use it. For a quick Vim tutorial, check [link](../vim.md)

<br><br>
Remove the # from the beginning of lines to uncomment and set the following parameters:
  - luster.name: {Name of your elastic cluster}
  - network.host: {Set to either a allocated IP, DNS name, or, for this instance, we'll set it to our VM's assigned IP by setting it to 0.0.0.0}
  - http.port: {Change from default 9200 for security reasons}
> [!IMPORTANT]
> Avoid using default ports to enhance security.
  - http.host: 0.0.0.0
<br><br>

### Self Signed Certificates 
To secure our Elastic Host and encrypt traffic over HTTPS, we'll set up Self-Signed Certificates using elasticsearch-certutil found in /usr/share/elasticsearch/bin.

1. Make our CA certificate and key for our clusters:
```
./elasticsearch-certutil ca --pem --out /etc/elasticsearch/certs/ca.zip

# ca - Generates a new local certificate authority
# --pem - Is the file format to make a our certificate and private key
# --out - Is our output path
```
2. Unzip the file:
```
sudo apt install zip
unzip ca.zip
```
3. Make our ElasticSearch host certificate:
```
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
```
unzip elastic.zip
```
5. Change Permission to certs directory
```
sudo chown -R /etc/elastisearch/certs
```
<br>

Now, add the certificates to your elasticsearch.yml file:
```
xpack.security.enabled: true
xpack.security.enrollment.enabled: true
xpack.security.http.ssl.enabled: true
xpack.security.transport.ssl.enabled: true
```
Now to add our certificates
```
xpack.security.http.ssl:
  enabled: true
  certificate: certs/elastic/elastic.crt
  key: certs/elastic/elastic.key
  certificate_authorities: certs/ca/ca.crt
```
Remove any truststore.path or keystore.path variables from the xpack.security.http.ssl parameter, as they are no longer needed. The xpack.security.transport.ssl parameters can be left as default since Elasticsearch generates its own certificates for this.

<br>

## Start up ElasticSearch 

To configure Elastic so that it starts automatically on boot, run the following command:
```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch
```
Elasticsearch can now be started using the following:
```
sudo systemctl start elasticsearch
```
> [!NOTE]
> This may take a minute to start up elastic
<br>
To check that elastic is running and startup hasn't failed, use the following command:
```
sudo systemctl status elasticsearch
```

In the event that Elasticsearch fails to start, use the following commands to look at the logs to determine the reason:
```
journalctl -u elasticsearch -n 50

# -n is the number of line you want to print out
```

To stop Elasticsearch at any point while running, use the following command:
```
sudo systemctl stop elasticsearch
```

<br><br><br>

## Kibana Installation 
Now that we have ElasticSearch installed and configured, we move on to Kibana. We'll be installing version 8.11 of Kibana. Make sure that whatever version you are using matches the version of ElasticSearch that you have running. More information can be found in the official Kibana documentation [Link](https://www.elastic.co/guide/en/kibana/current/setup.html).

To install Kibana, use the following command:
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt-get update && sudo apt-get install kibana
```

After installation, Kibana should output setup configs to the terminal. Note them down as they contain login credentials that you'll need later on.

Now, let's configure Kibana:
```
sudo vim /etc/kibana/kibana.yml
```
<br>

Remove the # from the beginning of lines to uncomment and set the following parameters:
   - server.port: {5601}
> [!IMPORTANT]
> Avoid using default ports to enhance security.

   - server.host: {"0.0.0.0"}
   - elasticsearch.hosts: {https:\\*Private_IP_Of_Elastic_Host*:9200"}

<br><br>
### Self Signed Certificates 
Now, we need to create a self-signed certificate for our Kibana host to secure its traffic over HTTPS instead of Kibana's default HTTP, which isn't secure.

To get started, create a directory to store your certs while in the /etc/kibana/ directory. Let's call our directory certs.

Head back over to your /usr/share/elasticsearch/bin directory so that you can create another certificate.
1. Make our Kibana host certificate:
```
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
```
unzip elastic.zip
```
3. Change Permission to certs directory
```
sudo chown -R /etc/kibana/certs
```

<br><br>

## Start up Kibana
Now to add our certificate to our kibana configs and enable SSL encryption. 
Heading back over to our '/etc/kibana/kibana.yml' file:
```
# =================== System: Kibana Server (Optional) ===================
server.ssl.enabled: true
server.ssl.certificateAuthorities: ["/etc/kibana/certs/ca.crt"]
server.ssl.certificate: /etc/kibana/certs/kibana/kibana.crt
server.ssl.key: /etc/kibana/certs/kibana/kibana.key
```
<br>

```
# =================== System: Elasticsearch (Optional) ===================

elasticsearch.ssl.certificateAuthorities: [ "/etc/kibana/certs/ca.crt" ]
elasticsearch.ssl.verificationMode: full
```
<br><br>

We'll also need to setup both a `xpack.encryptedSavedObjects.encryptionKey` and a `elasticsearch.serviceAccountToken` to put into the yml file.
<br>

To setup a `xpack.encryptedSavedObjects.encryptionKey:` which will need to be setup to use any of the connectors in elastic. This encrypts the stored variabled for connectos such as creds and API keys and does this by the encryption key provided. The encryption key has to be a minium of 32 bytes long. You can make a 32 byte key by using the following command:
```
openssl rand -hex 16
```
<br><br>

Next up is the `elasticsearch.serviceAccountToken` which we are going to use instead of a username & password to connect to elasticsearch as this in my more secure way then leaving creds in plain text. 
ElasticSearch has a tool to assist in making this tokem which can be found in your `/usr/share/elasticsearch/bin` directory.
To create a service account token use the following command
```
 ./elasticsearch-service-tokens create elastic/kibana kibana_token
```

<br><br>

Now instead of storing these creds
