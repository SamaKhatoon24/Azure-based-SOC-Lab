# Azure-based-SOC-Lab
This report outlines the steps taken to create a home-based Security Operation Center (SOC) using Microsoft Azure. The project focuses on deploying a honeypot virtual machine designed to lure and log real-world cyber threats. Using Microsoft Sentinel as a Security Information and Event Management (SIEM) tool, I will showcase how to monitor unsuccessful login attempts, trace the geographic origins of attackers, and visualize emerging security threats.

**Overview**

Here are the key components of my SOC Lab:

1. Create Free Azure Subscription: Set up a free Azure account to access cloud resources needed for the lab.

2. Create the Honey Pot (Azure Virtual Machine): Deploy a Windows 10 VM, expose it to the internet, and configure it as a honeypot to attract attackers.

3. Logging into the VM and inspecting logs: Simulate failed login attempts, collect security event logs, and analyse unauthorised access attempts using Event Viewer.

4. Log Forwarding and KQL: Forward security logs to Log Analytics Workspace and use KQL (Kusto Query Language) to analyse login attempts.

5. Log Enrichment and Finding Location Data: Import geolocation into Microsoft Sentinel to map attacker IP addresses to their geographic locations.

6. Attack Map Creation: Use Sentinel Workbooks to visualise attacker activity on a map, providing insights into global threat sources.

![](https://miro.medium.com/v2/resize:fit:1400/1*4HIPbZ-NBoWEOERyzKytXw.png)

**Creating the Honeypot (Virtual Machine)**

What is a Honeypot?

![](https://miro.medium.com/v2/resize:fit:1144/1*ochdvvemavJggEaKx9l2xg.png)

In cybersecurity, a honeypot is a decoy system or network designed to attract and study attackers. It acts as a trap, luring attackers away from real systems and providing valuable insights into their methods and tools.

Go to: [https://portal.azure.com](https://portal.azure.com/) and search for virtual machines

![](https://miro.medium.com/v2/resize:fit:1400/1*LVddvrjlGrGy1ikt_zMN2g.png)

In this stage, I will deploy a Windows 10 virtual machine (VM) and assign it a name designed to entice potential attackers — such as “CORP-NET-SOUTH-1.” Using a name that appears to belong to a legitimate corporate network increases the likelihood of the system being targeted.

To deliberately introduce vulnerabilities, I will start by removing the existing Remote Desktop Protocol (RDP) rule from the Inbound Security Rules. This will expose RDP traffic directly to the internet. Since RDP is a widely used protocol for remote administration, making it publicly accessible enlarges the attack surface and makes it easier to detect unauthorized access attempts in the security logs.

To further increase visibility and attract more malicious activity, I will create a new inbound rule to allow traffic on all ports, significantly amplifying the system’s exposure to potential threats.

**Steps:**

- Create a resource group

Name: honeypot-lab

A resource group acts as a logical container for managing and organizing related Azure resources.

**Configure the Virtual Machine**

- Virtual Machine Name: Corp-net-south-1
- Region:canada (Choose the region **you** are in)
- Availability Options: No infrastructure redundancy required
- Security Type: Standard
- Image: Windows 10 Pro, version 22H2 — x64 Gen2
- VM Architecture: x64
- Size: Use the default configuration (Standard_D2s_v3–2 vCPUs, 8 GiB RAM)

Administrator account configuration

Set up a username and password that will be used to access the virtual machine.

Important: These login credentials must be remembered, as they are required to sign in to the VM.

Inbound port rules

- Public inbound ports: Allow selected ports
- Selected port: RDP (3389), which will enable remote desktop access

Licensing

- Confirm the licensing agreement
- Click “Next: Disks >” to continue

Before:

![](https://miro.medium.com/v2/resize:fit:1400/1*OfXqTtCnF5t6Nvf27i_W9g.png)

After:

![](https://miro.medium.com/v2/resize:fit:1400/1*Eb-QjR-AMOnA-ln1-u_RCg.png)

In this stage, I’ve logged into my VM with the application “Windows App” where I can identify and test my VM later on in this document, however I’ve gone on and removed the Window Firewalls.

Here are the steps: Start -> wf.msc -> properties -> all off

![](https://miro.medium.com/v2/resize:fit:1376/1*LfxvsYOxFy8qqkvBY4qrKA.png)

Lastly, I’ll ping the VM to check if the machine is receiving bytes without any restrictions, and in this case the firewall is not preventing unauthorised devices from discovering the VM. The honeypot is now set and ready! The packets should be receiving and should look something like this. If it is timed out you may not have switched off all of the firewalls.

![](https://miro.medium.com/v2/resize:fit:1290/1*YceWlLLt1uXHfaY2tIgFEw.png)

Fail 3 logins with any name and password: To inspect some logs, I’ll be doing a simple failed login into the VM a few times so I can generate some understanding on how authentication failures are recorded

![](https://miro.medium.com/v2/resize:fit:692/1*U8ts-gKDMEcnEUIRhjkvIw.png)

Opening up Event Viewer and Inspecting the Security Logs:

![](https://miro.medium.com/v2/resize:fit:1400/1*-7HnADWGMcMtMB5lmeCObg.png)

By examining the Windows Security logs, we can focus on Event ID 4625, which signals a failed login attempt. This event provides valuable information such as the source network address, helping to pinpoint the origin of the attempted intrusion.



Join Medium for free to get updates from this writer.

Subscribe

Integrating Event Viewer data with Microsoft Sentinel enables the creation of detailed logs within a Security Information and Event Management (SIEM) system. This setup allows for automated alert triggers, making the detection and response to security incidents much faster and more efficient — an approach that reflects real-world security operations.

Microsoft Sentinel also supports regulatory compliance standards like GDPR and ISO 27001, which are essential for organizations focused on maintaining strong data protection frameworks.

![](https://miro.medium.com/v2/resize:fit:1316/1*5KHqG_Eh1a26b7gbMJH1LQ.png)

Here’s a filtered version of the Event ID 4625:

![](https://miro.medium.com/v2/resize:fit:1400/1*aUrrMZ-VhQlzRGn2DH9SeQ.png)

**Log Forwarding and KQL (Kusto Query Language)**

Kusto Query Language (KQL) is designed for querying, analyzing, and visualizing large volumes of log and event data, especially within Azure Log Analytics and Azure Monitor.

At this stage of the project, I enabled the Microsoft Sentinel free trial through an Azure trial subscription. This allowed me to configure Sentinel to collect Windows Security Event logs. With this setup, security logs from Windows systems are continuously collected, analyzed, and correlated. This enables the detection of suspicious activity — such as failed login attempts — and allows for real-time alerts and incident response actions.

![](https://miro.medium.com/v2/resize:fit:1400/1*hFK5oC4JaSE49-iWbMKWpQ.png)

Next, I set up a centralized log repository called a Log Analytics Workspace (LAW). This serves as the main hub for collecting, analyzing, and querying security logs. To facilitate this, I used the Azure extension known as “AzureMonitorWindowsAgent,” which is responsible for gathering data from the virtual machine. Once collected, the data is sent to the LAW, where I can begin conducting detailed security investigations.

![](https://miro.medium.com/v2/resize:fit:1400/1*RXji4Tib39IVnVdQ1_QWhA.png)

The Data Collection Extension Tool:

![](https://miro.medium.com/v2/resize:fit:1400/1*_dNBLEYQK4Q9YUS_9tkGdA.png)

The log report of the LAW

![](https://miro.medium.com/v2/resize:fit:1400/1*L8euYQEI8s-D8SHNMFj-pA.png)

By querying for logs within the Log Analytics Workspace (LAW) using Kusto Query Language (KQL), I was able to identify multiple failed login attempts from the same IP address over an extended period.

The logs revealed a brute-force attack originating from an IP address located in Malaysia, with a high frequency of login attempts spanning from 4:18 PM to 8:02 PM. This activity was consistent, suggesting an automated attack trying to gain unauthorized access.

[**202.165.17.0/24 IP RangeAll about 202.165.17.0/24**
ipinfo.io](https://ipinfo.io/202.165.17.229?source=post_page-----dfbf23b0cc86---------------------------------------)

To save time and find specific logs, I can use KQL to allow me to filter through large amounts of data, quickly and precisely.

KQL Commands:

SecurityEvent

| where EventID == 4625

| project TimeGenerated, Computer, Account, AccountType, IpAddress, LogonType, FailureReason, LogonProcessName

### **Log Enrichment and Finding Location Data:**

I’ll be observing the SecurityEvent logs in the Log Analytics Workspace as there is no location data, only IP address, which we can use to derive the location data. I’m going to import a spreadsheet known as a “Sentinel Watchlist” which contains geographic information for each block of IP addresses.

Spreadsheet File: https://drive.google.com/file/d/13EfjM_4BohrmaxqXZLB5VUBIz2sv9Siz/view

![](https://miro.medium.com/v2/resize:fit:1400/1*BRarbn8JD8h_JlhLXABNFQ.png)

![](https://miro.medium.com/v2/resize:fit:1400/1*qMjWaFVBv8iCAyU6i75sOg.png)

As you can see below, I was able to identify the attacker’s location along with their attempted login attempts shown above. This was a clear sight that the attacker was trying to gain access to my Azure account. Better luck next time!

![](https://miro.medium.com/v2/resize:fit:1400/1*AoUvd_94Hct2CRD9Ihumdg.png)

KQL Command to observe the logs that have geographic information, so you can see where the attacks are coming from:

let GeoIPDB_FULL = _GetWatchlist(“geoip”);

let WindowsEvents = SecurityEvent

| where IpAddress ==

| where EventID == 4625

| order by TimeGenerated desc

| evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network); WindowsEvents

*SecurityEvent*

*| where EventID == “4625”*

I will be using this command in italics beforehand to find the IP address that I want to analyze.

### **Attack Map Creation**

For the last stage, I’ll be generating a visual geographic map of all the attacks that were happening towards my VM. This provides insights into threat sources and attack trends.

The following step is to create a new Workbook

![](https://miro.medium.com/v2/resize:fit:1400/1*hVg86IxhIwMQPxvJxRBAuw.png)

This part will also simulate real-world cyber threats, enabling better understanding of security incidents, logging mechanisms, and threat analysis techniques.

Attack Map 16/6/2025 at 11pm:

![](https://miro.medium.com/v2/resize:fit:1400/1*9588lC3MgP2pzse8kDLpsw.png)

Attack Map 17/6/2025 at 11.30am:

![](https://miro.medium.com/v2/resize:fit:1400/1*q9hCDIImedaDdH5eGyN7vQ.png)

By reviewing the attack maps, I noticed that the majority of malicious traffic targeting my Azure virtual machine originated from Australia. Within a 12-hour period, the number of connection attempts ranged from 151 to 480, suggesting either aggressive automated scanning or a focused attack attempt.

Conclusion

Through this project, I successfully created a home-based Security Operation Center (SOC) using Microsoft Azure to monitor and analyze real-world cyber threats. By deploying a honeypot virtual machine and integrating it with Microsoft Sentinel, I was able to collect, examine, and visualize security log data. This setup provided valuable insights into how attackers operate, revealing their methods and geographic origins.

Through log forwarding, KQL analysis, and visualisation tools like Sentinel Workbooks, I effectively tracked failed login attempts and identified potential threats. This hands-on experience reinforced the importance of proactive security monitoring and the power of cloud-based SIEM solutions. Overall, this project highlights how organisations can leverage Azure’s security tools to enhance their threat detection and response capabilities.
