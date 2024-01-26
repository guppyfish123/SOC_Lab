# <img align="center" src="https://thehive-project.org/assets/ico/favicon.ico" height="45px" width="45px">&nbsp; Shuffle

## <div id="installation">💻 Installation
### Update Machine
Before diving into installating Shuffle, let's ensure our machine is up to date:
```bash
Sudo apt update -y && sudo apt upgrade -y
```
<br>

For this Lab we'll be running shuffle on docker. For that we'll need to have both ***Docker*** and ***Docker-Compose*** Installed on our host.
```bash
sudo apt install docker.io
sudo apt install docker-compose
```
To download Shuffle run the command:
```bash
git clone https://github.com/frikky/Shuffle
```
Navagate to the `Shuffle` Directory and run the following command to startup the docker file:
```bash
sudo docker-compose up -d # This may take a while when starting up the docker file for the first time, as the docker needs to install/setup the required packages
```
Fix prerequisites for the Opensearch database (Elasticsearch):
```bash
sudo chown 1000:1000 -R shuffle-database
```
Restart docker-compose to apply changes:
```bash
sudo docker-compose restart
```
Recommended actions for Opensearch
```bash
sudo sysctl -w vm.max_map_count=262144
```

## <div id="startup">🚀 Startup
Shuffle should now be running on our host and if you go to `https://<HOST_IP>:3443` you can now access Shuffle webpage
