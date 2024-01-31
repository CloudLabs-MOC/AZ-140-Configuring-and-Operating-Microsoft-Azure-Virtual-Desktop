# Module 05 - Deploy and manage host pools and hosts by using PowerShell

## Lab scenario

You need to automate the deployment of Azure Virtual Desktop host pools and hosts by using PowerShell in an Active Directory Domain Services (AD DS) environment.

## Lab Objectives
  
After completing this lab, you will be able to:

- Deploy Azure Virtual Desktop host pools and hosts by using PowerShell
- Add hosts to the Azure Virtual Desktop host pool by using PowerShell


## Estimated Timing: 60 minutes

## Architecture Diagram
  
  ![](./images/az-140-mod05.png)

## Exercise 1: Prerequisite - Setup Azure AD Connect

1. In the Azure portal, search for and select **Virtual machines** and, from the **Virtual machines** blade, select **az140-dc-vm11**.
   
3. On the **az140-dc-vm11** blade, select **Connect**.

4. On the **az140-dc-vm11| Connect** blade, select **Go to Bastion**.

   ![](./images/11.png)

5. On the **Bastion** tab of the **az140-dc-vm11**, provide the following credentials for the **Connection Settings** and select **Connect (4)**:

   |Setting|Value|
   |---|---|
   |Authentication Type|**VM Password (1)**|
   |Username|**Student (2)**|   
   |Password|**Pa55w.rd1234 (3)**|

    ![](./images/Az-140-Lab04-05-connectionsettings.png) 

   >**Note**: On clicking **Connect**, if you encounter an error **A popup blocker is preventing new window from opening. Please allow popups and retry**, then select the popup blocker icon at the top, select **Always allow pop-ups and redirects from https://portal.azure.com** and click on **Done**, and try connecting to the VM again.
  
   >**Note**: If you are prompted **See text and images copied to the clipboard**, select **Allow**. 

4. Once logged in, a logon task will start executing. When prompted **Do you want PowerShell to install and import the Nuget provider now?** enter **Y** and hit enter.
   
   >**Note**: Wait for the logon task to complete and present you with **Microsoft Azure Active Directory Connect** wizard. This should take about 10 minutes. If the **Microsoft Azure Active Directory Connect** wizard is not presented to you after the logon task completes, then launch it manually by double clicking the **Azure AD Connect** icon on the desktop.

5. On the **Welcome to Azure AD Connect** page of the **Microsoft Azure Active Directory Connect** wizard, select the checkbox **I agree to the license terms and privacy notice** and select **Continue**.

    ![](./images/31.png)
 
6. On the **Express Settings** page of the **Microsoft Azure Active Directory Connect** wizard, select the **Customize** option.

   ![](./images/18.png)
 
7. On the **Install required components** page, leave all optional configuration options deselected and select **Install**.

    ![](./images/28.png)
 
8. On the **User sign-in** page, ensure that only the **Password Hash Synchronization(1)** is selected and click on **Next(2)**.

    ![](./images/20.png)
 
9. On the **Connect to Azure AD** page, authenticate by using the credentials of the **aadsyncuser** user account and select **Next**. 

   ![](./images/08.png)

    >**Note**: Provide the userPrincipalName attribute of the **aadsyncuser** account available in the **LabValues** text file present on desktop and specify the password **Pa55w.rd1234**.

    ![](./images/19.png)

10. On the **Connect your directories** page, select the **Add Directory** button to the right of the **adatum.com** forest entry.

11. In the **AD forest account** window, ensure that the option to **Create new AD account** is selected, specify the following credentials, and select **OK**:

     |Setting|Value|
     |---|---|
     |User Name|**ADATUM\Student**|
     |Password|**Pa55w.rd1234**|

12. Back on the **Connect your directories** page, ensure that the **adatum.com** entry appears as a configured directory and select **Next**
  
13. On the **Azure AD sign-in configuration** page, note the warning stating **Users will not be able to sign-in to Azure AD with on-premises credentials if the UPN suffix does not match a verified domain name**, enable the checkbox **Continue without matching all UPN suffixes to verified domain**, and select **Next**.

    >**Note**: This is expected, since the Microsoft Entra ID tenant does not have a verified custom DNS domain matching one of the UPN suffixes of the **adatum.com** AD DS.

14. On the **Domain and OU filtering** page, select the option **Sync selected domains and OUs**, expand the adatum.com node, clear all checkboxes, select only the checkbox next to the **ToSync** OU, and select **Next**.

    ![](./images/07.png)
 
15. On the **Uniquely identifying your users** page, accept the default settings, and select **Next**.
   
16. On the **Filter users and devices** page, accept the default settings, and select **Next**.
    
17. On the **Optional features** page, accept the default settings, and select **Next**.
    
18. On the **Ready to configure** page, ensure that the **Start the synchronization process when configuration completes** checkbox is selected and select **Install**.

    ![](./images/25.png)

    >**Note**: Installation should take about 2 minutes.

19. Review the information on the **Configuration complete** page and select **Exit** to close the **Microsoft Azure Active Directory Connect** window.
 
    ![](./images/06.png)

20. Within the Bastion session to **az140-dc-vm11**, open Microsoft Edge browser shortcut for Azure or navigate to the [Azure portal](https://portal.azure.com). If prompted, sign in by using following Microsoft Entra ID credentials of the user account with the Owner role in the subscription you are using in this lab.

     * Email/Username: <inject key="AzureAdUserEmail"></inject>

     * Password: <inject key="AzureAdUserPassword"></inject>
     
21. In the Azure portal, use the **Search resources, services, and docs** text box at the top of the Azure portal page, search for and navigate to the **Microsoft Entra ID** blade and, on your Microsoft Entra ID tenant blade, in the **Manage** section of the hub menu, select **Users**.
    
22. On the **All users** blade, note that the list of user objects includes the listing of AD DS user accounts you created earlier in this lab, with the **Yes** entry appearing in the **On-premises sync enabled** column.

    >**Note**: You might have to wait a few minutes and refresh the browser page for the AD DS user accounts to appear. Proceed to next step only if you are able to see the listing of AD DS user accounts you created. 

23. Within the Bastion session to **az140-dc-vm11**, start **Windows PowerShell ISE** as administrator, and run the following to create an organizational unit that will host the computer objects of the Azure Virtual Desktop hosts:

    ```powershell
    New-ADOrganizationalUnit 'WVDInfra' –path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
    ```
24. Once the users are reflecting in the Microsoft Entra ID, right click on the **lab-prerequisite** PowerShell file present on the desktop and select **Run with PowerShell** in the popup options. This will configure the storage account with the naming convention `storage<DeploymentID>` and file share with the name `az140-22-profiles`.
   
    ![](./images/22.png)

    >**Note**: The script execution will take about 5 minutes. Once completed, the PowerShell window will display the text `Lab Pre-requisite Task Completed Successfully` in green color and the Powershell window will automatically close after a few seconds.


## Exercise 2: Implement Azure Virtual Desktop host pools and session hosts by using PowerShell
  
The main tasks for this exercise are as follows:

1. Prepare for deployment of Azure Virtual Desktop host pool by using PowerShell
1. Create a Azure Virtual Desktop host pool by using PowerShell
1. Perform a template-based deployment of an Azure VM running Windows 10 Enterprise by using PowerShell
1. Add an Azure VM running Windows 10 Enterprise as a session host to the Azure Virtual Desktop host pool by using PowerShell
1. Verify the deployment of the Azure Virtual Desktop session host

#### Task 1: Prepare for deployment of Azure Virtual Desktop host pool by using PowerShell

1. Within the Bastion session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to identify the distinguished name of the organizational unit named **WVDInfra** that will host the computer objects of the Azure Virtual Desktop pool session hosts:

   ```powershell
   (Get-ADOrganizationalUnit -Filter "Name -eq 'WVDInfra'").distinguishedName
   ```

1. Within the Bastion session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to identify the UPN suffix of the **ADATUM\\Student** account that you will use to join the Azure Virtual Desktop hosts to the AD DS domain (**student@adatum.com**):

   ```powershell
   (Get-ADUser -Filter {sAMAccountName -eq 'student'} -Properties userPrincipalName).userPrincipalName
   ```

1. Within the Bastion session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to install the DesktopVirtualization PowerShell module (when prompted, click **Yes to All**):

   ```powershell
   Install-Module -Name Az.DesktopVirtualization -Force
   ```

   >**Note**: Ignore any warnings regarding existing PowerShell modules in use.

1. Within the Bastion session to **az140-dc-vm11**, start Microsoft Edge and navigate to the [Azure portal](https://portal.azure.com). If prompted, sign in by using the Microsoft Entra ID credentials of the user account with the Owner role in the subscription you are using in this lab.

     * Email/Username: <inject key="AzureAdUserEmail"></inject>

     * Password: <inject key="AzureAdUserPassword"></inject>
   
1. Within the Bastion session to **az140-dc-vm11**, in the Azure portal, use the **Search resources, services, and docs** text box at the top of the Azure portal page to search for and navigate to **Virtual networks** and, on the **Virtual networks** blade, select **az140-adds-vnet11**.
    
1. On the **az140-adds-vnet11** blade, under **Settings** section, select **Subnets**, on the **Subnets** blade, select **+ Subnet**, on the **Add subnet** blade, specify the following settings (leave all other settings with their default values) and click **Save**:

   |Setting|Value|
   |---|---|
   |Name|**hp3-Subnet**|
   |Subnet address range|**10.0.3.0/24**|

1. Within the Bastion session to **az140-dc-vm11**, in the Azure portal, use the **Search resources, services, and docs** text box at the top of the Azure portal page to search for and navigate to **Network security groups** and, on the **Network security groups** blade, select the security group in the **az140-11-RG** resource group.
   
1. On the network security group blade, in the vertical menu on the left, in the **Settings** section, click **Properties**.
   
1. On the **Properties** blade, click the **Copy to clipboard** icon on the right side of the **Resource ID** textbox. 

   >**Note**: The value should resemble the format `/subscriptions/de8279a3-0675-40e6-91e2-5c3728792cb5/resourceGroups/az140-11-RG/providers/Microsoft.Network/networkSecurityGroups/az140-cl-vm11-nsg`, although the subscription ID will differ. Record it since you will need it in the next task.

#### Task 2: Create an Azure Virtual Desktop host pool by using PowerShell

1. Within the Bastion session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to sign in to your Azure subscription:

   ```powershell
   Connect-AzAccount
   ```

1. When prompted, provide the credentials of the user account with the Owner role in the subscription you are using in this lab.

     * Email/Username: <inject key="AzureAdUserEmail"></inject>

     * Password: <inject key="AzureAdUserPassword"></inject>
   
1. Within the Bastion session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to identify the Azure region hosting the Azure virtual network **az140-adds-vnet11**:

   ```powershell
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   ```

1. Within the Bastion session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to create a resource group that will host the host pool and its resources:

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. Within the Bastion session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to create an empty host pool:

   ```powershell
   $hostPoolName = 'az140-24-hp3'
   $workspaceName = 'az140-24-ws1'
   $dagAppGroupName = "$hostPoolName-DAG"
   New-AzWvdHostPool -ResourceGroupName $resourceGroupName -Name $hostPoolName -WorkspaceName $workspaceName -HostPoolType Pooled -LoadBalancerType BreadthFirst -Location $location -DesktopAppGroupName $dagAppGroupName -PreferredAppGroupType Desktop 
   ```

   >**Note**: The **New-AzWvdHostPool** cmdlet allows you to create a host pool, workspace, and the desktop app group, as well as to register the desktop app group with the workspace. You have the option of creating a new workspace or using an existing one.

1. Within the Bastion session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to retrieve the objectID attribute of the Microsoft Entra ID group named **az140-wvd-pooled**:

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-pooled').Id
   ```

1. Within the Bastion session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to assign the Microsoft Entra ID group named **az140-wvd-pooled** to the default desktop app group of the newly created host pool:

   ```powershell
   $roleDefinitionName = 'Desktop Virtualization User'
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName $roleDefinitionName -ResourceName $dagAppGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

#### Task 3: Perform a template-based deployment of an Azure VM running Windows 10 Enterprise by using PowerShell

1. From your lab computer, use the **Search resources, services, and docs** text box at the top of the Azure portal page to search for and navigate to **Storage accounts**, and select the storage account **storage<inject key="DeploymentID" enableCopy="false" />**. On the storage account blade, in the **Data storage** section, select **File shares** and then select the **az140-22-profiles** file share.

1. Select **Upload** and **browse for files**, navigate to the path **C:\AllFiles\AZ-140-Configuring-and-Operating-Microsoft-Azure-Virtual-Desktop\Allfiles\Labs\02** and upload the lab files **az140-24_azuredeployhp3.json** and **az140-24_azuredeployhp3.parameters.json** to the file share.

1. Within the Bastion session to **az140-dc-vm11**, open File Explorer and navigate to the previously configured **Z** drive:, or the drive letter assigned to the connection to the File Share. Copy the uploaded deployment files to **C:\AllFiles\Labs\02**.

    ![](./images/21.png)

1. Within the Bastion session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to deploy an Azure VM running Windows 10 Enterprise (multi-session) that will serve as a Azure Virtual Desktop session host in the host pool you created in the previous task:

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab24hp3Deployment `
     -TemplateFile C:\AllFiles\Labs\02\az140-24_azuredeployhp3.json `
     -TemplateParameterFile C:\AllFiles\Labs\02\az140-24_azuredeployhp3.parameters.json
   ```

   >**Note**: Wait for the deployment to complete before you proceed to the next task. This might take about 5 minutes. 

   >**Note**: The deployment uses an Azure Resource Manager template to provision an Azure VM and applies a VM extension that automatically joins the operating system to the **adatum.com** AD DS domain.

1. Within the Bastion session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to verify that the session host was successfully joined to the **adatum.com** AD DS domain:

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-24-p3-0$'"
   ```

#### Task 4: Add an Azure VM running Windows 10 Enterprise as a host to the Azure Virtual Desktop host pool by using PowerShell

1. Within the Bastion session to **az140-dc-vm11**, in the browser window displaying the Azure portal, search for and select **Virtual machines** and, on the **Virtual machines** blade, in the list of virtual machines, select **az140-24-p3-0**.
   
2. On the **az140-24-p3-0** blade, select **Connect**, and select **RDP**, on the **RDP** tab of the **az140-24-p3-0 | Connect** blade, Copy the **Private IP address (10.0.3.4)** under **Connecting using private IP address**, and then select **Native RDP** then under step 3 select **Download RDP File**, open the downloded file and click on **Connect**.
   
3. When prompted, sign in with the following credentials and click on **OK**.

   |Setting|Value|
   |---|---|
   |User Name|**Student**|
   |Password|**Pa55w.rd1234**|

   >**Note**: If you get **Welcome to Microsoft Teams: Get started** page, then close the application.

4. Within the Remote Desktop session to **az140-24-p3-0**, start **Windows PowerShell ISE** as administrator.

5. Within the Remote Desktop session to **az140-24-p3-0**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to create a folder that will host files required to add the newly deployed Azure VM as a session host to the host pool you provisioned earlier in this lab:

   ```powershell
   $labFilesFolder = 'C:\AllFiles\Labs\02'
   New-Item -ItemType Directory -Path $labFilesFolder
   ```

   >**Note:** Take care using the [T] construct to copy over the PowerShell cmdlets. In some instances, the text copied over can be incorrect, such as the $ sign showing as a 4 number character. You will need to correct these before issuing the cmdlet. Copy over to the PowerShell ISE **Script** pane, make the corrections there, and then highlight the corrected text and press **F8** (**Run Selection**).

6. Within the Remote Desktop session to **az140-24-p3-0**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to download the Azure Virtual Desktop Agent and Boot Loader installers, required to add the session host to the host pool:

   ```powershell
   $webClient = New-Object System.Net.WebClient
   $wvdAgentInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrmXv'
   $wvdAgentInstallerName = 'WVD-Agent.msi'
   $webClient.DownloadFile($wvdAgentInstallerURL,"$labFilesFolder/$wvdAgentInstallerName")
   $wvdBootLoaderInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrxrH'
   $wvdBootLoaderInstallerName = 'WVD-BootLoader.msi'
   $webClient.DownloadFile($wvdBootLoaderInstallerURL,"$labFilesFolder/$wvdBootLoaderInstallerName")
   ```

7. Within the Remote Desktop session to **az140-24-p3-0**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to install the latest version of the PowerShellGet module (select **Yes** when prompted for confirmation):

   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope CurrentUser -Force
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

8. From the **Administrator: Windows PowerShell ISE** console, run the following to install the latest version of the Az.DesktopVirtualization PowerShell module:

   ```powershell
   Install-Module -Name Az -AllowClobber -Force
   Install-Module -Name Az.DesktopVirtualization -AllowClobber -Force
   ```

9. From the **Administrator: Windows PowerShell ISE** console, run the following to modify the PowerShell execution policy and sign in to your Azure subscription:

   ```powershell
   Connect-AzAccount
   ```

10. When prompted, provide the credentials of the user account with the Owner role in the subscription you are using in this lab.

     * Email/Username: <inject key="AzureAdUserEmail"></inject>

     * Password: <inject key="AzureAdUserPassword"></inject>
     
11. Within the Remote Desktopliveid session to **az140-24-p3-0**, from the **Administrator: Windows PowerShell ISE** console, run the following to generate the token necessary to join new session hosts to the pool you provisioned earlier in this exercise:

     ```powershell
     $resourceGroupName = 'az140-24-RG'
     $hostPoolName = 'az140-24-hp3'
     $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName $resourceGroupName -HostPoolName $hostPoolName -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
     $registrationInfo
     $registrationInfo.Token
     ```
     >**Note**: A registration token is required to authorize a session host to join the host pool. The value of token's expiration date must be between one hour and one month from the current date and time.

12. Within the Remote Desktop session to **az140-24-p3-0**, from the **Administrator: Windows PowerShell ISE** console, run the following to install the Azure Virtual Desktop Agent:

     ```powershell
     Set-Location -Path $labFilesFolder
     Start-Process -FilePath 'msiexec.exe' -ArgumentList "/i $WVDAgentInstallerName", "/quiet", "/qn", "/norestart", "/passive", "REGISTRATIONTOKEN=$($registrationInfo.Token)", "/l* $labFilesFolder\AgentInstall.log" | Wait-Process
     ```

13. Within the Remote Desktop session to **az140-24-p3-0**, from the **Administrator: Windows PowerShell ISE** console, run the following to install the Azure Virtual Desktop Boot Loader:

     ```powershell
     Start-Process -FilePath "msiexec.exe" -ArgumentList "/i $wvdBootLoaderInstallerName", "/quiet", "/qn", "/norestart", "/passive", "/l* $labFilesFolder\BootLoaderInstall.log" | Wait-process
     ``` 

#### Task 5: Verify the deployment of the Azure Virtual Desktop host

1. Switch to the lab computer, in the web browser displaying the Azure portal, search for and select **Azure Virtual Desktop**, on the **Azure Virtual Desktop** blade, under **Manage** section, select **Host pools** and, on the **Azure Virtual Desktop \| Host pools** blade, select the entry **az140-24-hp3** representing the newly modified pool.
   
1. On the **az140-24-hp3** blade, in the vertical menu on the left side, in the **Manage** section, click **Session hosts**. 
1. On the **az140-24-hp3 \| Session hosts** blade, verify that the deployment includes a single host.

   >**Note**: If the host is in unavailable state, wait for a few seconds and click on **refresh** in the above tool bar.

#### Task 6: Manage app groups using PowerShell

1. From the lab computer, switch to the Bastion session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to create a Remote App group:

   ```powershell
   $subscriptionId = (Get-AzContext).Subscription.Id
   $appGroupName = 'az140-24-hp3-Office365-RAG'
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   New-AzWvdApplicationGroup -Name $appGroupName -ResourceGroupName $resourceGroupName -ApplicationGroupType 'RemoteApp' -HostPoolArmPath "/subscriptions/$subscriptionId/resourcegroups/$resourceGroupName/providers/Microsoft.DesktopVirtualization/hostPools/$hostPoolName"-Location $location
   ```

1. Within the Bastion session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to list the **Start** menu apps on the pool's hosts and review the output:

   ```powershell
   Get-AzWvdStartMenuItem -ApplicationGroupName $appGroupName -ResourceGroupName $resourceGroupName | Format-List | more
   ```

   >**Note**: For any application you want to publish, you should record the information included in the output, including such parameters as **FilePath**, **IconPath**, and **IconIndex**.

1. Within the Bastion session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to publish Microsoft Word:

   ```powershell
   $name = 'Microsoft Word'
   $filePath = 'C:\Program Files\Microsoft Office\root\Office16\WINWORD.EXE'
   $iconPath = 'C:\Program Files\Microsoft Office\Root\VFS\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\wordicon.exe'
   New-AzWvdApplication -GroupName $appGroupName -Name $name -ResourceGroupName $resourceGroupName -FriendlyName $name -Filepath $filePath -IconPath $iconPath -IconIndex 0 -CommandLineSetting 'DoNotAllow' -ShowInPortal:$true
   ```

1. Within the Bastion session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to publish Microsoft Word:

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-remote-app').Id
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName 'Desktop Virtualization User' -ResourceName $appGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

1. Switch to the lab computer, in the web browser displaying the Azure portal, on the **az140-24-hp3 \| Session hosts** blade, in the vertical menu on the left side, in the **Manage** section, select **Application groups**.
   
1. On the **az140-24-hp3 \| Application groups** blade, in the list of application groups, select the **az140-24-hp3-Office365-RAG** entry.
   
1. On the **az140-24-hp3-Office365-RAG** blade, verify the configuration of the application group, including the applications and assignments.

    **Congratulations** on completing the task! Now, it's time to validate it. Here are the steps:
     > - Navigate to the Lab Validation Page, from the upper right corner in the lab guide section.
     > - Hit the Validate button for the corresponding task. If you receive a success message, you can proceed to the next task. 
     > - If not, carefully read the error message and retry the step, following the instructions in the lab guide.
     > - If you need any assistance, please contact us at labs-support@spektrasystems.com. 

### Review
In this lab, you have completed the following:
- Deployment of  Azure Virtual Desktop host pools and hosts by using PowerShell
- Adding hosts to the Azure Virtual Desktop host pool by using PowerShell

## You have successfully completed the lab

