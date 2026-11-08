# 🛡️ LimaCharlie EDR & Tines SOAR Automated Incident Response Lab

[![SOC Automation](https://img.shields.io/badge/SOC-Automation-blue.svg)](https://github.com/Abdelrahman-Elzein/LimaCharlie-Tines-SOAR-EDR-Automation)
[![EDR](https://img.shields.io/badge/EDR-LimaCharlie-orange.svg)](https://limacharlie.io/)
[![SOAR](https://img.shields.io/badge/SOAR-Tines-purple.svg)](https://www.tines.com/)
[![Slack Integration](https://img.shields.io/badge/Alerts-Slack-green.svg)](https://slack.com/)

An end-to-end Security Orchestration, Automation, and Response (SOAR) lab demonstrating automated threat detection, multi-channel alerting (Slack & Email), analyst interaction via user prompt pages, and automated endpoint isolation using **LimaCharlie EDR** and **Tines**.

---

## 🏗️ Topology & Architecture

### 1. Architecture Diagram (Draw.io)
1. I started by creating the diagram/topology of the SOAR automation process using "draw.io".
2. In this process, the PC malware "LaZagne.exe" compromises a Windows endpoint, which is detected by the LimaCharlie EDR sensor, triggering an alert on LimaCharlie EDR.
3. LimaCharlie then sends the detection to Tines.
4. Tines sends a message with details to Slack and also sends an email with the same detection details.
5. Tines also sends a "User Prompt" where the user/SOC analyst is asked whether they want to isolate the infected machine or not.
-- If the user chooses to isolate it, then the infected machine is automatically isolated, a message is sent to "Slack" with the isolation status, and the message "The computer <computer> has been isolated" is displayed, where <computer> is the actual hostname of the infected endpoint.
-- If the user chooses not to isolate it, then the message "The computer <computer> was not isolated, please investigate" is sent, where <computer> is the actual hostname of the infected endpoint.

![Draw.io Topology](Screenshots/01%20SOAR-Workflow-Diagram%20(Draw.io).png)

### 2. Tines Storyboard Overview
![Tines Workflow Diagram](Screenshots/01%20SOAR-Workflow-Diagram%20(Tines).png)

---

## ⚙️ Implementation Steps

### Step 1: LimaCharlie Installation Key Creation
1. On LimaCharlie, I started by creating an installation key (via the "Sensors -> Installation Keys" path), which will be used to onboard the Windows endpoint to LimaCharlie.

| Installation Key Creation - Step 1 | Installation Key Creation - Step 2 |
| :---: | :---: |
| ![LimaCharlie Installation Key 1](Screenshots/02%20LimaCharlie-Installation-Key-Creation-1.png) | ![LimaCharlie Installation Key 2](Screenshots/03%20LimaCharlie-Installation-Key-Creation-2.png) |

---

### Step 2: Onboarding Windows Endpoint
2. Then, on that Windows endpoint, I downloaded the LimaCharlie EDR sensor. Using PowerShell, I navigated to the directory of the downloaded file and executed the following command: .\hcp_win_x64_release_5.4.2.exe -i <sensor key>, where <sensor key> is the key copied from LimaCharlie after creating the installation key.

![Successfully Onboarded Windows Endpoint](Screenshots/04%20LimaCharlie-Successfully-Onboarded-Win-Endpoint.png)

---

### Step 3: Executing Malware (LaZagne.exe)
3. I then downloaded the malware 'LaZagne.exe' from GitHub onto the Windows endpoint and executed it via PowerShell using the command .\LaZagne.exe all from its local directory.

![Executing LaZagne Malware](Screenshots/05%20Win-Endpoint-Executed-LaZagne-Malware.png)

---

### Step 4: LimaCharlie EDR Event Analysis & Detection Rule Creation
4. Next, on LimaCharlie, I navigated to the path <The onboarded win endpoint> -> Timeline, where I was able to view the events corresponding to the execution of the LaZagne malware. The fields from those events helped me create the required LimaCharlie EDR detection and response rule (to be honest, I modified a template, but those events were still helpful).

![LimaCharlie EDR LaZagne Execution Event](Screenshots/06%20LimaCharlie-EDR-LaZagne-Execution-Event.png)

5. The created detection and response rule was then tested against one of the previous events on LimaCharlie (under the path: <The onboarded win endpoint> -> Timeline), and the test was a success.

| Created Detection Rule | Tested Detection Rule |
| :---: | :---: |
| ![Created Detection Rule](Screenshots/07%20LimaCharlie-Created-The-Detection-Rule.png) | ![Tested Detection Rule](Screenshots/08%20LimaCharlie-Tested-The-Detection-Rule.png) |

6. I then executed the malware (LaZagne.exe) on the Windows endpoint again, and detections began appearing in LimaCharlie's "Detections" tab, utilizing the detection and response rule that we had created earlier.

![LaZagne Execution Detection LimaCharlie](Screenshots/09%20LaZagne-Execution-Detection-LimaCharlie.png)

---

### Step 5: Integrating LimaCharlie with Tines Webhook
7. Moving over to Tines, I added a webhook to retrieve detections from LimaCharlie and copied the webhook's URL. Then, on LimaCharlie, I created a new output and pasted the webhook's URL into the "Destination Host" field during the creation process.

| Tines Webhook Created | LimaCharlie Output Created |
| :---: | :---: |
| ![Tines Created Webhook](Screenshots/10%20Tines-Created-Webhook.png) | ![LimaCharlie Created Output](Screenshots/11%20LimaCharlie-Created-Output.png) |

8. I executed the malware (LaZagne.exe) once more on the Windows endpoint and verified that detection events started appearing on the configured Tines webhook (by clicking on the webhook, then navigating to "Events").

![Tines Webhook Detection Event](Screenshots/12%20Tines-Webhook-Detection-Event.png)

---

### Step 6: Slack Integration & Notification Action
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
