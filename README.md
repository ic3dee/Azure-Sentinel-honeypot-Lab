# Azure-Sentinel-honeypot-Lab
 In this lab by Josh Madakor, I set up Azure Sentinel (SIEM) and connected it to a live VM acting as a honeypot to observe real-time RDP brute force attacks from around the world. We’ll use a custom PowerShell script to find attackers' geolocations and plot them on the Azure Sentinel map.


# Step 1: Creating the Virtual Machine in Azure

To set up the virtual machine (VM) that will serve as our honeypot, follow these steps:

1. Navigate to the Azure portal and search for virtual machines.
2. Click on "Create" to initiate the creation process.
3. Choose a suitable resource group name, such as "honeypot-lab", to logically group all resources related to this project.
4. Enter a name for the virtual machine, for example, "honeypot-vm", and select the appropriate region.
5. Keep the default settings for image and size, but ensure to specify a username and password for accessing the VM later.
6. Proceed to the next steps, leaving the settings as default unless specific customization is required.
7. Under "Networking", configure the network security group (NSG), which acts as a firewall. Click on "Advanced" to access more options.
![VMscreenshot](https://github.com/JacobAragon1/Azure-Sentinel-honeypot-Lab/assets/72784819/7aec5bb3-1abd-48fc-b86a-2f3bd59849b2)
![CreatingHoneyPot](https://github.com/JacobAragon1/Azure-Sentinel-honeypot-Lab/assets/72784819/882d4734-dc73-46e1-b7d2-49b80e2a46cb)


# Step 2: Allowing All in Firewall Settings

To open our logging server to potential attacks for the purpose of this lab, we configure the firewall to allow all inbound traffic from the public internet. We start by creating a new inbound rule in the network security group (NSG) associated with our virtual machine. This rule will essentially allow any type of traffic to reach the VM. By setting the destination port to "*", specifying the action as "allow", and assigning a low priority, such as 100, we ensure that all traffic is permitted. While this configuration is not recommended for production environments, it serves the purpose of our lab, where we aim to make the virtual machine easily discoverable to simulate real-world attack scenarios. After creating the rule, we proceed with deploying the changes, making our honeypot accessible to potential attackers
![honeypotconfiguration](https://github.com/JacobAragon1/Azure-Sentinel-honeypot-Lab/assets/72784819/36b22935-7ecb-4d9e-82d5-33113e463b6d)



# Step 3: Creating Log Analytics Workspace

To begin collecting and analyzing logs from our honeypot VM, we need to create a Log Analytics Workspace (LAW). This workspace serves as the central repository for storing and analyzing log data, including Windows event logs and the custom logs containing geographic information about potential attackers. By creating this workspace, we establish the foundation for our SIEM implementation and enable Microsoft Sentinel to visualize geodata on a map.

1. Navigate to the Azure portal and search "Log Analytics Workspace".
2. Choose the resource group that was created earlier and provide a name for the workspace, such as "la-honeypot-1".
3. Select the region and proceed to review and create the workspace.
4. After creating the workspace, enable the gathering of VM logs in Microsoft Defender For Cloud. Navigate to Environment, enable gathering logs from the VM into the Log Analytics Workspace, and configure the data collection settings to "All Events".
5. Connect the Log Analytics Workspace to the VM by selecting the workspace and navigating to the virtual machine tab. Click "Connect" to establish the connection.
![creatinglawanalytics](https://github.com/JacobAragon1/Azure-Sentinel-honeypot-Lab/assets/72784819/65e11617-35db-44bb-9aab-ba1bbff45b8d)
![enablinggatheringvmsinWS](https://github.com/JacobAragon1/Azure-Sentinel-honeypot-Lab/assets/72784819/11acc08a-4c43-41d7-85bf-a54cbe60d979)
![Connectingloganalyticstovm](https://github.com/JacobAragon1/Azure-Sentinel-honeypot-Lab/assets/72784819/14c81490-3043-4272-a670-43cdbe4586c2)



# Step 4: Setting up Azure Sentinel

In this step, we configure Microsoft Sentinel, our SIEM tool, to visualize and analyze the attack data collected by our Log Analytics Workspace. 

1. Navigate to the Azure portal and search for "Microsoft Sentinel".
2. Click on "Microsoft Sentinel" and then "Create".
3. Select the Log Analytics Workspace we created earlier (e.g., "la-honeypot-1").
4. Add Microsoft Sentinel to this workspace and let it initialize.
5. While Azure Sentinel is being set up, ensure the Log Analytics Workspace is connected to the VM.
6. Navigate back to the Log Analytics Workspace and ensure the connection to the VM is established.
7. Browse to the virtual machine, click on the public IP address, and log into the VM using Remote Desktop Protocol (RDP).
![SentinelSetup](https://github.com/JacobAragon1/Azure-Sentinel-honeypot-Lab/assets/72784819/edc50483-1e8c-4f1b-91d9-687f6363aa07)


# Step 5: Logging into the VM and turning Off Windows Firewall on the VM

To ensure our honeypot VM is easily discoverable on the internet and responds to ICMP echo requests (pings), we need to turn off the Windows Firewall. This makes it quicker for attackers to find our VM, increasing the likelihood of capturing meaningful log data.
![RDPintoVM](https://github.com/JacobAragon1/Azure-Sentinel-honeypot-Lab/assets/72784819/8768b5f8-e6ca-42dd-bdf6-4835a0158787)

1. Log into the VM using Remote Desktop Protocol (RDP).
2. Open the Start menu, type wf.msc, and press Enter to open the Windows Defender Firewall with Advanced Security.

In the Firewall properties, turn off the firewall for all profiles:

Domain Profile: Set "Firewall state" to "Off".
Private Profile: Set "Firewall state" to "Off".
Public Profile: Set "Firewall state" to "Off".
Click "Apply" and then "OK" to save the changes.

Minimize the VM window and on your local machine, open Command Prompt.
Ping the VM's public IP address with a perpetual ping command: ping -t.
By turning off the Windows Firewall, we ensure our VM responds to ICMP echo requests, making it easier for attackers to discover it.
![TurnOffWindowsFirewallVM](https://github.com/JacobAragon1/Azure-Sentinel-honeypot-Lab/assets/72784819/24a2ad38-20a7-406b-91cf-76d1eaf9a3e1)



# Step 6: Create Geolocation.io API key, download and configure PowerShell Script

In this step, we will crate our own unique geolocation API key that will give us the location based off the attackers IP addresses and download a PowerShell script created by Josh Madakor to gather geolocation data of IP addresses from failed logon attempts. This script will help us map the origin of these attacks.

Get Geolocation.io API Key:
- Go to ipgeolocation.io and sign up for a free API key.
- After signing up, you will receive an API key.

Download the PowerShell Script:
- Visit the GitHub repository containing the script (https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1) and copy it.
- On your VM, open PowerShell ISE and paste the copied script into a new script window.
- Replace your Geolocation.io API key into the 2nd line of the Powershell script.
- Save the script on your desktop as "log_exporter.ps1".
![CreatingPowerShellScript](https://github.com/JacobAragon1/Azure-Sentinel-honeypot-Lab/assets/72784819/5e46bd6d-7a99-4bc7-b2dd-6e6a7c89ddcf)
![LoadedCustomLogsInLAW](https://github.com/JacobAragon1/Azure-Sentinel-honeypot-Lab/assets/72784819/484aca9e-0474-4449-9dbd-ba049302da9b)

Run the script by clicking the "Run Script" button (green arrow) in PowerShell ISE.
The script continuously monitors the Event Viewer for event ID 4625 (failed logon attempts), extracts the source IP addresses, queries the ipgeolocation.io API for geolocation data, and writes this data to a log file at C:\ProgramData\failed_rdp.log. (Toggle on hidden folders if you can't see your ProgramData folder at first.)
![RunScriptToGetGeoData](https://github.com/JacobAragon1/Azure-Sentinel-honeypot-Lab/assets/72784819/a01421e5-d8d5-4a53-9486-74c7de932098)

Verify Script Functionality:
- On your local machine (not the VM), open Command Prompt.
- Attempt to log into the VM using RDP with incorrect credentials to generate failed logon events.
- On the VM, navigate to C:\ProgramData\failed_rdp.log to check the new entries logged by the script.


# Step 7: Create Custom Log in Log Analytics Workspace

To enable Microsoft Sentinel to visualize our custom log data with geolocation, we need to create a custom log in our Log Analytics Workspace (LAW).

1. Prepare the Custom Log File:
- On your VM, navigate to C:\ProgramData\.
- Copy the contents of failed_rdp.log.
- On your local machine, open Notepad and paste the copied content.
- Save this file on your desktop as failed_rdp.log.

2. Create Custom Log in Log Analytics Workspace:
- In the Azure portal, search for "Log Analytics Workspace" and select your workspace (e.g., honeypot).
- Go to Tables > Create > New custom log (MMA Based).
- Upload the custom log file (failed_rdp.log) that you saved on your desktop.
- Click Next to see a preview of the log. Ensure it looks correct.
- Set the collection path to C:\ProgramData\failed_rdp.log.
- Click Next, name the custom log (e.g., failed_rdp_with_geo_CL), and click Next again.
Finish by clicking Create.
![CreatingCustomLogInLAWtoBringInOurCustomLog](https://github.com/JacobAragon1/Azure-Sentinel-honeypot-Lab/assets/72784819/7be1d448-8fd5-4b4a-bf21-c427b976b998)
![LoadedCustomLogsInLAW](https://github.com/JacobAragon1/Azure-Sentinel-honeypot-Lab/assets/72784819/e51ca641-32be-4a49-aa39-e6fdd4ac5415)

3. Verify Custom Log Data:
- In your Log Analytics Workspace, go to Logs.
- Search for your custom log by typing failed_rdp_with_geo_CL.
- Initially, you might not see the log data immediately as it takes some time to sync.
- You can also check existing logs by searching for SecurityEvent and running the query to see Windows Event Logs being ingested.
By creating this custom log, we ensure that our log data, including geolocation information, is ingested into Azure Log Analytics Workspace. This step is crucial for mapping and visualizing attacks in Microsoft Sentinel.

# Step 8: Visualize the Data in Microsoft Sentinel
1. Create a Workbook
- Go to your Microsoft Sentinel instance.
- Navigate to Workbooks and create a new workbook.
- Click Add > Add Query.
- Microsoft removed the extraction tools from Azure, so in order to correctly segment the different fields from the failedrdp.log raw data, use this query:
  
FAILED_RDP_WITH_GEO_CL
| extend latitude = extract("latitude:([0-9.-]+)", 1, RawData),
         longitude = extract("longitude:([0-9.-]+)", 1, RawData),
         destinationhost = extract("destinationhost:([^,]+)", 1, RawData),
         username = extract("username:([^,]+)", 1, RawData),
         sourcehost = extract("sourcehost:([^,]+)", 1, RawData),
         state = extract("state:([^,]+)", 1, RawData),
         country = extract("country:([^,]+)", 1, RawData),
         label = extract("label:([^,]+)", 1, RawData),
         timestamp = extract("timestamp:([^,]+)", 1, RawData)
| project TimeGenerated, Computer, latitude, longitude, destinationhost, username, sourcehost, state, country, label, timestamp
| summarize event_count=count() by sourcehost, latitude, longitude, country, label, destinationhost
| where destinationhost != "samplehost"
| where sourcehost != ""

![SetupMapInSentinel](https://github.com/JacobAragon1/Azure-Sentinel-honeypot-Lab/assets/72784819/702b5ddb-0903-4566-aab6-4cb46773fb47)
- Choose map as your visualization type.
