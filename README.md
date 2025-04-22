# SOAR-EDR Automation Project

## Objective
This project aims to enhance incident response efficiency by integrating a SOAR platform (Tines) with an EDR tool (Limacharlie). It involves creating custom detection rules to identify suspicious endpoint activity and building an automated playbook to streamline responses. The system is connected to Slack and email to deliver real-time alerts and ensure rapid communication. Through this project, I gained hands-on experience in security automation, threat detection, and orchestrating workflows across multiple tools.

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
1. Make an account for both Slack and Tines

2. Create an #alerts channel in Slack
   ![24 slack alert channel](https://github.com/user-attachments/assets/1e2bf11d-3def-4671-8706-8fe7582da0d4)

3. Establish a link between LimaCharlie and Tines
   ![25 tines webhook URL](https://github.com/user-attachments/assets/214541ff-4a18-4b69-b92f-f3508a8ad85b)
   - In Tines, click and drag "webhook" into the workspace and copy the webhook's URL
   In LimaCharlie, select "Outputs" and then "Add Output"
<br clear="left"/>![26 limacharlie ouput](https://github.com/user-attachments/assets/a57605f7-abde-4ed2-97f2-5cd2b96ebd5a)
<br clear="left"/>Select "Detection" and then find Tines
<br clear="left"/>![27 save output](https://github.com/user-attachments/assets/805c7849-7313-4ca7-bbdc-3e93ae4adcb3)
	- Name the output and paste Tine's Webhook URL into "Destination Host"

4. Test if Tines and LimaCharlie are connected
   <br clear="left"/>![28 limacharlie tines test](https://github.com/user-attachments/assets/e65021fd-41f9-4a09-b733-e8e5bffeee29)
   - Use the “all” option again from Lazagne.exe and select “Refresh Samples”
   - If it generates the detection, then Tines and LimaCharlie are now connected
![29 tines has limacharlie](https://github.com/user-attachments/assets/d0e7ab4c-2d8b-4753-b11f-e7009d042355)
	- To double-check, in Tines select the webhook, and choose an event, you should see the event that was detected from LimaCharlie

5. Connect Tines with Slack
![30 slack adding tines](https://github.com/user-attachments/assets/d197dd97-3aa2-46cf-997d-fb76f658b585)
- Select "Automation", search Tines and then "Add"
<br clear="left"/>![31 your teams tab](https://github.com/user-attachments/assets/0c52bac8-5406-4323-ac10-9e2c2b596728)
- In Tines, select "Your first team"
![32 click credentials and new](https://github.com/user-attachments/assets/8344b4b4-8687-4314-86af-dcfd1c8062fa)
- Select “Credentials” and “+New” and then select “Slack”
![33 Use tines's app for slack](https://github.com/user-attachments/assets/9537fc5c-7d3f-40c7-8d9a-9a9c73054956)
- A new window will pop up, and select “Use Tines’s app for Slack”
![34 Tines and slack ](https://github.com/user-attachments/assets/76f09a07-6ecd-4cdf-b247-12752ac9370f)
- Once installed, Slack should now be in your credentials in Tines

## Part 6: Create a Playbook/Stories (in Tines)
1. Add and test Slack and email to the playbook
   ![35 Tines sending message](https://github.com/user-attachments/assets/e5cb17cb-7d65-4fc7-bacc-7ffac76aca38)
   - In the "Template" tab, select Slack and drag it into the story
   - Select the “Send a message” option
   - The channel ID will be the #alert channel in slack
   - The test message will be “Hello world!”
   ![36 slack's channel ID](https://github.com/user-attachments/assets/58df123c-9a78-41c9-bd5c-9137cbb9f32a)
	- Right click the #alerts channel, and you should find the channel's ID
<br clear="left"/>![37 Run Tines to Slack test](https://github.com/user-attachments/assets/6f41a8eb-ebcb-4e58-a408-b31689d371d5)
- Back in Tines's story, connect the Webhook with Slack and on Slack select "Run" to test
![38 it works](https://github.com/user-attachments/assets/57b5eb78-6ae3-4271-bed7-3c958091a29e)
- Back in Slack, the message was successfully sent. It works!

2. Do the same thing with email. 
   ![39 same thing with temp email](https://github.com/user-attachments/assets/15a0b19c-9079-4660-ba53-890418612387)
- I will be using a temporary email
![40 email test works](https://github.com/user-attachments/assets/9f1c8b2b-848d-4b36-8ce1-50a9e845d5ca)
- Email was successfully sent
  
4. Create the User Prompt of “Do you want to isolate the computer? Yes or no?”
   ![41 prompt or page](https://github.com/user-attachments/assets/26e5f9c4-fff6-4cb9-a63b-e306c89f96d0)
   - Go to tools and select "Page" (Page will make our prompt)
   - I will name this “User Prompt”, description: "Isolate computer (Yes/no) with a Success message: Thank you. Please close this window."
   - Connect the webhook with the page

5. Gather interesting fields to be included in "Message Details"
   ![42 interesting fields](https://github.com/user-attachments/assets/c499e7d0-983a-4eee-b123-371a0bff5e02)
   - Go to the "webhook" in the story and select the previous event
   - Here you'll find the fields that will be included
   - Copy the field path of each 
  <br clear="left"/>![43 interesting fields in notepad](https://github.com/user-attachments/assets/a2bfc058-2a7f-4af5-ba97-a5be7ab7cd4c)
	- Use Notepad to organize all fields in this order
```bash
Title: <<retrieve_detection.body.cat>>
Time: <<retrieve_detection.body.routing.event_time>>
Hostname: <<retrieve_detection.body.routing.hostname>>
IP: <<retrieve_detection.body.routing.int_ip>>
Username: <<retrieve_detection.body.detect.event.USER_NAME>>
Filepath: <<retrieve_detection.body.detect.event.FILE_PATH>>
Commandline: <<retrieve_detection.body.detect.event.COMMAND_LINE>>
Sensor ID: <<retrieve_detection.body.routing.sid>>
Link to detection: <<retrieve_detection.body.link>>
```
<br clear="left"/>![44 slack message with details](https://github.com/user-attachments/assets/1e1e04ed-c8cb-496b-9db7-747dadb8da10)
- Copy and paste the field paths into Slack's message in Tines's story
<br clear="left"/>![45 slack alert sent success](https://github.com/user-attachments/assets/698acbf5-8b44-402d-b221-255237675ef4)
	- Test to see if message was sent, and looks like it was successful
   ![46 same but with labels](https://github.com/user-attachments/assets/5101ffe7-408f-4061-9ace-ad6e8e413482)
   - Added labels some others can understand what they are looking at
![47 email with details](https://github.com/user-attachments/assets/a76da95a-cbcf-4595-bb9f-c7b0f625bc9e)
	- Do the same thing with email
 	- This is in HTML format, so if you want, edit the email message in Tines to be in HTML format, adding <“br”> with each detail to make it more readable

6. Edit User Prompt
   <br clear="left"/>![48 user prompt test](https://github.com/user-attachments/assets/f8088707-ace8-4cbd-9928-751734e56b07)
   - Add message details and visit the page
   - The message is showing (Yes and No buttons are not functional yet)

7. Add functionality to the "No" button
   ![49 No option part1](https://github.com/user-attachments/assets/1e5156af-3ae6-4ebd-b19b-3a706bebafe0)
   - Select the trigger icon
   - The value for “rule” will only show if you tested out the “page” message from before

<br clear="left"/>Create a Slack message after selecting "no" in the user prompt
![50 no option part2](https://github.com/user-attachments/assets/193f294f-5ba8-4bb9-a998-0ee66b712d13)
- Copy and paste Slack, connect the trigger “no” with the new Slack
- Edit the message to say: “the computer: (computer name) was not isolated, please investigate”
  <br clear="left"/>![51 slack message sent for NO](https://github.com/user-attachments/assets/ee8eba69-c9bd-403e-a809-0fc18032a2f7)
  - Test starting from the User Prompt
  - Message was sent successfully
  
8. Add functionality to the "Yes" button (automated response)
   <br clear="left"/>Copy and paste the already-made trigger and change the rule to equal “true” rather than “false”
![52 add limacharlie](https://github.com/user-attachments/assets/a04dfddb-d73a-4972-98e5-dd93c6dcd673)
- Add LimaCharlie from "Template" and select the "Isolate sensor" option
- In "URL", change the path to the sensor ID path
<br clear="left"/> Create a new credentials for LimaCharlie
![53 limacharlie org API key](https://github.com/user-attachments/assets/47bd2da8-dc1a-48d9-be18-bf3c34ba2c22)
- LimaCharlie's documentation recommends using an org’s API key when possible
- To find it, go back to LimaCharlie, "Access Management", "Rest API", and you should see “Org JWT” for the API key
![54 create limecharlie credentials](https://github.com/user-attachments/assets/b599d47b-0dbf-43e6-aa36-ffe77fe0c8a9)
- Back in Tines, create new credentials and select “text”
- Paste the org's JWT into value
- In “URLs and Domains”, put “*.limacharlie.io” (ensures that only this credential can only be used towards this site/subdomains)

<br clear="left"/> Run the event from the User Prompt again
![55 isolate successful part1](https://github.com/user-attachments/assets/0d7518d9-e0c2-4abd-a64a-dc973457eb2e)
![56 isolated part2](https://github.com/user-attachments/assets/98e47a99-eec8-4746-ae17-06980726af54)
- The computer has been isolated!
- ***I was doing this on a VM, so the entire VM was disconnected from the internet after it was isolated***
 	- Make sure to use another machine and not your test machine when testing the "yes" prompt, otherwise your test machine will disconnect

9. Create a Slack message that describes the machine's isolation status
   Add another LimaCharlie template and choose “Get Isolation status” with the same URL and credentials as the other LimaCharlie. Add another Slack
   ![57 slack message for YES](https://github.com/user-attachments/assets/dcd33e1d-e44f-4c63-8a34-b5a653d532b2)
   - To get the path values in the message, connect every template and run the event again.
   - This time, I did this on my host computer since doing it on the VM would only isolate the VM itself, leaving it with no internet connection.
  
10. Final Test
    <br clear="left"/>Rerun the event from the start to double-check that everything is working
    ![58 Slack final](https://github.com/user-attachments/assets/bfbc8128-fac7-489d-a965-542ffa69d69d)
    - Slack has sent the detailed message about the affected machine
    ![59 email final](https://github.com/user-attachments/assets/7d502414-9129-4f79-80cd-9917bbdbf2a2)
	- Email also received the detailed message about the affect machine
![60 slack final no](https://github.com/user-attachments/assets/f36433e7-dc19-4ddb-a1f4-d4251ea83819)
	- Slack's message when selecting "No" from the User Prompt
![61 slack final Yes](https://github.com/user-attachments/assets/cfc9b39a-84fb-4f10-a08b-29731ea3e0bd)
	- Slack's Message when selecting "Yes" from the User Prompt
![56 isolated part2](https://github.com/user-attachments/assets/0b0d3754-ca77-4e03-9cba-e1ef79790164)
	- The test machine has been isolated and disconnected from the network
  	- The automated response was successful!

<br clear="left"/>The completed automated playbook/story
![62 final tines story or playbook](https://github.com/user-attachments/assets/44979f59-2848-4daa-a90d-749865d69742)


















