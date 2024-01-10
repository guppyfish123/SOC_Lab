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
In order to connect elasticsearch to cortex, we'll need to make some changes to the configuration `/etc/cortex/application.conf` file as follows:
```conf
## ElasticSearch
search {
  # Name of the index
  index = cortex
  # ElasticSearch instance address.
  uri = "http://127.0.0.1:9200"
}
```

### Analyzers & Responders
These are the core of cortex and what will allow us to analyze & responde the data we input into cortex. 
First though before we can download these moduels, cortex require certain packages for these modules to work. These can be downloaded with the following command: 
```bash
sudo apt install -y --no-install-recommends python3-pip python3-dev ssdeep libfuzzy-dev libfuzzy2 libimage-exiftool-perl libmagic1 build-essential git libssl-dev
sudo pip3 install -U pip setuptools
```
We will create a directory in our `/opt` directory to store these modules in by running the command:
```bash
# Move to the /opt directory 
cd /opt

# Download the Analyzers & Responders modules from github 
git clone https://github.com/TheHive-Project/Cortex-Analyzers

# Give permission to cortex to access/read the Cortex-Analyzers directory 
chown -R cortex:cortex /opt/Cortex-Analyzers
```
Now we can setup each module which has been mapped out in our Cortex-Analyzers directory with each module having its own requirements.txt to install them with.
```bash
for I in $(find Cortex-Analyzers -name 'requirements.txt'); do sudo -H pip3 install -r $I || true; done
```
> [!NOTE]
> This download may take a while due to the large volume of modules that need to be installed.

<br>

Finally we need to configure Cortex to pull from the right file path to our Analyzers & Responders in our `/etc/cortex/application.conf` and make the following changes:
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
Now that we have both elasticsearch and cortex installed and configured as can startup both services.
<br>

### Elasticsearch 
```bash
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```
To check that elasticsearch has started up without failure, use command:
```bash
sudo systemctl status elasticsearch
```
In event that elasticsearch fails to start, you can check the logs using:
```bash
journalctl -u elasticsearch -n 100

# Or directory for elastic Log data
cd /var/log/elasticsearch
```
<br>

### Cortex
Now that we have elasticsearch running we can startup cortex by using command:
```bash
sudo systemctl enable cortex
sudo systemctl start cortex
```
To check that cortex has started up without failure, use command:
```bash
sudo systemctl status cortex
```
In event that cortex fails to start, you can check the logs using:
```bash
journalctl -u cortex -n 100

# Or directory for cortex Log data
cd /var/log/cortex
```
<br>

### Account setup 
Thse services may take some time to start up though one they have intilized completily you should be able to reach the cortex site `http://YOUR_SERVER_ADDRESS:9001/`.
There are just a couple more steps we need to do before we have everything ready to use:
1. Open your browser and connect to `http://YOUR_SERVER_ADDRESS:9001/`
2. When loading the page for the first time you'll be prompted to ***Update Database*** which will create a database in elasticsearch that were using to store all of our data for cortex.
3. You will be ask to then create your first cortex user which will be your global administrator account.
4. Now that we are logged into our global administrator account, we can now create our first organization by clicking the ***Add organization*** button.
5. Within your organization we have to make our first user by clicking the ***Add User*** button. Ensure to set Roles to ***Read, Analyze, orgadmin***.
6. Once your first user has been created for your oganization you'll need to reset the password before we can login to it. Click the ***New Password*** button to do this.
7. Logout of your global admin account on the top right and sign into your new account that we just made

### Analyzers & Responders
Now that we our in our organization that we have just made we are now able to access our **Analyzers & Responders** which you can enable by going to **Organization**>**Analyzers**. Listed are all installed Analyzers & Responders that we can enable as we need.

<br><br>

## Security 
To ensure that our data remain safe & secure we are going to configure traffic to be sent over HTTPS. Cortex shouldn't have a built in function that does this so we are going to use a reverse proxy setup.
We'll be using nginx to setup our reverse proxy with our cortex service to run over HTTPS.

### Nginx Install
To install nginx, use command:
```bash
sudo apt-get install nginx
```
<br>

### Self-Signed certificates 
In order for us to generate a certificate and key with a Cerficiatae Authority, we'll be using `openssl`.
<br>

**OpenSSL Install**<br>
```bash
sudo apt-get install openssl
```
<br>

**Generate Certificates**
1. First we'll make a directory to store our certificates in.
  ```bash
  mkdir /etc/nginx/sites-available/certs

  # Change permissiont to give nginx access to the folder and files in it
  chown -R www-data:www-data ./certs
  ```
2. First we need a to generate a private key which you can do by running the command:
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

## Nginx configuration
1. Change to `/etc/nginx/sites-available/` directory and create a file `cortex.conf` with the following code inside:
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
          proxy_pass              http://127.0.0.1:9001/;
          proxy_http_version      1.1;
          proxy_set_header        Upgrade $http_upgrade;
          proxy_set_header        Connection 'upgrade';
          proxy_set_header        Host $host;
          proxy_cache_bypass      $http_upgrade;
      }
  }
  ```
2. Ensure that you give permissions to cortex access/run the file.
  ```bash
  chown -R www-data:www-data ./cortex.conf
  ```
3. Navigate to the `/etc/nginx/sites-enabled/` directory so that we can Create a symbolic link to our cortex.conf file so that cortex knows what to run:
  ```bash
  sudo ln -s /etc/nginx/sites-available/cortex.conf cortex
  ```
4. Remove the following files to remove the defualt nginx page:
  ```bash
  rm /etc/nginx/sites-available/defualt
  rm /etc/nginx/sites-enabled/defualt
  ```
5. Ensure that our next `./cortex.conf` can run with the following command to test the configurations with nginx:
  ```bash
  sudo nginx -t
  ```
6. Reload nginx to apply the changes we have made.
  ```bash
  sudo systemctl reload nginx
  ```
7. Cortex should now be accessable on `https://YOUR_SERVER_ADDRESS`
