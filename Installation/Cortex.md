# <img align="center" src="https://thehive-project.github.io/Cortex-Analyzers/images/cortex-logo.png" height="45px" width="45px">&nbsp; Cortex
Cortex stands as a robust analysis platform for quering various observables on a large scale, including IP addresses, emails, URLs, domains, files, and hashes, all within a unified single environment. Serving as a central hub for threat intelligence, digital forensics, and incident response. Through its build in REST API, automation capabilities are also possible.
<br><br>
The Cortex platform empowers SOCs and security administrators to conduct in-depth investigations using a comprehensive library of built-in analyzer and response actions. Moreover, Cortex provides the flexibility of customization to tailor its functionalities according to specific needs.
<br><br>
For detailed information about Cortex and its official documentation, refer to there [official documentation.](https://docs.thehive-project.org/cortex/)

<br>

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
To kick off the Lab we first require a virtual machine (VM) to host our Cortex services. In this lab, I'll be utilizing AWS's EC2 Instances Service to run my VMs. However, feel free to choose any cloud or on-premises service that suits your preferences and budget. If you're unfamiliar with setting up a VM in AWS, you can follow a step-by-step walkthrough provided [here](./aws).
<br>

## <div id="installation">💻 Installation
### Update Machine
Before diving into the installation of Cortex, let's ensure our host is up to date:
```bash
Sudo apt update && sudo apt upgrade -y
```
<br>
 
Cortex requires several Library/Packages that need to be installed before hand. Use the following command to install these required packages:
```bash
apt install wget gnupg apt-transport-https git ca-certificates ca-certificates-java curl  software-properties-common python3-pip lsb_release
```
<br>

### Java
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

### Cortex
```bash
wget -O- "https://raw.githubusercontent.com/TheHive-Project/Cortex/master/PGP-PUBLIC-KEY"  | sudo apt-key add -
wget -qO- https://raw.githubusercontent.com/TheHive-Project/Cortex/master/PGP-PUBLIC-KEY |  sudo gpg --dearmor -o /usr/share/keyrings/thehive-project.gpg
echo 'deb https://deb.thehive-project.org release main' | sudo tee -a /etc/apt/sources.list.d/thehive-project.list
apt install cortex
```
<br><br>

## <div id="configurations">⚙️ Configurations
### Elasticsearch
To enable Cortex to connect to Elasticsearch services, you must configure Elasticsearch to run on your localhost. Modify the configuration file `/etc/elasticsearch/elasticsearch.yml` as follows:
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
<br>

### Cortex
To establish a connection between Elasticsearch and Cortex, adjustments to the configuration file `/etc/cortex/application.conf` as follows:
```conf
## ElasticSearch
search {
  # Name of the index
  index = cortex
  # ElasticSearch instance address.
  uri = "http://127.0.0.1:9200"
}
```
<br>

It is always best practise to not leave services running on their default ports, as this makes it easier for threat actors to know what services you are running on your host. As such, change
the port that cortex is running on by changing the following line in the configuration file `/etc/cortex/application.conf` as follows:   
```conf
http.port = 8090
```

<br>

### Analyzers & Responders
Analyzers and Responders are the core function of Cortex, enabling us to analyze and respond is a wide range of forms with multiple different sources of data.

1. Cortex required certain library/packages installed before hand to install the Analyzers and Responders modules into cortex:
 ```bash
 sudo apt install -y --no-install-recommends python3-pip python3-dev ssdeep libfuzzy-dev libfuzzy2 libimage-exiftool-perl libmagic1 build-essential git libssl-dev
 sudo pip3 install -U pip setuptools
 ```
2. These modules will be stored in our `/opt` directory, which will then be used by cortex:
 ```bash
 # Move to the /opt directory 
 cd /opt
 
 # Download the Analyzers & Responders modules from github 
 git clone https://github.com/TheHive-Project/Cortex-Analyzers
```
3. With the `Cortex-Analyzers` downloaded, containing within it the Analyzers and Responders modules. To install these each Analyzers and Responders modules has a requirements.txt that outlines to installation requirements. This command will loop through each of those requirements.txt files and download the required python modules for the Analyzers and Responders:
 ```bash
 for I in $(find Cortex-Analyzers -name 'requirements.txt'); do sudo -H pip3 install -r $I || true; done
 ```
> [!NOTE]
> The download may take a while due to the large volume of modules that need to be installed.
<br>

4. Update the permission to the `Cortex-Analyzers` directory so that cortex can access the file within it:
 ```bash
 chown -R cortex:cortex /opt/Cortex-Analyzers
 ```
5.Finally we need to update the cortex configuration file`/etc/cortex/application.conf`, to pull from the correct file paths where our Analyzers and Responders modules are located:
 ```bash
 [..]
 analyzer {
   urls = [
     "/opt/Cortex-Analyzers/analyzer",
   ]
 
   fork-join-executor {
     parallelism-min = 2
     parallelism-factor = 2.0
     parallelism-max = 4
   }
 }
 
 responder {
   urls = [
     "/opt/Cortex-Analyzers/responders"
   ]
 
   fork-join-executor {
     parallelism-min = 2
     parallelism-factor = 2.0
     parallelism-max = 4
   }
 }
 [..]
 ```
<br><br>

## <div id="startup">🚀 Startup
Now that both Elasticsearch and Cortex are installed and configured, there respected services can be started.
<br>

### Elasticsearch 
```bash
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```
To verify that Elasticsearch has started without any errors, use the command:
```bash
sudo systemctl status elasticsearch
```
In the event that Elasticsearch fails to start, you can check the logs using:
```bash
journalctl -u elasticsearch -n 100

# Or directory for elastic Log data
cd /var/log/elasticsearch
```
<br>

### Cortex
Now that Elasticsearch is running, we can startup Cortex with the following commands:
```bash
sudo systemctl enable cortex
sudo systemctl start cortex
```
To check if Cortex has started successfully, use the command:
```bash
sudo systemctl status cortex
```
If Cortex fails to start, you can examine the logs using:
```bash
journalctl -u cortex -n 100

# Or directory for cortex Log data
cd /var/log/cortex
```
<br>

### Account setup 
These services may take some time to start up, but once they have initialized completely, Cortex Webpage will be reachable on `http://YOUR_SERVER_ADDRESS:8090/`.
**Account/Organisation Setup**
1. Open a browser and connect to your Cortex page `http://YOUR_SERVER_ADDRESS:8090/`
2. When loading the page for the first time, you'll be prompted to Update the Database, which will create a database in Elasticsearch that we're using to store all of our data for Cortex.
3. Following, will be creating your first Cortex user, which will be your global administrator account.
4. log into the global administrator account which will have limited functionality. Click on ***Add Organization*** to create a new organisation.
5. To create a new user under an organisation, while in your organization click on ***Add User*** and ensure that the role is set to ***Read, Analyze, orgadmin***.
6. To create a password for a new user, click on ***New Password*** on the user to reset the passsword to the account so that it can be logged into.
7. Logout of your global admin account on the top right and sign into your new user account. 
<br>

### Analyzers & Responders
Full functionality to Cortex should now be open and access Analyzers & Responders will be open. To enable our Analyzers to be used, go to Organization > Analyzers page. Here will be a listed view all built in Analyzers that can be enabled at choice. The same can be done for Responders under Organization > Responders.
<br><br>

## Security 
To ensure data remains encrypted while in trasnit and cannot be read by threat actors, Cortex can be configured to send traffic over TLS. Cortex by default will send traffic over HTTP instead of HTTPS and doesn't have a built in function to secure traffic over TLS.
Setting up a Nginx service to redirect traffic over HTTPS, will secure our traffic and ensure that communication between server and end user remains encrypted.  
<br>

### Nginx Install
Install the nginx service using:
```bash
sudo apt-get install nginx
```
<br>

### Self-Signed Certificates 
To generate Self Signed Certificates will require the use of `openssl` library.
<br>

**OpenSSL Install**<br>
```bash
sudo apt-get install openssl
```
<br>

**Generate Certificates**
1. First make a directory to store the certificates we create in.
  ```bash
  mkdir /etc/nginx/sites-available/certs

  # Change permissions to give nginx access to the folder and files in it
  chown -R www-data:www-data ./certs
  ```
2. First item that we will require is a Private Key which can be generated using:
  ```bash
  openssl genpkey -algorithm RSA -out certificate.key
  ```
3. Generate a certificate signing request (CSR).
  ```bash
  openssl req -new -key certificate.key -out certificate.csr
  ```
4. Generate a Self-Signed Certificate.
  ```bash
  openssl x509 -req -days 365 -in certificate.csr -signkey certificate.key -out certificate.crt
  ```
<br>

## Nginx Configuration
1. Change to the `/etc/nginx/sites-available/` directory and create a file `cortex.conf` with the following code:
  ```conf
  server {
      listen 443 ssl http2;
      server_name cortex;
  
      #Change with path to your certificates
      ssl_certificate     /etc/nginx/sites-available/certs/certificate.crt; 
      ssl_certificate_key /etc/nginx/sites-available/certs/certificate.key;
  
      ssl_protocols       TLSv1.2 TLSv1.3;
      ssl_ciphers         'TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384';
  
      ssl_prefer_server_ciphers off;
      ssl_session_timeout 1d;
      ssl_session_cache shared:SSL:50m;
  
      # Enable OCSP stapling
      ssl_stapling on;
      ssl_stapling_verify on;
  
      # Set the resolver to a DNS server that supports OCSP stapling (e.g., Google's public DNS)
      resolver 8.8.8.8 8.8.4.4 valid=300s;
      resolver_timeout 5s;
  
      proxy_connect_timeout   600;
      proxy_send_timeout      600;
      proxy_read_timeout      600;
      send_timeout            600;
      client_max_body_size    2G;
      proxy_buffering off;
      client_header_buffer_size 8k;
  
      location / {
          add_header              Strict-Transport-Security "max-age=31536000; includeSubDomains";
          proxy_pass              http://127.0.0.1:8090/;
          proxy_http_version      1.1;
          proxy_set_header        Upgrade $http_upgrade;
          proxy_set_header        Connection 'upgrade';
          proxy_set_header        Host $host;
          proxy_cache_bypass      $http_upgrade;
      }
  }
  ```
2. Ensure that you give permissions for cortex to access the file.
  ```bash
  chown -R www-data:www-data ./cortex.conf
  ```
3. Navigate to the `/etc/nginx/sites-enabled/` directory so that we can Create a symbolic link to our cortex.conf file so that cortex knows what site run:
  ```bash
  sudo ln -s /etc/nginx/sites-available/cortex.conf cortex
  ```
4. Remove the following files to remove the default nginx page:
  ```bash
  rm /etc/nginx/sites-available/default
  rm /etc/nginx/sites-enabled/default
  ```
5. Ensure that our next `./cortex.conf` can run with the following command to test the configurations with nginx:
  ```bash
  sudo nginx -t
  ```
6. Reload nginx to apply the changes we have made.
  ```bash
  sudo systemctl reload nginx
  ```
7. Cortex should now be accessible on `https://YOUR_SERVER_ADDRESS`
<br><br>

### Close port 8090 
As we now have our reverse proxy setup to redirect traffic through port 443 for secure encryption, port 8090 is still open. To ensure we don't leave any unnecessary ports open, we are going to add a rule to our internal Iptable on our ubuntu host to redirect all traffic from *:8090 to 127.0.0.1:8090. This will ensure that our reverse proxy will still function, while at the same time no users can access cortex through port 8090.
Use the following command to make the necessary changes to your IPTable:
```bash
# Allow loopback interface
sudo iptables -A INPUT -i lo -j ACCEPT

# Allow incoming connections to port 8090 from localhost
sudo iptables -A INPUT -p tcp --dport 8090 -s 127.0.0.1 -j ACCEPT

# Drop all other incoming connections to port 8090
sudo iptables -A INPUT -p tcp --dport 8090 -j DROP
```
To save these changes that we have made so that on reboot they are not lost, use command:
```bash
# Save the rules
sudo sh -c "iptables-save > /etc/iptables.rules"

# Restore the rules during startup
sudo sh -c "iptables-restore < /etc/iptables.rules"
```
Using the following command will show all ports that our open on our host, ensuring that we only have port 443 open for cortex:
```bash
ss -tuln
```
<br>
> [!NOTE]
> While this step might seem unnecessary since we already have security groups assigned to each of our AWS EC2 instances, managing open ports individually adds an extra layer of security. Even though we control open ports through security groups, it's always good practice to ensure that no ports are left unchecked.
<br><br>

### Security groups
In AWS, managing security groups is crucial to controlling traffic coming into and from our host. Adhering to the principle of least privilege ensures that only the necessary ports are open that the host required in order to operation with. For detailed instructions on configuring security groups in AWS, refer to [GUIDE](./aws).
