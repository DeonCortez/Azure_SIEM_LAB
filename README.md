# Azure_SIEM_LAB
Overview
SIEM, or Security Information and Event Management, is a comprehensive system used to identify, analyze, and address security threats before they impact operations. It aggregates log data from various network sources like firewalls, IDS/IPS, and identity systems, enabling security teams to monitor, prioritize, and respond to threats in real time. A honeypot, on the other hand, is a security tool designed to attract attackers within a controlled environment. It simulates a vulnerable system to observe attacker behavior, analyze threats, and enhance security measures. This tutorial demonstrates how to collect honeypot attack logs, query them using a SIEM, and visualize the data on a world map based on event count and geolocation.

Learning Goals:
Set up and configure Azure resources, including virtual machines, Log Analytics Workspaces, and Azure Sentinel.
Gain practical experience with Microsoft's Azure Sentinel SIEM tool.
Understand Windows Security Event logs.
Use Kusto Query Language (KQL) to query logs.
Display attack data on a dashboard using Workbooks (World Map).
Tools & Requirements:
Microsoft Azure Subscription
Azure Sentinel
Kusto Query Language (KQL - Used for world map creation)
Network Security Groups (Layer 4/3 Firewall in Azure)
Remote Desktop Protocol (RDP)
3rd Party API: ipgeolocation.io
Custom PowerShell Script by Josh Madakor
Step 1: Set Up a Microsoft Azure Subscription
Sign up for Azure to receive a $200 credit valid for 30 days.
Step 2: Deploy a Honeypot Virtual Machine
Basics

Navigate to Azure Portal.
Search for "virtual machines" and select "Create > Azure virtual machine."
Project details: Create a new resource group named "honeypotlab."
A resource group groups resources with shared lifecycle, permissions, and policies.
Instance details:
VM Name: "honeypot-vm"
Region: (US) East US 2
Availability options: No infrastructure redundancy required
Security type: Standard
Image: Windows 10 Pro, version 21H2 - x64 Gen2
Size: Default (Standard_D2s_v3 – 2vcpus, 8 GiB memory)
Administrator account: Set up a username and password for the VM. (Keep these credentials secure.)
Inbound port rules:
Public inbound ports: Allow RDP (3389)
Licensing:
Confirm licensing and proceed to Disks.
Disks

Use default settings and proceed to Networking.
Networking

Network interface: Create a new Network Security Group (NSG) under Advanced.
NSGs manage inbound and outbound network traffic rules for the VM.
Modify inbound rules:
Remove rule (1000: default-allow-rdp).
Add a new inbound rule:
Destination port ranges: *
Protocol: Any
Action: Allow
Priority: 100
Name: ALLOW_ALL_INBOUND
Review and create the VM.
Note: Configuring the firewall to allow traffic from anywhere increases the VM's visibility.

Step 3: Create a Log Analytics Workspace
Search for "Log analytics workspaces" and select "Create Log Analytics workspace."
Place it in the same resource group as the VM ("honeypotlab").
Name it "honeypot-log" and select the same region (East US 2).
Review and create the workspace.
Purpose: Logs from Windows Event Viewer and custom logs with geographic data will be ingested into this workspace.

Step 4: Configure Microsoft Defender for Cloud
Search for "Microsoft Defender for Cloud."
Under "Environment settings," select the subscription name and the Log Analytics workspace ("honeypot-log").
Settings | Defender plans:
Cloud Security Posture Management: ON
Servers: ON
SQL servers on machines: OFF
Save settings.
Settings | Data collection:
Choose "All Events" and save.
Step 5: Link Log Analytics Workspace to Virtual Machine
Search for "Log Analytics workspaces."
Select the workspace ("honeypot-log"), then go to "Virtual machines" and choose the VM ("honeypot-vm").
Click "Connect."
Step 6: Set Up Microsoft Sentinel
Search for "Microsoft Sentinel" and select "Create Microsoft Sentinel."
Choose the Log Analytics workspace ("honeypot-log") and click "Add."
Step 7: Disable Firewall on Virtual Machine
Locate the VM ("honeypot-vm") and copy its IP address.
Use RDP to log in with credentials from Step 2.
Accept the Certificate warning and select "NO" for privacy settings.
Open "Windows Defender Firewall" via Start menu and turn off the firewall for all profiles.
Apply changes and ensure the VM is reachable with a ping test.
Step 8: Run the Security Log Exporter Script
Open PowerShell ISE on the VM.
Configure Edge without signing in.
Paste the provided PowerShell script (by Josh Madakor) into PowerShell ISE and save it as "Log_Exporter" on the Desktop.
Register for a free IP Geolocation API account for 1,000 API calls per day or a paid plan for more calls.
Insert the API key into line 2 of the script and save.
Execute the script to continuously produce log data.
Function: The script exports data from Windows Event Viewer, uses IP Geolocation to extract latitude and longitude, and creates a log file at C:\ProgramData\failed_rdp.log.

Step 9: Create a Custom Log in Log Analytics Workspace
On the VM, search for "C:\ProgramData" and open the "failed_rdp" file.
Copy the content to a Notepad file on your host PC and save it as "failed_rdp.log."
In Azure, navigate to Log Analytics Workspaces > "honeypot-log" > Custom logs > Add custom log.
Upload the sample log ("failed_rdp.log") and configure record delimiter and collection paths.
Provide a name ("FAILED_RDP_WITH_GEO") and create the custom log.
Step 10: Query the Custom Log
In Log Analytics Workspaces, access "honeypot-log" > Logs.
Run a query to view data from the custom log ("FAILED_RDP_WITH_GEO_CL").
Allow time for Azure to synchronize data.
Step 11: Extract Fields from Custom Log
Right-click on any log result and select "Extract fields from 'FAILED_RDP_WITH_GEO_CL'."
Highlight and name each field, setting appropriate data types.
Save extractions and adjust as needed if results are incorrect.
Step 12: Visualize Data in Microsoft Sentinel
In Microsoft Sentinel, go to the Overview page and click "Workbooks."
Add a new workbook and edit it.
Remove default widgets and add a new query:
kql
Copy code
FAILED_RDP_WITH_GEO_CL | summarize event_count=count() by sourcehost_CF, latitude_CF, longitude_CF, country_CF, label_CF, destinationhost_CF
| where destinationhost_CF != "samplehost"
| where sourcehost_CF != ""
Set visualization to "Map" and configure settings:
Layout: Latitude/Longitude for location info.
Size by: event_count.
Color: Heatmap, Color by: event_count.
Save as "Failed RDP World Map" in the same region under "honeypotlab."
Note: The map displays failed RDP attempts and not all attacks the VM may receive.

Step 13: Decommission Resources
Search for "Resource groups," select "honeypotlab," and delete the resource group.
Confirm by typing the resource group name and applying force delete for virtual machines.
Confirm deletion to prevent unnecessary charges.
