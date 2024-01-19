<h1 align=center><img align="center" src="https://www.elastic.co/apple-icon-57x57.png" height="45px" width="45px">&nbsp; Fleet Server</h1>

## <div id="installation">💻 Installation
Open up your elastic webpage and navage down to Management > Fleet > Settings.

### Output 
Since we are running a single node cluster we can use our Elasticsearch service that we already have running on our host as out output. 
While in fleet settings, scroll down to output and click the pencil icon to edit the defualt elasticsearch configurations.
1. Change the ***Name*** to Elasticsearch
2. Set ***Type*** to Elasticsearch
3. Host can be set to Https://localhost:9000
4. ***Advance YAML Configurations*** enter the following
   ```yml
   ssl.certificate_authorities: [//Your CA certificate]
   ```
5. Once complete click the ***Save and Apply Settings***

### Add Fleet Server
To add our fleet server, while still in the fleet settings click the ***Add Fleet Server*** button which will open up the configuration for your fleet server you want to add.
1. Click on ***Advance*** to use the advance configurations
2. Under Select a policy for Fleet Server tick ***Collect System logs and metrics*** and then clik ***Create Policy*** to create your fleet server policy
3. Set ***Choose a deployment mode for security*** to ***Production***
4. Now under Add your Fleet Server Host set ***Name*** to Fleet and ***URL*** to https://<IP-FleetServer>:8220 then click ***Add host***
5. Click on ***Generate Service Token*** to generate your service token
6. Elastic should provide your the set of command to run attach your fleet server to your elastic host. The command will need to be modified slightly before running on your host

<br><br>


## <div id="configurations">⚙️ Configurations

## <div id="startup">🚀 Startup
