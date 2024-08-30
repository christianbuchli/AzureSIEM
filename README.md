# Creating a Microsoft Azure SIEM and Geolocation Map
To start this project, a virtual machine needs to be created with port 3389, rdp, set to open and the only firewall rule set to allow all inbound traffic, and a log analytics workspace needs to be created as well.

Add Sentinel to the log analytics workspace

![SentinelSetup1](https://github.com/user-attachments/assets/3521fc89-63ce-46cd-89df-bcbee251aa9a)

Setup data connector to connect to the virtual machine. To do so click content hub then search for azure monitor agent and select Windows Security Events.

![SentinelSetup2](https://github.com/user-attachments/assets/71747896-2e3e-4ac3-9606-dae454e070ce)

![SentinelSetup3](https://github.com/user-attachments/assets/0b57ec56-2620-4d3d-b084-36614ec149f9)

Back in Sentinel data connectors tab, refresh, select windows security events via AMA, and open connector page.

![SentinelSetup4](https://github.com/user-attachments/assets/56f27496-a591-4fb4-aa13-63b1a9e7aad2)

Create a new data collection rule. In the resources tab select the azure subscription and expand the drop-down menus to select the resource group and virtual machine.

![SentinelSetup5](https://github.com/user-attachments/assets/d51f1a49-18de-417f-bde3-48534e39ca2b)

![SentinelSetup6](https://github.com/user-attachments/assets/7d59dc0c-c799-4919-ad56-25477cace270)

Once the rule is created, back in the sentinel dashboard, it’s time to create log rules. The rule is SecurityEvents where Activity contains “success” and Account !contains “system”. This rule will create an alert whenever there is a successful login that’s not the system. Next is to create a new alert rule. The general setup is setting “MITRE ATT&CK” to Initial Access, as that’s what an unauthorized sign-in falls under. In rule logic the query will be scheduled to run every 5 minutes and lookup data from the last to every 5 minutes as well and select “Group all events into a single alert”. In incident settings “create incidents from alerts triggered by this analytics rule” is selected. Once created the new rule is found in the analytics tab.

![SentinelSetup7](https://github.com/user-attachments/assets/c4c725ad-576b-49af-9c1c-e1eae8a0e20f)

![SentinelLogRule](https://github.com/user-attachments/assets/3b44de6f-bb48-46c2-9944-a676b255bb8e)

![SentinelLogRule2](https://github.com/user-attachments/assets/b93c381f-afd8-41c4-ba9f-59adc9464285)

![SentinelLogRule3](https://github.com/user-attachments/assets/a2bbc0a1-ab18-4e21-a3be-19324597b327)

![SentinelLogRule4](https://github.com/user-attachments/assets/b243e5f6-3e57-42ca-bf6c-2fc7e39c7ae3)

![SentinelLogRule5](https://github.com/user-attachments/assets/b183ac7f-fa1b-4d17-a124-c1db061ba581)

Now to test the alert. Signing in from my local pc to the virtual machine via rdp should trigger an alert and with that the Sentinel SIEM is now setup

![SentinelLogRule6](https://github.com/user-attachments/assets/538bf24f-16fb-4ba4-a12e-69482665ecba)

![SentinelLogRule7](https://github.com/user-attachments/assets/aea29027-0d1d-434e-b746-8e3b6497a7fa)

What the dashboard looks like after the whole lab was completed:

![PostLab](https://github.com/user-attachments/assets/927afb66-e01a-4e33-9cd6-356c72366400)

Now to map out geolocation from failed rdp logins based on IP addresses pulled from the event viewer logs. To do this, I will first connect to the virtual machine via rdp and access the event viewer logs. The logs are found in the security tab of the Window Logs folder and from my local pc I will ping the virtual machine from the command line to generate audit failures. Next is to open the firewall and disable the firewall for the domain, private, and public profiles. 

![geomap1](https://github.com/user-attachments/assets/117c49b4-b834-433e-8ad3-22a4630614a7)

Then use ipgeolocation.io to get an api key for mapping IP addresses to locations.

![geomap2](https://github.com/user-attachments/assets/3b74d03a-8cc6-408f-ae66-9e9e9d570459)

Using a powershell script from: https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1 the process of extracting IP addresses and geolocation data can be automated.

![geomap3](https://github.com/user-attachments/assets/ceeb1861-d984-4805-bb02-55ae2a597176)

In powershell ISE the script can be input and the api key from ipgeolocation.io inserted. After running the powershell script a log file will be created in “C:\ProgramData\failed_rdp.log”.

![geomap4](https://github.com/user-attachments/assets/42840f18-77a3-4494-98ae-27218f32ba6e)

Now back in the log analytics workspace, in the Tables tab, I can create a new custom table(MMA-based) and select a copy of the failed_rdp.log as the sample log

![geomap5](https://github.com/user-attachments/assets/0357d451-d67d-46e5-911b-aee412659324)

Record delimiter stays as New Line. In Collection Path, the path needs to be set to “C:\ProgramData\failed_rdp.log”. In details just set a log name. Then review + create.

![geomap6](https://github.com/user-attachments/assets/c8fdb7d7-7c4b-4367-8424-7a06cc9fdb44)

![geomap7](https://github.com/user-attachments/assets/421fa618-88ab-4fc2-8433-a6ef3b7dcc28)

![geomap8](https://github.com/user-attachments/assets/9d4537cb-752d-4d8a-b757-a8e93e9d19ac)

Next is to create an Azure Monitor Workspace and fill in the fields to link it to the resource group.

![MonitorWorkspace](https://github.com/user-attachments/assets/2b913f48-c5a1-45ae-b665-8c86be316816)

![MonitorWorkspace2](https://github.com/user-attachments/assets/2a4a7797-f41e-48d3-aeac-5f99d6fccd44)

Then in Data Collection Rules, a new rule needs to be created. Link it to the resource group and the created endpoint. Set the scope to the resources from the lab. Set the collection path to “C:\programdata\failed_rdp.log” and the delivery path to the Azure Monitor Logs.

![LogFix](https://github.com/user-attachments/assets/502e389e-0ff5-4fc0-9f3d-3e94ed5a4634)

![LogFix2](https://github.com/user-attachments/assets/98105465-090a-47e3-b0b2-939bb5f486ae)

![LogFix3](https://github.com/user-attachments/assets/3f432b20-5b9b-484e-ab29-b1cc57951e51)

With all that done, it’s time to create a Sentinel workbook to display the login data on a map. First is to create a new workbook and remove all of the default contents. Then add a new query and set the visualization to Map, and enter the query from: https://github.com/AnastasiaCoskuner/Sentinel-Lab/blob/main/query_log 
In the map settings, change the Metric Label to country and change any of the other size and color settings as needed to display a custom map.

![SentinelWorkbook](https://github.com/user-attachments/assets/5841f4ea-402b-4803-9ba2-4c39591f4c17)

![SentinelWorkbook2](https://github.com/user-attachments/assets/7260f103-84ca-437a-922e-7b0f9883f5d9)

![SentinelWorkbook3](https://github.com/user-attachments/assets/0205d829-399a-4f75-b908-bc2559bd62c4)

The final product:

![SentinelWorkbook4](https://github.com/user-attachments/assets/a803efb0-5fc7-4880-ba8f-0228aa228d11)

That’s all for this project, thanks for reading!

