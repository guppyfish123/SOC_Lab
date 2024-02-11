# <img align="center" src="https://seeklogo.com/images/E/elastic-beats-logo-02512BDFD2-seeklogo.com.png" height="40px" width="30px">&nbsp; Filebeat
Filebeat, a component of the Elastic Stack, operates as a lightweight log shipper designed to streamline the process of collecting, forwarding, and centralizing log data. Installed on individual hosts, Filebeat excels in harvesting specific log files or data from various services, offering a flexible and configurable solution. Once configured, it efficiently forwards the collected logs to your Elasticsearch or Kibana instance, facilitating centralized storage and analysis. This seamless integration with Kibana enables users to harness the power of Elastic Stack's visualization and querying capabilities, making Filebeat a crucial tool for organizations seeking to aggregate and analyze log data from diverse sources across their infrastructure.
<br><br>
In this lab, we'll leverage Filebeat version 8.11. Refer to the official [Filebeat documentation](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation-configuration.html) for comprehensive installation and configuration details.

> [!NOTE]
> Ensure that the filebeat version that is used aligns with that of the version used on you kibana instance
<br>

## :books: Table Of Content
 - [Installation](#installation)
   - [Install Filebeat](#install-filebeat)
 - [Configurations](#configurations)
   - [Input](#input)
   - [Modules](# modules)
   - [Dashboard](#dashboard)
   - [Kibana Setup](#kibana-setup)
   - [Elasticsearch Output](#elasticsearch-output)
   - [API Key](#api-key)
   - [Enabled Modules](#enabled-modules)
   - [Apache Web Server](#apache-web-server)
 - [Startup](#startup)

<br>

## Introduction
In this part of the lab, we'll set up a host running a default Apache website with Filebeat installed. This host will seamlessly connect to our Kibana and Elasticsearch instances, becoming the conduit for capturing real-world traffic from the public internet.

This process allows us to feed valuable data into our Elastic SIEM, providing a practical and hands-on experience in the realm of security operations.
<br>

## <img align="center" src="https://files.softicons.com/download/social-media-icons/free-social-media-icons-by-uiconstock/png/512x512/AWS-Icon.png" height="33px" width="33px">&nbsp;  AWS
To kick off, the first step is to launch a virtual machine (VM) to host our Cortex services. In this guide, I'll be using an AWS EC2 Ubuntu instance for this purpose. However, feel free to choose any cloud or on-premises service that suits your preferences. If you're unfamiliar with setting up a VM on AWS, you can follow a step-by-step walkthrough provided [here](./aws).

<br>

## <div id="installation">💻 Installation
### Update Machine
Before diving into Cortex installation, let's ensure our machine is up to date:
```bash
Sudo apt update && sudo apt upgrade -y
```
<br>

### Install Filebeat
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

### Input 
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

### Modules
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
Connecting Filebeat to our Elasticsearch host involves specifying its configuration port and private IP. Since we've configured Elasticsearch to work over HTTPS, we add the self-signed certificate. Elasticsearch also requires credentials to log in to the service, which can be done either through a username and password or through an API key. For this lab, as it is generally more secure, we are using an API key which we will need to set up first back in our elastic host.
```yml
# ---------------------------- Elasticsearch Output ----------------------------

output.elasticsearch:
  hosts: ["172.31.8.55:9200"]
  protocol: "https"
  ssl:
    certificate_authorities: ["/etc/filebeat/certs/ca.crt"]
    certificate: "/etc/filebeat/certs/kibana/kibana.crt"
    key: "/etc/filebeat/certs/kibana/kibana.key"
```
<br>

### API Key 
To create an API key for our Filebeat host, we'll first need to set up a user specifically for the purpose of uploading logs to Elasticsearch with the minimum permissions, following the principle of least privilege.

1. **Create New User:**
   - Navigate to **Management > Stack Management > Security > Users.**
   - Click on "Create User" in the top right-hand corner.
     - **Username:** filebeat-publisher
     - **Full Name:** filebeat-publisher
     - **Email:** filebeat-publisher@local.com *(can be a made-up email)*
     - **Roles:** Editor *(will change later)*
   - Click on "Create User."

2. **Create Custom Role:**
   - Go to the **Roles** tab under the Security section in Stack Management.
   - Click on "Create Role" in the top right-hand corner.
     - **Role Name:** filebeat-publisher
     - **Cluster Privileges:** Monitor, Read_ilm, Read_pipeline, manage_ingest_pipelines, manage_pipeline
     - **Indices:** Filebeat-*
     - **Privileges:** Create_doc
   - Click on "Create Role."

3. **Update Role:**
   - Go back to the Users section, edit the filebeat-publisher user, and add the newly created role to it.
   - Ensure to hit "Update User" to apply the changes to the user.

4. **Create API:**
   - Go to **Dev Tools** located under the Management category on the main nav bar.
   - In the console, paste the following command:
     ```json
     POST /_security/api_key/grant
     {
       "grant_type": "password",
       "username": "Username", // Username for the user created for Filebeat
       "password": "Password", // Password for that user 
       "api_key" : {
         "name": "apmuser-key" // Name you want to give your API 
       }
     }
     ```
   - The return output should be simular to the following:
     ```json
     {
        "id": "VuaCfGcBCdbkQm-e5aOx",        
        "name": "my-api-key",
        "expiration": 1544068612110,         
        "api_key": "ui2lp2axTNmsyakw9tvNnw", 
        "encoded": "VnVhQ2ZHY0JDZGJrUW0tZTVhT3g6dWkybHAyYXhUTm1zeWFrdzl0dk5udw=="  
     }
     ```
   -  This key should also be listed in your **API Keys** tab where you can also manage and delete the key
   -  Any changes made to the permission/roles of the user will require the API key to be reissued for the changes to apply
> [!CAUTION] 
> Ensure that you note down this data, as you won't be able to access your key again once you leave this page.
    
<br>

5. Add to KeyStore
   - To ensure that we don't leave our API key in plain text within the YAML file, we can add Filebeat's built-in keystore, similar to what we did with Kibana.
   - Go to our `/usr/share/filebeat/bin` directory and run the following command:
     ```bash
     ./filebeat keystore add ES_API_KEY -c /etc/filebeat/filebeat.yml --path.home /usr/share/filebeat --path.data /var/lib/filebeat
     # Enter Y if you are prompted to create a keystore
     # Enter your API key in the following format id:api_key
     ```
<br>

> [!WARNING]
> As API keys are highly sensitive in nature it is always best practise to not store them locally and instead store them in a externally managed vault. AWS Secrets Manager is a perfect service for these types of situations, here is a quick guide on how to setup them up with your EC2 instances [Link](./aws)

<br>

### Enabled Modules
Modules are a simplified way of integrating common services into filebeat that have predefined input configurations and ingest pipelines. Making it a easier to have services integrated into filebeat to tracking specified log data that can be passed onto elastic. These modules can be found in the `/etc/filebeat/modules` directory where you can enable and configure which services and log data for each service is collected. For this lab as we'll be enabling and apache web server, we can enable that service by the following:
  - Rename the file as apache.yml, removing the disabled EOF which will enable the file
  - To configure the type of log data you want collected from the apache service, you can edit the file and set the log and error data to true, enabling the collection of those logs
That service is now enabled and filebeat will start collecting the log data from that service and passing it onto elastic.
<br>

### Apache Web Server
We'll be running a apache web service on our filebeats host so that we have more log data to collect and pass onto elastic. We can download/install apache with the following command:
```
sudo apt install apache2
```
We now should have apache running on our host and should be able to access the default web page on our port 80.
<br><br>

## <div id="startup">🚀 Startup
1.To configure Filebeat so that it starts automatically on boot, run the following command:
```bash
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable filebeat
```
2. Filebeat can now be started using the following:
```bash
sudo systemctl start filebeat
```
3. To check that filebeat is running and startup hasn't failed, use the following command:
```bash
sudo systemctl status filebeat
```
4. In the event that filebeat fails to start, use the following commands to look at the logs to determine the reason:
```bash
journalctl -u filebeat -n 50

# -n is the number of line you want to print out
```
6. To stop filebeat at any point while running, use the following command:
```bash
sudo systemctl stop filebeat
```
<br><br>

## Conclusion
Filebeat and elastic are now setup and should be communicating with each other. It may take some time for the logs to carry through from the filebeat host to the elastic host. Though eventually you should see logs come through on your filebeat-* index which you can find in the discovery tab.
