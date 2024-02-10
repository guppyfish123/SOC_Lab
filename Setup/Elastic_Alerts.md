# 🔔 Elastic Alerts
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
2. Set the following parameters of the connect as follows:
    - **Name:** Shuffle
    - **Method:** POST
    - **URL:** <SHUFFLE_WEBHOOK_TRIGGER_URI>
3. Once completed, click ***Save & test*** to contiue. We can now test our connector to ensure that it is reaching our Shuffle Flow.
4. Parse in some random JSON to the body of the request:
  ```json
  {
    {"Test": "Test"}
  }
  ```
  Click ***Test*** to send the request.
5. Moving over to our Shuffle flow we should be able to see the flow being triggered in the Webhook history. Confirming that our elastic connector is working as intended.
<br>

## Alert Response
With our Shuffle connector now setup we can now utilize it in our Alerts as a response action to trigger our SOAR. For each of our ***Malware Prevention Alert*** we'll set the alert to trigger our SOAR flow in Shuffle from elastic, through our webhook setup in our flow to send the alert data.
1. Navagating back over to our Alerts page under ***Security*** > ***Alerts***, we can edit ***Malware Prevention Alert*** by clicking on the blue text of it from our previous demo alerts with conducted ealier on
our windows host.
2. From their we should be token into the dashboard to the ***Malware Prevention Alert***, which can be edited by click the ***Edit Alert*** on the top right of the page. 
3. Under ***Actions*** enable a ***Webhook*** action with the follow parameters
    - **Webhook Connector:** <SHUFFLE_CONNECTOR>
    - **Action Frequency:** Per Each Alert - Per Rule Run
    - **If Alert Matches a Query:** Disabled
    - **If Alert is Generated During Timeframe:** Disabled
    - **Body:** { "context": "{{context:alerts}}" }
4. Save the changes made and trigger another alert from your windows endpoint to confirm response action for the alerts is working as exspected.
