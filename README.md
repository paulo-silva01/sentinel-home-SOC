# Sentinel Home Security Operations Center

In this project, we are creating a basic at-home SOC in the cloud using Microsoft Azure. We will create a virtual machine (VM) that is exposed to the internet as a honeypot, forwarding those logs to a repository where we can use Microsoft Sentinel to analyze them

---

### Prerequisites
- Computer with stable internet connection
- Free Azure Account - Possible to do with Free Subscription

## Create our Free Azure Account
1. Sign up: [Azure Free Account](https://azure.microsoft.com/en-us/free/)
2. Login: [Azure Portal](https://portal.azure.com)

---

# Table of Contents

- [Setting up the Virtual Machine](#Setting-up-the-Virtual-Machine)
- [Accessing and Pinging our Virtual Machine](#Accessing-and-Pinging-our-Virtual-Machine)
- [View Raw Logs on the Virtual Machine](#View-Raw-Logs-on-the-Virtual-Machine)
- [Creating our Log Repository](#Creating-our-Log-Repository)
- [Connecting our VM to LAW](#Connecting-our-VM-to-LAW)
- [Querying our Log Repository with KQL](#Querying-our-Log-Repository-with-KQL)
- [Uploading our Geolocation Data to the SIEM](#Uploading-our-Geolocation-Data-to-the-SIEM)
- [Inspecting our Enriched Logs](#Inspecting-our-Enriched-Logs)

## Setting up the Virtual Machine

1. Once we have logged into Microsoft Azure we can create our Resource Group
    - Create a resource group
    - Name it anything you would like
    - Region: East US 2
    - Review and create
<img width="1000" alt="1" src="https://github.com/user-attachments/assets/83c650eb-4487-49aa-8d57-5ae67ed9f627" />

2. Create the Virtual Network
    - Locate virtual network page
    - Create virtual network using a suitable name
    - Same region as resource group
    - Review and create
    - Create
<img width="1000" alt="2" src="https://github.com/user-attachments/assets/d614864e-d49d-4134-80b8-38132e921fe2" />

3. Create the Virtual Machine (Honeypot)
    - Navigate to the virtual machine page
    - Create
    - Use the resource group we created
    - Do not use an obvious honeypot name
    - Region: East US 2
    - Image: Win10
    - Size: Standard_D2s_v3
    - Create a username and password, but do not forget it
    - Select "I confirm I have an eligible Windows 10/11 license with multi-tenant hosting rights."
4. Disks
    - OS Disk Type: Premium SSD
5. Networking
    - Select the Virtual network you created previously
6. Monitoring
    - Boot Diagnostics: Disabled
7. Review and create
<img width="1000" alt="3" src="https://github.com/user-attachments/assets/8bdc1784-edef-4711-948e-344150060279" />

    Navigate back to the resource groups and we can see the different types of services we are using
8. We have to open up the firewall on our virtual machine for potential attacks to come in
<img width="1000" alt="4" src="https://github.com/user-attachments/assets/65b0ad3c-592c-48b0-8c47-5b08324192be" />

   Delete the RDP inbound rule so we can create another that allows any traffic
9. Add new inbound rule that allows traffic through
     - Source: Any
     - Port: Any (*)
     - Destination: Any (*)
     - Service: Custom
     - Destination Port Ranges: Any (*)
     - Priority: 100
     - Name: Renaming to DANGER infront lets us know that this is used within a controlled environment
<img width="1000" alt="5" src="https://github.com/user-attachments/assets/4fb2dbd3-2c21-4f13-807d-9a9c206bbae2" />

---

## Accessing and Pinging our Virtual Machine

1. Now we need to get logged into our virtual machine to disable to internal firewall
    - We will be using Remote Desktop Protocol
    - MacOS will need to download Windows Remote Desktop
    - Use the given Public IP address on the virtual machine to remote desktop into it
<img width="1000" alt="6" src="https://github.com/user-attachments/assets/d3d6cdcf-eeca-4e1a-9d0e-5867a14f9bd0" />
<img width="1000" alt="7" src="https://github.com/user-attachments/assets/580e654a-0874-466f-a697-3275658ce17c" />

2. Once connected to the virtual machine, type "wf.msc" in the start menu
3. Navigate to Windows Defender Firewall Properties
    - Domain Profile Firewall State: Off
    - Private Profile Firewall State: Off
    - Public Profile Firewall State: Off
    - Select Apply, then OK
<img width="1000" alt="8" src="https://github.com/user-attachments/assets/7ee2540f-a386-4a96-9e58-85e67c6e721f" />
<img width="1000" alt="9" src="https://github.com/user-attachments/assets/4bc5dde2-ba10-4236-8ab7-c5d996204299" />
<img width="1000" alt="10" src="https://github.com/user-attachments/assets/53a0589e-c7fd-4bac-9169-26c4118632f8" />

4. Next go back to your main computer and access Powershell, or the Command line on Windows machines
     - Terminal for MacOS
5. Type ping (IP address of virtual machine) to check if the machine is on
     - If you are receiving bytes from the IP address then the machine is on and working
     - Ctrl+C to stop the pings
<img width="1000" alt="11" src="https://github.com/user-attachments/assets/10544822-2218-4d30-b5e9-bc53d9baa479" />

---

## View Raw Logs on the Virtual Machine

1. Turn off the virtual machine and attempt to reconnect with incorrect credentials
     - I will use the username of ATTACKER and an incorrect password for lab purposes
<img width="1000" alt="12" src="https://github.com/user-attachments/assets/1e28b2b8-58b1-4737-8ccf-d8f226e938b3" />

    Ping a few times to find the log easier
2. After a few incorrect attempts to access the virtual machine, Log in with the correct credentials
     - Navigate to Event Viewer in the start menu of the virtual machine
     - This is where if something happens on the computer, it will create a log
     - Windows Logs -> Security
     - Notice we have multiple Audit Failures, with an Event ID of 4625, which stands for attempted breach of our system
     - Selecting one of the Audit Failures, we see the name ATTACKER displayed, which is the username we used to try and gain access to our virtual machine
<img width="1512" alt="13" src="https://github.com/user-attachments/assets/8e53f0da-a41f-48c4-92e0-318a6c8e4c27" />

    This is only from our computer, now that the firewall is down, anyone can attempt to access our virtual machine

---

## Creating our Log Repository

1. Now that we know the logs are working and anyone can ping our computer, it is time to send those logs to a repository where we can collect the data
2. In Azure on our main computer, we navigate to Log Analytics Workspace
     - Create one
     - Place inside of the same resource group created earlier
     - Log Analytics Workspace (LAW)
     - Same time zone, East US 2
<img width="1000" alt="15" src="https://github.com/user-attachments/assets/6bf4d3fb-db75-4dea-a013-0b01492e3a42" />

3. Next, create a Sentinel instance which is a Security Information and Event Management (SIEM) system
     - Search for Sentinel
     - Create Microsoft Sentinel
     - Log Analytics Workspace (LAW) must be completed before proceding
     - Select the LAW previously created
  <img width="1000" alt="16" src="https://github.com/user-attachments/assets/f1d72caf-8a8d-426b-804f-a5196ea34856" />

    This is going to connect the two programs together
4. Next we need to configure the Azure Monitoring Agent Security Event Connector
5. This will create the connection between our virtual machine, and the LAW which will allow us to pass the logs to our SIEM
<img width="1508" alt="17" src="https://github.com/user-attachments/assets/88ccabe2-560b-4e68-b867-d7c53c47d8c5" />

---

## Connecting our VM to LAW

1. Go Content hub under Microsoft Sentinel
     - Search for security event
     - Find Windows Security Events
     - install on the right hand side
     - Once completed, select manage
     - Find the Windows Security Event via AMA item
     - Select Open connector page on the right
<img width="1000" alt="19" src="https://github.com/user-attachments/assets/c8ad2aab-982f-4eb3-b846-c925765c78e3" />

2. Now we are going to create a data collection rule in the Windows Security Events via AMA
<img width="1000" alt="20" src="https://github.com/user-attachments/assets/9e21cba4-36ec-4c26-aebd-4fec78d4cf93" />

    Collection rule is used by the virtual machine to forward logs
3. Navigate to the virtual machines page
     - Under settings go to Extensions + applications
     - Here you will see the Monitor Agent we just installed
     - May take some time to finish installing
<img width="1000" alt="21" src="https://github.com/user-attachments/assets/a7b52a43-1ad3-4a71-b8d7-611e713eee8e" />

4. Go to our Log Analytics Workspace
     - Under logs we should slowly be receiving logs from the virtual machine
     - Type SecurityEvent on the right
     - Press Run
     - Nothing should appear initially
<img width="1000" alt="22" src="https://github.com/user-attachments/assets/af292220-2fef-4409-af5a-5e37366e3a8e" />

    Wait about 30 minutes to an hour to let the logs come in

---

## Querying our Log Repository with KQL

1. Once we wait for a while for the logs to come in, run SecurityEvent again to see all the attempted entries onto our virtual machine
     - The account name and type being displayed
<img width="1000" alt="23" src="https://github.com/user-attachments/assets/d3ac2e81-0531-4166-a7fb-fc5f4662f016" />

2. By using Kusto Query Language (KQL) we can filter the logs
     - | where Account == "\\ACCOUNTNAME" resolves the the specific name of the account
     - | project TimeGenerated, Account, Computer, EventID, Activity, IpAddress resolves to only showing these categories in the log
<img width="1000" alt="24" src="https://github.com/user-attachments/assets/eab55177-730b-4014-a13f-9778dc267ecd" />

3. We can also filter using specific categories, using the value 4625 that we saw earlier in the security event logs, we can see all the failed attempts to access the virtual machine
<img width="1000" alt="25" src="https://github.com/user-attachments/assets/44fc7f73-d828-4a04-bd54-a392c60d0f38" />

---

## Uploading our Geolocation Data to the SIEM

1. We are going to upload a geolocation file that allows us to filter the query in KQL based on other aspects such as location, latitude, and longitude
<img width="1000" alt="27" src="https://github.com/user-attachments/assets/beb58215-3fa5-4a84-a928-9166034db842" />

2. After downloading the file we can head to the Microsoft Sentinel page
     - Configuration tab -> Watchlist and create a new watchlist
     - Name: geoip
     - Alias: geoip
     - Source: Browse for the geolocation file
     - SearchKey: Network
     - Review and create
<img width="1000" alt="28" src="https://github.com/user-attachments/assets/11eb7c0c-a164-486e-8bfb-592da78abdab" />

---

## Inspecting our Enriched Logs

1. Now that everything has been setup, lets head back to our Log Analytics Workspace and review our logs
     - We have recieved roughly 30,000 logs!
2. To import our watchlist, we enter _GetWatchlist("geoip")
     - Now the categories shown are from the geoip file
<img width="1000" alt="29" src="https://github.com/user-attachments/assets/3ae73aa3-1155-4552-b61d-5ed9b619c952" />

3. To further filter our query, we are going to use KQL
<img width="1000" alt="30" src="https://github.com/user-attachments/assets/cb95033a-35e6-4123-a4e1-992f76d68943" />

4. GeoIP Lookup retrieves a watchlist of GeoIP data (containing IP-to-location mapping) and assigns it to the variable GEOIPDB_FULL
     - let WindowsEvents = SecurityEvent references the SecurityEvent table, which contains logs related to security events (such as login attempts, authentication failures, etc.)
     - | where IpAddress = <attacker IP address> filters the logs to include only those records where the IpAddress fields matches a specific attackers IP address
     - | where EventID == 4625 filters further to include only records with the EventID 4625, which represents failed login attempts in Windows Event Logs
     - | order by TimeGenerated desc orders the resulting records in descending order based on the timestamp (TimeGenerated) to display the most recent events first
     - | evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network); uses the ipv4_lookup function to perform a lookup of the IP address from the GeoIPDB_FULL watchlist to encirch the event with geographical data (such as location, country, city, etc.) related to the IP address
     - Windows Events returns the filtered and enriched events after applying the above transformations, showing the failed login attempts along with additional geographical information about the attackers IP address

<img width="1000" alt="32" src="https://github.com/user-attachments/assets/a2f6eea9-596f-4d53-8045-4ce28903e5f4" />

  Replacing IpAddress value with "185.156.73.169" we receive only logs from that specific IPaddress

<img width="1000" alt="33" src="https://github.com/user-attachments/assets/e1261386-b7b4-47e9-9d5c-2aeedfcebae9" />

5. Now to use Sentinel Workbooks to create a heatmap of all the attackers and where they are from
     - Go to Sentinel Instance
     - Workbooks -> add workbook
     - Edit and remove so we have a clear page
     - We are going to insert this JSON file into the workbook query we just created
<img width="1000" alt="34" src="https://github.com/user-attachments/assets/6aa8c4c3-d3d6-4886-966d-232f31d5c857" />
<img width="1000" alt="36" src="https://github.com/user-attachments/assets/422e2f09-0ebc-46f4-affa-24e9b0bb16e2" />
