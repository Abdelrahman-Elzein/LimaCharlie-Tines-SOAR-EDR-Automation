# 🛡️ LimaCharlie EDR & Tines SOAR Automated Incident Response Lab

An end-to-end Security Orchestration, Automation, and Response (SOAR) lab demonstrating automated threat detection, multi-channel alerting (Slack & Email), analyst interaction via user prompt pages, and automated endpoint isolation using **LimaCharlie EDR** and **Tines**.

## 🏗️ Topology & Architecture

1. I started by creating the diagram/topology of the SOAR automation process using "draw.io".

2. In this process, the PC malware "LaZagne.exe" compromises a Windows endpoint, which is detected by the LimaCharlie EDR sensor, triggering an alert on LimaCharlie EDR.

3. LimaCharlie then sends the detection to Tines.

4. Tines sends a message with details to Slack and also sends an email with the same detection details.

5. Tines also sends a "User Prompt" where the user/SOC analyst is asked whether they want to isolate the infected machine or not.
   -- If the user chooses to isolate it, then the infected machine is automatically isolated, a message is sent to "Slack" with the isolation status, and the message "The computer `<computer>` has been isolated" is displayed, where `<computer>` is the actual hostname of the infected endpoint.
   -- If the user chooses not to isolate it, then the message "The computer `<computer>` was not isolated, please investigate" is sent, where `<computer>` is the actual hostname of the infected endpoint.

### Topology Diagram (Draw.io)

### Tines Workflow Diagram

## ⚙️ Implementation Steps

1. On LimaCharlie, I started by creating an installation key (via the "Sensors -> Installation Keys" path), which will be used to onboard the Windows endpoint to LimaCharlie.

| Installation Key Creation 1 | Installation Key Creation 2 | 
| ----- | ----- | 
| ![Installation Key 1](Screenshots/02%20LimaCharlie-Installation-Key-Creation-1.png) | ![Installation Key 2](Screenshots/03%20LimaCharlie-Installation-Key-Creation-2.png) | 

2. Then, on that Windows endpoint, I downloaded the LimaCharlie EDR sensor. Using PowerShell, I navigated to the directory of the downloaded file and executed the following command: `.\hcp_win_x64_release_5.4.2.exe -i <sensor key>`, where `<sensor key>` is the key copied from LimaCharlie after creating the installation key.

![Successfully Onboarded Win Endpoint](Screenshots/04%20LimaCharlie-Successfully-Onboarded-Win-Endpoint.png)

3. I then downloaded the malware 'LaZagne.exe' from GitHub onto the Windows endpoint and executed it via PowerShell using the command `.\LaZagne.exe all` from its local directory.

![Win Endpoint Executed LaZagne Malware](Screenshots/05%20Win-Endpoint-Executed-LaZagne-Malware.png)

4. Next, on LimaCharlie, I navigated to the path `<The onboarded win endpoint> -> Timeline`, where I was able to view the events corresponding to the execution of the LaZagne malware. The fields from those events helped me create the required LimaCharlie EDR detection and response rule (to be honest, I modified a template, but those events were still helpful).

![LimaCharlie EDR LaZagne Execution Event](Screenshots/06%20LimaCharlie-EDR-LaZagne-Execution-Event.png)

5. The created detection and response rule was then tested against one of the previous events on LimaCharlie (under the path: `<The onboarded win endpoint> -> Timeline`), and the test was a success.

> 📄 **Exported Rule Artifact:** The complete exported rule can be found in the repository under [`Detection-Rules/Abdelrahman-Elzein-LaZagne-Malware-Detection 01.yaml`](Detection-Rules/Abdelrahman-Elzein-LaZagne-Malware-Detection%2001.yaml).

| Created Detection Rule | Tested Detection Rule | 
| ----- | ----- | 
| ![Created The Detection Rule](Screenshots/07%20LimaCharlie-Created-The-Detection-Rule.png) | ![Tested The Detection Rule](Screenshots/08%20LimaCharlie-Tested-The-Detection-Rule.png) | 

6. I then executed the malware (LaZagne.exe) on the Windows endpoint again, and detections began appearing in LimaCharlie's "Detections" tab, utilizing the detection and response rule that we had created earlier.

![LaZagne Execution Detection LimaCharlie](Screenshots/09%20LaZagne-Execution-Detection-LimaCharlie.png)

7. Moving over to Tines, I added a webhook to retrieve detections from LimaCharlie and copied the webhook's URL. Then, on LimaCharlie, I created a new output and pasted the webhook's URL into the "Destination Host" field during the creation process.

| Tines Created Webhook | LimaCharlie Created Output | 
| ----- | ----- | 
| ![Tines Created Webhook](Screenshots/10%20Tines-Created-Webhook.png) | ![LimaCharlie Created Output](Screenshots/11%20LimaCharlie-Created-Output.png) | 

8. I executed the malware (LaZagne.exe) once more on the Windows endpoint and verified that detection events started appearing on the configured Tines webhook (by clicking on the webhook, then navigating to "Events").

![Tines Webhook Detection Event](Screenshots/12%20Tines-Webhook-Detection-Event.png)

9. Next, I went to Slack, created a new channel named "alerts", and copied its channel ID.

![Created Slack Alerts Channel](Screenshots/13%20Created-Slack-alerts-Channel.png)

10. I then visited the URL "https://explained.tines.com/en/articles/8713184-slack-authentication-guide" and followed the documentation to successfully integrate Tines with Slack. Afterward, I added a new "Send a message (Slack action)" block in Tines and connected the previously created webhook to it. I clicked on the action block and pasted the previously copied channel ID of the "alerts" channel into the "Channel / User ID" field displayed in the right configuration panel.

![Slack Tines Integration](Screenshots/14%20Slack-Tines-Integration.png)

11. Finally, I went back to the webhook configured in Tines, clicked on it, selected "Events", and navigated through the existing detection event. I copied the path of every relevant field (any field of interest that I wanted to include in the detection messages sent to Slack's alerts channel) by clicking the copy icon on its left and choosing "Copy path".

![Copying Paths Of Relevant Event Fields](Screenshots/15%20Copying-Paths-Of-Relevant-Event-Fields.png)

12. I went back to the "Slack (Send a message)" action in Tines. In the "Message" field, I pasted the field paths that I copied earlier, while also adding a descriptive label before each one to indicate the information it contains:

```text
Title: <<retrieve_detections.body.cat>>
Time: <<retrieve_detections.body.detect.routing.event_time>>
Username: <<retrieve_detections.body.detect.event.USER_NAME>>
Hostname: <<retrieve_detections.body.detect.routing.hostname>>
Source IP: <<retrieve_detections.body.detect.routing.int_ip>>
File path: <<retrieve_detections.body.detect.event.FILE_PATH>>
Command line: <<retrieve_detections.body.detect.event.COMMAND_LINE>>
Routing SID: <<retrieve_detections.body.detect.routing.sid>>
File hash: <<retrieve_detections.body.detect.event.HASH>>
Detection URL: <<retrieve_detections.body.link>>
```

![Tines Slack Pasted Field Paths](Screenshots/16%20Tines-Slack-Pasted-Field-Paths.png)

13. Then, I added a "Send Email" action in Tines and connected the previously created webhook to it. I configured the options for "Recipients", "Reply to", "Sender name" (as "Alerts"), "Subject" (as "[Security Alert] <<retrieve_detections.body.cat>>") to identify the email as a security alert and dynamically include the detection title in the subject line, making it easier for analysts to recognize and identify incoming alerts, and the "Body". In the "Body" field, I followed the same process as the "Slack (Send a message)" action by pasting the exact same field paths with brief descriptions. However, unlike before, I had to add an HTML `<br>` tag between each field path:

```text
Title: <<retrieve_detections.body.cat>>
<br>
Time: <<retrieve_detections.body.detect.routing.event_time>>
<br>
Username: <<retrieve_detections.body.detect.event.USER_NAME>>
<br>
Hostname: <<retrieve_detections.body.detect.routing.hostname>>
<br>
Source IP: <<retrieve_detections.body.detect.routing.int_ip>>
<br>
File path: <<retrieve_detections.body.detect.event.FILE_PATH>>
<br>
Command line: <<retrieve_detections.body.detect.event.COMMAND_LINE>>
<br>
Routing SID: <<retrieve_detections.body.detect.routing.sid>>
<br>
File hash: <<retrieve_detections.body.detect.event.HASH>>
<br>
Detection URL: <<retrieve_detections.body.link>>
```

(I did this so the detection details sent via email would be properly formatted. During testing, I noticed that without adding the `<br>` tags between the field paths, all fields appeared as a single messy paragraph when the email was sent—unlike the "Slack (Send a message)" action, where the listed fields format perfectly without them.)

![Tines Send Mail Action Config](Screenshots/17%20Tines-Send-Mail-Action-Config.png)

14. I also added a "Page" in Tines, connected the previously created webhook to it, named it "User Prompt", and made it available to "Members of this Tines tenant". I set the "Page behavior" to "Show success message" and defined the "Success message" as "Thank you, you can now close this window." Next, I clicked the "Edit page" button, added a "Heading" section with the text "Abdelrahman-Elzein-SOAR-EDR-Project", and added a "Rich text" section containing the same field paths I copied earlier:

```text
Title: <<retrieve_detections.body.cat>>
Time: <<retrieve_detections.body.detect.routing.event_time>>
Username: <<retrieve_detections.body.detect.event.USER_NAME>>
Hostname: <<retrieve_detections.body.detect.routing.hostname>>
Source IP: <<retrieve_detections.body.detect.routing.int_ip>>
File path: <<retrieve_detections.body.detect.event.FILE_PATH>>
Command line: <<retrieve_detections.body.detect.event.COMMAND_LINE>>
Routing SID: <<retrieve_detections.body.detect.routing.sid>>
File hash: <<retrieve_detections.body.detect.event.HASH>>
Detection URL: <<retrieve_detections.body.link>>
```

| Tines Added New Page | Tines Edited The New Page | 
| ----- | ----- | 
| ![Tines Added New Page](Screenshots/18%20Tines-Added-New-Page.png) | ![Tines Edited The New Page](Screenshots/19%20Tines-Edited-The-New-Page.png) | 

15. While still in the "Edit page" view, I added a "Boolean" component to the page, naming it "Isolate?" and setting the "Labels" to "Yes" and "No".

16. I also added a "Button" component and set its "Label" to "Submit".

17. Next, I added two conditions, naming one "Yes" and the other "No". Both have their "Rules" field set to "`<<user_prompt.body.isolate>>`". For the "Yes" condition, I set "is equal to" to "true", and for the "No" condition, I set it to "false". I then connected the previously created "User Prompt" page to both of these conditions.

| Tines Created Condition 1 | Tines Created Condition 2 | 
| ----- | ----- | 
| ![Tines Created Condition 1](Screenshots/20%20Tines-Created-Condition-1.png) | ![Tines Created Condition 2](Screenshots/21%20Tines-Created-Condition-2.png) | 

18. I copied the original "Slack (Send a message)" action and pasted it to create a duplicate version. I connected the "No" condition to it and modified its "Message" section to:

```text
The computer: "<<retrieve_detections.body.detect.routing.hostname>>" was not isolated, please investigate.
```

(I used the exact same hostname field path that I utilized in the previous Slack and email actions.)

![Tines No Condition Slack Message](Screenshots/22%20Tines-No-Condition-Slack-Message.png)

19. I then went back to the Tines dashboard by clicking "Personal" on the top left to exit the playbook story. Next, I clicked on "Personal -> Credentials -> New -> Manual creation -> Text" and named the credential "LimaCharlie-credential1" (any name can be used). Regarding the "Value" field, I went to LimaCharlie under "Access Management -> REST API", copied the "ORG JWT" value, returned to Tines, and pasted it into the "Value" field. In the "URLs and Domains" field, I entered `*.limacharlie.io` so that this credential can only be used with the LimaCharlie domain and its subdomains. Finally, I clicked "Save" to complete the creation of the text credential.

> **Note:** Keep in mind that for a Tines story/playbook to be published, it must be moved out of the "Personal" team folder into "Your First Team" or any other team folder. While initial building steps were performed inside the "Personal" folder, the story was eventually moved to a team folder for publishing.

| Creating Text Credential 1 | Copied ORG JWT | Creating Text Credential 2 | 
| ----- | ----- | ----- | 
| ![Creating Credential 1](Screenshots/23%20Tines-Creating-LimaCharlie-Text-Credential.png) | ![Copied ORG JWT](Screenshots/24%20LimaCharlie-Copied-ORG-JWT.png) | ![Creating Credential 2](Screenshots/25%20Tines-Creating-LimaCharlie-Text-Credential-2.png) | 

20. Afterward, I searched for LimaCharlie in Tines and added the "Isolate Sensor" template, I then connected the "Yes" condition to it. When configuring it, the default "URL" option was `https://api.limacharlie.io/v1/<<sid>>/isolation`, I then changed it to `https://api.limacharlie.io/v1/<<retrieve_detections.body.detect.routing.sid>>/isolation`, replacing the default sensor ID field path of "`<<sid>>`" with "`<<retrieve_detections.body.detect.routing.sid>>`" (the same one used in the earlier "Slack (Send a message)" & "Send Email" actions). In the "Authorization" field, I specified the "`<<CREDENTIAL.limacharlie_credential1>>`" credential by hovering near the field, clicking the "+" symbol, and selecting "Value -> CREDENTIAL -> limaCharlie-credential1".

![Tines Isolate Sensor LimaCharlie Template](Screenshots/26%20Tines-Isolate-Sensor-LimaCharlie-Template.png)

21. I then searched for LimaCharlie in Tines again and added the "Get Isolation Status" template, I then connected the previously added "Isolate Sensor" template to it. When configuring it, the default "URL" option was `https://api.limacharlie.io/v1/<<sid>>/isolation`, I then changed it to `https://api.limacharlie.io/v1/<<retrieve_detections.body.detect.routing.sid>>/isolation`, replacing the default sensor ID field path of "`<<sid>>`" with "`<<retrieve_detections.body.detect.routing.sid>>`" (the same one used in the earlier "Slack (Send a message)" & "Send Email" actions). In the "Authorization" field, I specified the "`<<CREDENTIAL.limacharlie_credential1>>`" credential by hovering near the field, clicking the "+" symbol, and selecting "Value -> CREDENTIAL -> limaCharlie-credential1".

![Tines Get Isolation Status Template](Screenshots/27%20Tines-Get-Isolation-Status-LimaCharlie-Template.png)

22. Finally, I copied the first "Slack (Send a message)" action to create another duplicate and connected the "Get Isolation Status" template to it. I modified its "Message" field to:

```text
Isolation Status: <<get_isolation_status.body.is_isolated>>
The computer: "<<retrieve_detections.body.detect.routing.hostname>>" has been isolated.
```

(I grabbed the "`<<get_isolation_status.body.is_isolated>>`" field path by clicking on the previous "Get Isolation Status" template, navigating to "Events", selecting a past event, expanding it until I found `"is_isolated": true`, clicking the copy icon to its left, and choosing "Copy path". The hostname path is the same one I used previously.)

(Hint: Keep in mind that if some field paths or field values do not appear in a Tines action/block's "Events" settings, you may need to re-execute the "LaZagne" malware once again. This usually happens if you have newly or recently added that action/block after the malware's execution, meaning the events related to the malware's execution have not reached that action/block yet.)

![Tines Isolate Sensor Slack Message](Screenshots/28%20Tines-Isolate-Sensor-Slack-Message.png)

## 🧪 Workflow Execution & Proof of Concept

### Endpoint Isolation Approved Execution

| Workflow 1 | Workflow 2 | Workflow 3 | Workflow 4 | 
| ----- | ----- | ----- | ----- | 
| ![Ex 1](Screenshots/29%20Tines-Endpoint-Isolation-Workflow-Execution-1.png) | ![Ex 2](Screenshots/30%20Tines-Endpoint-Isolation-Workflow-Execution-2.png) | ![Ex 3](Screenshots/31%20Tines-Endpoint-Isolation-Workflow-Execution-3.png) | ![Ex 4](Screenshots/32%20Tines-Endpoint-Isolation-Workflow-Execution-4.png) | 

| Workflow 5 | Workflow 6 | Workflow 7 | Workflow 8 | 
| ----- | ----- | ----- | ----- | 
| ![Ex 5](Screenshots/33%20Tines-Endpoint-Isolation-Workflow-Execution-5.png) | ![Ex 6](Screenshots/34%20Tines-Endpoint-Isolation-Workflow-Execution-6.png) | ![Ex 7](Screenshots/35%20Tines-Endpoint-Isolation-Workflow-Execution-7.png) | ![Ex 8](Screenshots/36%20Tines-Endpoint-Isolation-Workflow-Execution-8.png) | 

| Workflow 9 | Workflow 10 | Workflow 11 | 
| ----- | ----- | ----- | 
| ![Ex 9](Screenshots/37%20Tines-Endpoint-Isolation-Workflow-Execution-9.png) | ![Ex 10](Screenshots/38%20Tines-Endpoint-Isolation-Workflow-Execution-10.png) | ![Ex 11](Screenshots/39%20Tines-Endpoint-Isolation-Workflow-Execution-11.png) | 

### Endpoint Isolation Declined Execution

| Declined 1 | Declined 2 | Declined 3 | 
| ----- | ----- | ----- | 
| ![Declined 1](Screenshots/40%20Tines-Endpoint-Isolation-Declined-1.png) | ![Declined 2](Screenshots/41%20Tines-Endpoint-Isolation-Declined-2.png) | ![Declined 3](Screenshots/42%20Tines-Endpoint-Isolation-Declined-3.png) | 

## 👨‍💻 Author

**Abdelrahman Elzein**
