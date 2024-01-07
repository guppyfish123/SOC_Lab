# <img align="center" src="https://seeklogo.com/images/E/elastic-beats-logo-02512BDFD2-seeklogo.com.png" height="40px" width="30px">&nbsp; Filebeat
In this lab, we'll leverage Filebeat version 8.11. Refer to the official [Filebeat documentation](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation-configuration.html) for comprehensive installation and configuration details.

> [!NOTE]
> Ensure consistency by using the same version of Filebeat as your Elasticsearch and Kibana on the other host.
<br>

## Introduction
In this part of the lab, we'll set up a host running a default Apache website with Filebeat installed. This host will seamlessly connect to our Kibana and Elasticsearch instances, becoming the conduit for capturing real-world traffic from the public internet.

This process allows us to feed valuable data into our Elastic SIEM, providing a practical and hands-on experience in the realm of security operations.

### VM Setup
First thing, we need a Virtual Machine (VM) to host our filebeat and our web server. I'll be utilizing a T2.small instance in aws, which will serve as a primary host throughout this lab.
If you're unfamiliar with setting up a VM in AWS, I've provided a handy mini-tutorial [here](./aws). It guides you through the process, ensuring you're ready to launch your AWS instance seamlessly.
<br><br>

## <div id="installation">💻 Installation
To install filebeat, use the following command:
```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.11.3-amd64.deb
sudo dpkg -i filebeat-8.11.3-amd64.deb
```
Filbeat is now installed, you can check that you have the right version installed by using the following command:
```bash
filebeat version
```
<br><br>

## <div id="configurations">⚙️ Configurations
Now to need to configure the filebeat service and connect it to our elasticsearch & kibana on our other host so that we can send log data to it.
Do this we'll need to head over to our yml file
```bash
sudo vim /etc/filebeat/filebeat.yml
```
<br>

### Filebeat input 
This section defines the inputs for Filebeat, specifying which logs it will capture and forward to our Elasticsearch instance. While many inputs are covered by modules, this manual method is employed for logs located in the `/var/log` directory.
   
```yml
# ============================== Filebeat inputs ===============================

filebeat.inputs:
- type: filestream
  id: my-filestream-id
  enabled: true
  paths:
    - /var/log/*.log
```
<br>

### Filebeat modules
Modules are crucial for determining the logs to be collected. Filebeat comes with several pre-installed modules for popular services, making it efficient to choose and enable specific services for log capture and forwarding to Elasticsearch.
```yml
# ============================== Filebeat modules ==============================

filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: true
  reload.period: 10s

setup.template.settings:
  index.number_of_shards: 1
```
<br>

### Dashboard
```yml
setup.dashboards.enabled: true
```
<br>

### SSL Certificates Setup for Filebeat
As we've configured both Elasticsearch and Kibana to operate over HTTPS with SSL enabled using our self-signed certificates, it's essential to have copies of the Certificate Authority (CA), Certificates, and Keys for both services on our Filebeat host.

Here's the directory structure we've set up on our Filebeat machine:

```
├── certs
| ├── ca.crt 
| ├── elastic                   
| │   ├── elastic.crt          
| │   ├── elastic.key         
| ├── kibana                   
| │   ├── kibana.crt            
| │   ├── kiabna.key                     
```
<br>

### Kibana Setup 
Connecting Filebeat to our Kibana host involves specifying its configuration port and private IP. Since we've configured Kibana to work over HTTPS, we add the self-signed certificate.
```yml
setup.kibana:
  host: "https://172.31.8.55:5601"
  ssl:
    enabled: true
    certificate_authorities: ["/etc/filebeat/certs/ca.crt"]
    certificate: "/etc/filebeat/certs/elastic/elastic.crt"
    key: "/etc/filebeat/certs/elastic/elastic.key"
```
<br>

### Elasticsearch Output
Connecting Filebeat to our Elasticsearch host involves specifying its configuration port and private IP. Since we've configured Elasticsearch to work over HTTPS, we add the self-signed certificate. Elasticsearch also require credentials to login to the service, which can be done either through a username and password or through a API key. 
```yml
# ---------------------------- Elasticsearch Output ----------------------------

output.elasticsearch:
  hosts: ["172.31.8.55:9200"]
  protocol: "https"
  ssl:
    certificate_authorities: ["/etc/filebeat/certs/ca.crt"]
    certificate: "/etc/filebeat/certs/kibana/kibana.crt"
    key: "/etc/filebeat/certs/kibana/kibana.key"

  api_key: "{api key}"

```

<br><br>

## <div id="startup">🚀 Startup


