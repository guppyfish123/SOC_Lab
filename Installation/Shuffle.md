# <img align="center" src="https://thehive-project.org/assets/ico/favicon.ico" height="45px" width="45px">&nbsp; Shuffle

## :books: Table Of Content
 - [Installation](#installation)
    - [Shuffle Docker](#shuffle-docker)
 - [Configuration](#configuration)
    - [Folder Security](#folder-security)
    - [Secure TLS](#secure-tls)
 - [Startup](#startup)

<br>

## <img align="center" src="https://files.softicons.com/download/social-media-icons/free-social-media-icons-by-uiconstock/png/512x512/AWS-Icon.png" height="33px" width="33px">&nbsp;  AWS
To kick off, the first step is to launch a virtual machine (VM) to host our Shuffle services. In this guide, I'll be using an AWS EC2 Ubuntu instance for this purpose. However, feel free to choose any cloud or on-premises service that suits your preferences. If you're unfamiliar with setting up a VM in AWS, you can follow my step-by-step walkthrough provided [here](./aws).
<br><br>

## <div id="installation">💻 Installation
### Update Machine
Before diving into installating Shuffle, let's ensure our machine is up to date:
```bash
Sudo apt update -y && sudo apt upgrade -y
```
<br>

### Shuffle Docker
1. For this Lab we'll be running shuffle on docker. For that we'll need to have both ***Docker*** and ***Docker-Compose*** Installed on our host.
  ```bash
  sudo apt install docker.io
  sudo apt install docker-compose
  ```
2. To download Shuffle run the command:
  ```bash
  git clone https://github.com/frikky/Shuffle
  ```
3. Navagate to the `Shuffle` Directory and run the following command to startup the docker file:
  ```bash
  sudo docker-compose up -d # This may take a while when starting up the docker file for the first time, as the docker needs to install/setup the required packages
  ```
4. Fix prerequisites for the Opensearch database (Elasticsearch):
  ```bash
  sudo chown 1000:1000 -R shuffle-database
  ```
5. Restart docker-compose to apply changes:
  ```bash
  sudo docker-compose restart
  ```
6. Recommended actions for Opensearch
  ```bash
  sudo sysctl -w vm.max_map_count=262144
  ```
<br>

## <div id="configuration">🚀 Configuration
### Folder Security
To secure our `Shuffle` folder with our docker configs, we'll move the folder over to the etc directory:
```bash
cp /path/to/destination /path/to/file
```
To ensure our Shuffle folder isn't accessable by all users, we'll lock to file down so that docker and root can access it
```bash
sudo chown root:docker /Shuffle
sudo chmod 770 /Shuffle
```
<br>

### Secure TLS
As we want to use a secure connection to our shuffle webpage that uses TLS, we can to close off port 80. 
Navagate to your `docker-compose.yml` file and make the following changes:
```yml
services:
  frontend:
    image: ghcr.io/shuffle/shuffle-frontend:latest
    container_name: shuffle-frontend
    hostname: shuffle-frontend
    ports:
#     - "${FRONTEND_PORT}:80"  Comment out this line to disable shuffle running on port 80
      - "${FRONTEND_PORT_HTTPS}:443"
    networks:
      - shuffle
    [...]
```
<br><br>

## <div id="startup">🚀 Startup
Now that we have our shuffle instance up and running we want to make it start on boot so that everytime we need to restart or startup or machine we don't have manually startup the docker file. 
1. Create a systemd service file using the following comannd:
  ```bash
  sudo vim /etc/systemd/system/my-docker-app.service
  ```
2. Add the following content to the file and edit to fit the file path to your `docker-compose.yml` shuffle file:
  ```ini
  [Unit]
  Description=Docker Compose Shuffle Application
  Requires=docker.service
  After=docker.service
  
  [Service]
  Type=oneshot
  RemainAfterExit=yes
  WorkingDirectory=/home/ubuntu/Shuffle
  ExecStart=/usr/bin/docker-compose up -d
  ExecStop=/usr/bin/docker-compose down
  
  [Install]
  WantedBy=multi-user.target
```
3. Reload systemd to apply changes:
  ```bash
  sudo systemctl daemon-reload
  ```
4. Enaled the service to start on boot:
  ```bash
  sudo systemctl enable my-docker-app.service
  ```
5. The following command can be used to start the shuffle service and check status:
  ```bash
  #Start Service
  sudo systemctl start my-docker-app.service

  # Check status
  sudo systemctl status my-docker-app.service
  ```
<br>

Shuffle is now be running on our host and accessable through `https://<HOST_IP>:3443` webpage on your browser.
When accessing for the first time you should be prompted to create your admin account.
