# SOAR-EDR Automation Project

## Objective
This project aims to enhance incident response efficiency by integrating a SOAR platform (Tines) with an EDR tool (Limacharlie). It involves creating custom detection rules to identify suspicious endpoint activity and building automated playbooks to streamline responses. The system is connected to Slack and email to deliver real-time alerts and ensure rapid communication. Through this project, I gained hands-on experience in security automation, threat detection, and orchestrating workflows across multiple tools.

## Lesson Learned
- Gained hands-on experience creating and automating a runbook/playbook
- Creating a custom detection and response rule using Limacharlie
- Implementation of a SOAR (Tines) and EDR (Limacharlie)
- Understood the core functionalities and capabilities of modern security automation and response tools.
- Navigated API integrations and automation logic to coordinate between tools seamlessly.
- Integration of a SOAR and EDR with Slack and email to provide real-time alerts/notifications
- Enabled real-time incident visibility through Slack and email, ensuring swift awareness and action.



## Tools
- Tines (SOAR)
- LimaCharlie (EDR Platform)
- Slack
- VirtualBox (hypervisor)
- Windows 10 (target machine)

## Part 1: Playbook Workflow
![1 Playbook Workflow drawio (3)](https://github.com/user-attachments/assets/356dc24f-4729-44c4-ac02-dbca380703c8)
1. LimaCharlie (EDR) will detect the hacking tool (Lazagne.exe)
2. LimaCharlie sends data/alert over to Tines (SOAR), which will have a playbook (called stories in Tines) that was created
3. The playbook will send an email and Slack message with the details of the alert
4. In Tines, it will give the option of isolating the machine or not
5. If "no", LimaCharlie will not isolate the machine and send an update on the isolation status
6. If "yes", LimaCharlie will isolate the machine and send an update on the isolation status

## Part 2: Install and set up LimaCharlie
1. Create a LimaCharlie Account
![2 signup for limacharlie](https://github.com/user-attachments/assets/05f95fea-15c5-4f36-9c2e-10cac9c06f9f)
![3](https://github.com/user-attachments/assets/2174696d-7566-4ca8-873f-ec4dcfe35168)

2. Create an installation key. This will be used to enroll your target/test machine into LimaCharlie
![4 creating installation key](https://github.com/user-attachments/assets/43ae223d-a01c-4fa1-93d3-a1f4dc326cf8)
![5 installation key](https://github.com/user-attachments/assets/f407b93c-8365-4992-8e66-7ed2c7013a21)
![6 key is made](https://github.com/user-attachments/assets/a974710f-188d-4927-9351-92d1d8592d2c)
- You should now see the new installation key "SOAR-EDR Project"

3. Enroll the target machine into LimaCharlie
![7 scroll down](https://github.com/user-attachments/assets/ce124e16-5ab7-4eaa-ae84-053f7c51bb5b)
- In the "Installation Keys" tab, scroll down to "Sensor Download" and "EDR" and copy the link of your version of the machine you want to enroll (Windows x64)
![8 new tab will start downloading](https://github.com/user-attachments/assets/a47404e0-7102-4fb1-b545-36d9a04b1beb)
- Open a new tab and paste the link to start downloading the executable file for LimaCharlie
![9 copy sensor key](https://github.com/user-attachments/assets/ee26f2e1-3371-4f5e-ad6f-6333cdf054af)
- As it downloads, copy the "Sensor Key" from the newly created installation key 
![10 open powershell and run command](https://github.com/user-attachments/assets/203e6073-ba90-452c-9c33-eed90d3e76cd)
- Open PowerShell as Admin on the target machine and find the location of the downloaded file for LimaCharlie.
- Use the command:
```bash
  hcp_win_x64_release_4.33.4.exe -i (sensor key)
```
![11 agent installed](https://github.com/user-attachments/assets/dfcc8443-3402-4d84-ad16-42f72556a0a8)
- LimaCharlie has been installed, and the target machine has been enrolled

4. Double-check LimaCharlie and target machine has been installed correctly
![12 successfully install limacharlie](https://github.com/user-attachments/assets/1b7381f1-f96f-4095-bb30-04888aeef270)
- In LimaCharlie, check the "Sensor Lists" to see if the target machine was added into LimaCharlie
![13 check services](https://github.com/user-attachments/assets/3efeb062-6251-4400-89a5-f36e8748d125)
- Open up "Services" and find LimaCharlie

## Part 3: Generate telemetry using Lazagne.exe (credential access password recovery tool)
1. Turn off "Real-Time Protection"
![14 turn off real time protection](https://github.com/user-attachments/assets/badd8440-575d-4d9f-bed0-7e14d69722ab)
2. Download Lazagne.exe from github
![15 download lazagne](https://github.com/user-attachments/assets/291809df-f78f-4820-997f-d651a2e74501)
3. Open the executable file in PowerShell and use the "all" option to generate telemetry
4. Back in LimaCharlie, go to "Sensor List", select your machine, scroll down to "Timeline" and search for "lazagne"
![16 limacharlie has detected lazagne](https://github.com/user-attachments/assets/453f01a4-07f5-4d18-9926-155ccff26ebe)
- LimaCharlie has detected Lazagne.exe

## Part 4: Create a detection and response rule in LimaCharlie
1. Select the "Automation" tab and then "D & R" (Detection and Response)
   ![17 D R](https://github.com/user-attachments/assets/e433c811-4b88-4cc6-9360-9c1faf0a8077)
- Knowing Lazagne.exe is a credential access tool, type in credential in the search bar to obtain some premade detection rules
2. Copy the premade rule and paste it into a new rule by selecting “Add Rule”
  ![18 Add rule](https://github.com/user-attachments/assets/93ca335e-a0ed-43cd-9df5-794c9861dd9e)
- modify the detection and response rule:
```bash
Detection Rule:
Events:
- NEW_PROCESS
- EXISTING_PROCESS
Op: and
Rules:
- op: is windows
- op: or
  rules:
- case sensitive: false
Op: ends with
Path: event/FILE_PATH
Value: Lazagne.exe
- case sensitive: false
Op: ends with
Path: event/COMMAND_LINE
Value: all (#there will be false positives like ipconfig /all)
- case sensitive: false
Op: contains
Path: event/COMMAND_LINE
Value: Lazagne
- case sensitive: false
Op: is
Path: event/HASH
value : ‘3cc5ee93a9ba1fc57389705283b760c8bd61f35e9398bbfa3210e2becf6d4b05’
```
The detection rule is saying:
- The event type must be either a New_Process or Existing_Process, AND must be Windows.
- Ignore case sensitivity, AND
- The FILE_PATH must end with "Lazagne.exe" OR COMMAND_LINE ends with "all" OR COMMAND_LINE contains "lazagne" OR HASH == lazagne (hash-value)

```bash
Response Rule:
- action: report
Metadata:
	Author: Timmy
	Description: Detects Lazagne (SOAR-EDR TOOL)
	Falsepositives:
	- To the moon!
	Level: medium
	Tags:
	- attack.credential_access
Name: Timmy - HackTool - Lazagne (SOAR-EDR)
```
The response rule is saying:
- When the conditions for this rule are met, LimaCharlie will report the event. It does not automatically block, isolate, or take other remediation steps — just logs and alerts.
- In this case, it will report to Tines for automation
3. Test out the new rule
  <br clear="left"/>![20 copy event to test](https://github.com/user-attachments/assets/08d59eb2-33c1-428d-ad5b-0d647b75abdb)
  - Go back to "Timeline" and copy the event/process lazagne.exe generated
  <br clear="left"/>![21 paste to test new rule](https://github.com/user-attachments/assets/d6cfd61a-d800-4a6c-ba73-7f6dfb3fe22f)
  - Back in the "D & R" tab, paste the event under the "Target Event" tab
  <br clear="left"/>![22 successful test](https://github.com/user-attachments/assets/2d86cfa3-396a-4409-9430-fa30bf0aba6c)
  - Underneath, select "Test Event". It appears the new rule has successfully detected the event
4. Test if LimaCharlie was able to detect the event
![23 detection successful](https://github.com/user-attachments/assets/fb885ba0-6375-4167-a692-32abaa7d532a)
- In PowerShell again, use the "all" option for lazagne
- Head to the "Detection" tab in LimaCharlie
- Looks like the detection was successful

## Part 5: Setup Slack and Tines for automation

  





