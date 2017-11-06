# Overview
To release into an Azure Service Environment that has an Internal load balancer you need to create a VM in your VNET, install a release agent and deploy the code from there.  VSTS is used for the build and release process, just part of the release process runs on a VM that is inside a particular VNET.
This will work for any deployment inside a VNET, this just demonstrates a website.

# Process flow
- Build definition runs
- Build definition publishes the website as a server artifact
- Release process runs
- Custom agent
- Deploys the Website into the ASE by downloading the artifacts from TFS
 
# VM
- Created a VM in the VNET (this machine does not need to be that large or fast: A1 Standard or A2/A3 – you can always resize)
- Enter a Windows Server VM (you can do Linux, these instructions are for Windows)
- A market place image: Visual Studio 2015 Community Edition with Azure SDK / Update 3
- Turned off IE Enhanced Security
![alt tag](https://raw.githubusercontent.com/AdamPaternostro/Azure-Deploy-VSTS-On-VNET-App-Service-Environment/master/TurnOffIE.jpg)

- Added the role .NET 3.5

![alt tag](https://raw.githubusercontent.com/AdamPaternostro/Azure-Deploy-VSTS-On-VNET-App-Service-Environment/master/TurnOnNET35.jpg)

![alt tag](https://raw.githubusercontent.com/AdamPaternostro/Azure-Deploy-VSTS-On-VNET-App-Service-Environment/master/TurnOnAppServer.jpg)

- Installed Azure PowerShell SDK: https://azure.microsoft.com/en-us/downloads/  (not needed if part of VM)
- Set-Execution Policy to Unrestricted in PowerShell
- Update the hosts file (this is not needed if you have proper DNS).  You need both the site and the “scm” site.  Tjhe "scm" site is the Azure Web App management site.

Put the IP address Internal Load Balancer of the ASE in here with your domain of your ASE.

C:\Windows\System32\drivers\etc\hosts (open Notepad in Admin mode)
![alt tag](https://raw.githubusercontent.com/AdamPaternostro/Azure-Deploy-VSTS-On-VNET-App-Service-Environment/master/DNSEntries.jpg)

 
Also, you need to import the self-signed SSL into MMC.  Start MMC, add the certificates snap in, select computer account, right click on Trusted Root Certificate Authorities and import the PFX with your password.
- Install Visual Studio  (not needed if part of VM)
- Reboot

# VSTS
- Create a release definition (NOTE: you might need to run the agent on the VM steps below to register) 
- Have a Agent which uses a new Agent pool.  The agent poll is called “Julia VNET” since it will be on the VNET that the ASE is on.
Click the Manage link to create new agent (2nd screen shot)
![alt tag](https://raw.githubusercontent.com/AdamPaternostro/Azure-Deploy-VSTS-On-VNET-App-Service-Environment/master/NewAgent.jpg)

![alt tag](https://raw.githubusercontent.com/AdamPaternostro/Azure-Deploy-VSTS-On-VNET-App-Service-Environment/master/RunOnAgent.png)
 
- Deploy the web app (note the C:\VSTSRelease is from the PowerShell script)
![alt tag](https://raw.githubusercontent.com/AdamPaternostro/Azure-Deploy-VSTS-On-VNET-App-Service-Environment/master/DeployAzureAppService.png)

NOTE: If you are using an self-signed SSL, you need this the "allowUntrusted" as an additional argument.
 
- Create a token: PAL Token: https://www.visualstudio.com/en-us/docs/setup-admin/team-services/use-personal-access-tokens-to-authenticate
 
# VM
- Login to the VM
- Login to VSTS
- Go the release definition and click the Manage agent link again
- Download the agent and follow the instructions to install.  You have the PAT token, use that for Authentication.
![alt tag](https://raw.githubusercontent.com/AdamPaternostro/Azure-Deploy-VSTS-On-VNET-App-Service-Environment/master/DownloadAgent.jpg)

![alt tag](https://raw.githubusercontent.com/AdamPaternostro/Azure-Deploy-VSTS-On-VNET-App-Service-Environment/master/ConfigureAgent.jpg)

- Run the agent interactively:  .\run.cmd for debugging purposes
 
# VSTS
- Go to the agent again and enter the capability
![alt tag](https://raw.githubusercontent.com/AdamPaternostro/Azure-Deploy-VSTS-On-VNET-App-Service-Environment/master/Capabilites.jpg)
 
If you look at your release definition it has demands (meaning certain items must be installed on the machine in order to be able to run the tasks you have selected).  Not all tasks have this, so this is informational.
![alt tag](https://raw.githubusercontent.com/AdamPaternostro/Azure-Deploy-VSTS-On-VNET-App-Service-Environment/master/RunOnAgentQueue.jpg)
 
You should be able to run your release definition.
![alt tag](https://raw.githubusercontent.com/AdamPaternostro/Azure-Deploy-VSTS-On-VNET-App-Service-Environment/master/PowerShellOutput.jpg)
 
On the VM you should now see:
![alt tag](https://raw.githubusercontent.com/AdamPaternostro/Azure-Deploy-VSTS-On-VNET-App-Service-Environment/master/Website.jpg)
 
