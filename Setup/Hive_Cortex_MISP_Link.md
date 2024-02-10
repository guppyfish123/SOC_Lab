# 🌐 Hive, Cortex & MISP Intergration
One of the key feature of TheHive is the seamless intergration with MISP and Cortex, widing its capabilities to be used a central point of case management. 


## MISP Connector
Before we can enable our MISP connector on our Hive host, we first need to setup an API for the connector to use to communicate from our Hive our to MISP.

### API Key
To ensure that we can track and manage our API key more easily, we'll first create a user specificly for our API.
**Create User**<br>


<br>

**Create API Key**<br>
1. Navagate to the ***Users index*** page by going to ***Administration*** > ***List Users***.
2. Click on ***View*** icon(eye) on the far most right for our new API user.
3. Down the bottom of the users details, click on ***Auth Keys*** which should direct you to the ***Authentication key Index*** page.
4. While in the ***Authentication key Index*** page click ***Add Authentication Key***.
5. ***User*** should be set your API user, ***Allowed IPs*** should be filled with your Hive host IP, and ***Expiration*** should be set to < 1 year.
6. Once completed, click submit to create your API key and you should be prompted with your API key.
> [!IMPORTANT]
> You will only be prompted with your API after you create it and will not be able to retrieve the key after woulds. So take not of the API as to not lose it. 

<br>

### TheHive Connector
Since we now have our API key to access MISP, move back over to your Hive Host where we'll need to make some changes to the configuration files to enable the MISP connector.
While on your Hive host, navagate to the file `/etc/thehive/ .yml` and make the following changes
```yml
## MISP configuration
# More information at https://github.com/TheHive-Project/TheHiveDocs/TheHive4/Administration/Connectors.md
# Enable MISP connector
play.modules.enabled += org.thp.thehive.connector.misp.MispModule
misp {
  interval: 1 hour
  servers: [
    {
      name = "MISP Server"     
      url = "https://misp.server" # Change to your MISP host 
      auth {
        type = key
        key = "<YOUR_APIKEY>" # Change to your API key
      }
      wsConfig.ssl.loose.acceptAnyCertificate = true # Disables Certificates verification since we're using SSC
    }
  ]
}
```
<br><br>

## Cortex Connector
Before we enable our Cortex connector on our Hive host, we first need to setup an API for the connector to use to communicate from our Hive our to Cortex.

### API Key
To ensure that we can track and manage our API key more easily, we'll first create a user specificly for our API used with TheHive.
**Create User**<br>
1. Navagate to the ***Users*** tab on top right to open the ***Users*** page.
2. Click ***Add User*** to create a new use on top left.
3. Supply a ***Login*** Name and ***Full Name*** to your user, set your organisation, and change ***Role*** to ***Read, Anaylyze, orgadmin***.
4. Once completed, click ***Save User*** to create a new user.

<br>
**Create API Key**<br>
1. Navagate to the ***Users*** tab on top right to open the ***Users*** page.
2. Find your API user and click ***Create API Key***
3. To view your API key click ***Reveal***.
<br>

### TheHive Connector
With your API key on hand, move over to your Hive host. We can now enable our Cortex connector to have TheHive and cortex communicate with each other.
Open the following file `/etc/cortex/` and make the following changes:
```yml
play.modules.enabled += org.thp.thehive.connector.cortex.CortexModule
cortex {
  servers = [
    {
      name = Cortex
      url = "http://cortex1:9001" # Change to your cortex host 
      auth {
        type = "bearer"
        key = "<YOUR_APIKEY>" # Change to your API key
      }
      wsConfig.ssl.loose.acceptAnyCertificate = true # Disables Certificates verification since we're using SSC
    }
  ]
  refreshDelay = 5 seconds
  maxRetryOnError = 3
  statusCheckInterval = 1 minute
}
```
<br>

## TheHive Cases
Ensure that once both MISP & Cortex connectors have been enabled and configured to restart TheHive service to apply to changes made:
```bash
sudo systemctl restart thehive
```
To confirm that TheHive service can communicate with both MISP and Cortex, open up Thehive webpage. Once logged in, on the top right hand side of the window click on your profile and in the dropdown menu go to ***About***. A new window with appear and should confirm that TheHive can now communicate with both MISP and Cortex.

### Test Case
We are going to make a case that utilizes the services of both MISP and Cortex while in TheHive:
1. To create a new case, ensure that you are in your ***Cases*** Page and click on ***Create Case +*** on the top of the page.
2. The details of our case should be as follows:
  - Title: Cortex & MISP Intergration Demo
  - Description: Deming a case to test the intergration of MISP and Cortex
  The rest of the settings can be left as defualt.
3. Once completed, click ***Confirm*** to create the case.
4. Opening our new case, navagate to the ***Observables*** tab and click the ***+*** Icon to create a new Obserbable.
5. Fill in the details of the observable as follows:
  - ***Type:*** Hash
  - ***Value:*** 4a821767c6ce723e6fb4b8d54efd52df6cbd63fc0de47a7b8b39a6ec72b4be69
  - Enable ***Is IOC***
  - Tags: Hash
6. Once completed, click ***Confirm*** to create the Observable.
7. With this Observable we can now either ***Export/Share*** it to MISP for further investigation on what this hash value may link to or...
8. Run an analyzer using our Virustotal connector we setup in Cortex by clicking the ***...*** Icon on the far most right of the Oberservable and in the dropdown select ***Run Analyzers***.
9. Cortex will then return the results from the Analyzer to the Observable in TheHive for you to view, though MISP is recommended to use if you require a deeper investigation into the details of the hash.

