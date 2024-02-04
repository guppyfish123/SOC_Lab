# Elastic Alerts
Since we now have our Fleet Server setup with our endpoints connected we can start to generate some traffic and setup alerts to trigger on certains conditions. 
Our Kibana EDR that we setup for our windows host will be our main log connection method and already has preconfigured alerts that we will use in this lab.
With these alerts, we'll have them exported using a websocket that we can use from Shuffle to start our security orchestration, automation and response.

## EDR
Lets generate some traffic on our windows host to see what our elastic EDR does.
Though to achieve this, we'll need to find some malicous files that we can download onto our windows host. For this head to [bazaar.abuse.ch](https://bazaar.abuse.ch/browse/) 
were a library of sample malious files can be downloaded. Pick a file from list of choices, though it would be proferred to pick a exe file, and donwload it onto your windows host.
It will come in a compressed in a ZIP file that you will have to unzip first, with the provided password from the website. Once unzip depending of the file our Elastic EDR might pickup 
the hash straight away and perminitaly remove the file from use. Other times you will need to execute the file for the EDR to pickup its signiture and determine it malious nature. 
Though both times our elastic EDR should notifiy the user that it has detected a malious file once identified and remidated.
<br>
Heading back over to our Elastic Webpage, to our ***Security*** > ***Alerts*** page. We should now see some generated alerts from our windows host, such as ***Malware Prevention Alert*** 
(These alerts may take a couple of minutes to generate from the host to elastic). Elastic Security will even let show us a timeline of events leading to what the caused the alerts and the services
& application that the file executed which can be viewed by clicking the *Cube* Icon.
<br>

## Shuffle Connector 
Connectors are powerful tool in Elastic, which enables external exporting of data to 3rd party services. Connectors can be managed in Elastic under Management > Connectors, where new connector can be added in and
existing one edited. 
***Create New Connector:***<br>
1. Navagate to the ***Connectors*** Page by going to ***Management*** > ***Connectors*** and click ***Create Connector***
