# Shuffle SOAR
Shuffle with is diverse library of connectors to industy leading applications is a key service for our security orchestration, automation, and response plateform. 
Shuffle will be the starting point of where are alerts are first sent from elastic when triggered. Shuffle will handle actions such as Notification through email, 
the creation of cases in TheHive with Key information from elastic, and triggering Crotex Analyze and exporting to MISP. Automating this flow of alerts enable security
administrators to spend more time in investigating the threate in a more effeicent manner with all the key information they can have in one place.

## Create Flow
1. To create our first flow in Shuffle, move over to your Shuffle webpage and login.
2. To create a new flow, while in the ***Workflows*** page click on ***+*** Icon in a box. Provide a Name and Description for your flow and click ***Done***.
3. First we'll start off by adding the trigger to our flow by tragging in the ***Webhook*** trigger into our flow. A unique ***Webhook URI*** we be generated which will be required in making your connector in elastic.
5. Connect the webhook trigger to your ***Change Me***, which will be used to filter the data from the webhook.
6. Drag in a new app called ***TheHive***, connecting it to the ***Change Me*** variable and name the action ***Create_Case***. As authentication to connect to your Hive host an API key will be required.
7. Selecting the action of ***Create Case*** which will make a case in our TheHive host with our elastic data. In order to filter the data from the webhook which is in JSON format,
use the variable ***$change_me*** with the the Key to the desired value.
     - **Title:** $change_me.context.kibana.alert.rule.name
     - **Description:** Alert Information:
        $change_me.context.kibana.alert.reason
        
        Affected Host:
          HostName - $change_me.context.host.hostname
          IP - $change_me.context.host.ip
          MAC - $change_me.context.host.mac
        
        Event Code: $change_me.context.event.code
        Severity: $change_me.context.event.severity
8. From the case that we have just created we want to be able to add an observable to that case to run an analyzer on. Drag in another TheHive action connecting it to the previous one.
9. Setting the action to ***Create Observable in Case*** fill in the required parameters:
    - **CaseID:** $create_case.caseId
    - **Data:** $change_me.context.file.hash.sha256
    - **Datatype:** hash
    - **Ignoresimilarity** false
    - **IOC:** true
    - **Iszip:** false
    - **Message:** $change_me.context.file.pe.original_file_name
    - **Pap:** 2
    - **Sighted:** true
10. To trigger the Run Anayzer on our observable, drag in another TheHive action connecting it to our previous one and set the action to ***Run Analyzer***.
11. Fill in the required paramters to the action:
    - **Cortex Id:** Cortex
    - **Analyzer Id:** VirusTotal_GetReport_3_1
    - **Artifact Id:** $create_observable.body.#._id
12. Lastly we want to add a notification email action connect to our ***Creat Case*** Action to warm security administration when the flow is triggered due to an alert.
13. Drag the Email action into the flow and link it to the ***Create Case*** action. An Shuffler api key will be require for this which can be found in your user settings.
14. Once the desired parameters to the email are entered the flow is now ready to use.
