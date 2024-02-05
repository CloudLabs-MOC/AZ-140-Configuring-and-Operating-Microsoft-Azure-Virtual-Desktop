# Module 04 - Deploy host pools and hosts by using Azure Resource Manager templates

## Lab scenario

You need to automate the deployment of Azure Virtual Desktop host pools and hosts by using Azure Resource Manager templates.

## Lab Objectives
  
After completing this lab, you will be able to deploy Azure Virtual Desktop host pools and hosts by using Azure Resource Manager templates

## Estimated Timing: 45 minutes

## Architecture Diagram
  
  ![](./images/az-140-mod04.png)



## Lab files

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuredeployhp23.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json

## Exercise 1: Prerequisite - Setup Azure AD Connect

1. In the Azure portal, search for and select **Virtual machines** and, from the **Virtual machines** blade, select **az140-dc-vm11**.

1. On the **az140-dc-vm11** blade, select **Connect**.

1. On the **az140-dc-vm11| Connect** blade,, select **Go to Bastion**.

   ![](./images/11.png)

1. On the **Bastion** tab of the **az140-dc-vm11**, provide the following credentials for the **Connection Settings** and select **Connect (4)**:

   |Setting|Value|
   |---|---|
   |Authentication Type|**VM Password (1)**| 
   |Username|**Student (2)**|
   |Password|**Pa55w.rd1234 (3)**|

    ![](./images/Az-140-Lab04-05-connectionsettings.png)

     **Note**: On clicking **Connect**, if you encounter an error: **A popup blocker is preventing new window from opening. Please allow popups and retry**, then select the popup blocker icon at the top, select **Always allow pop-ups and redirects from https://portal.azure.com** and click on **Done**, and try connecting to the VM again.

    ![](./images/16.png) 

     **Note**: If you are prompted **See text and images copied to the clipboard**, select **Allow**.

5. Once logged in, a logon task will start executing. When prompted **Do you want PowerShell to install and import the Nuget provider now?** enter **Y** and hit enter.
   
      > **Note**: Wait for the logon task to complete and present you with **Microsoft Azure Active Directory Connect** wizard. This should take about 10 minutes. If the **Microsoft Azure Active Directory Connect** wizard is not presented to you after the logon task completes, then launch it manually by double-clicking the **Azure AD Connect** icon on the desktop.


6. On the **Welcome to Azure AD Connect** page of the **Microsoft Azure Active Directory Connect** wizard, select the checkbox **I agree to the license terms and privacy notice** and select **Continue**.

    ![](./images/31.png)

7. On the **Express Settings** page of the **Microsoft Azure Active Directory Connect** wizard, select the **Customize** option.

   ![](./images/18.png)

8. On the **Install required components** page, leave all optional configuration options deselected and select **Install**.

   ![](./images/28.png)

9. On the **User sign-in** page, ensure that only the **Password Hash Synchronization (1)** is enabled and select **Next(2)**.

   ![](./images/20.png)

10. On the **Connect to Azure AD** page, authenticate by using the credentials of the **aadsyncuser** user account and select **Next**.

    ![](./images/08.png)

      > **Note**: Provide the userPrincipalName attribute of the **aadsyncuser** account available in the **LabValues** text file present on desktop and specify the password **Pa55w.rd1234**.

      ![](./images/19.png)

      > **Note**: If the sign-in pop-up comes, sign in by using the Azure AD credentials of the user account with the Owner role in the subscription you are using in this lab.

11. On the **Connect your directories** page, select the **Add Directory** button to the right of the **adatum.com** forest entry.

    ![](./images/17.png)

12. In the **AD forest account** window, ensure that the option to **Create new AD account** is selected, specify the following credentials, and select **OK**:

      |Setting|Value|
      |---|---|
      |User Name|**ADATUM\Student**|
      |Password|**Pa55w.rd1234**|

13. Back on the **Connect your directories** page, ensure that the **adatum.com** entry appears as a configured directory and select **Next**

14. On the **Azure AD sign-in configuration** page, note the warning stating **Users will not be able to sign in to Azure AD with on-premises credentials if the UPN suffix does not match a verified domain name**, enable the checkbox **Continue without matching all UPN suffixes to verified domain**, and select **Next**.

      > **Note**: This is expected since the Azure AD tenant does not have a verified custom DNS domain matching one of the UPN suffixes of the **adatum.com** AD DS.

15. On the **Domain and OU filtering** page, select the option **Sync selected domains and OUs**, expand the adatum.com node, clear all checkboxes, select only the checkbox next to the **ToSync** OU, and select **Next**.

    ![](./images/07.png)

16. On the **Uniquely identifying your users** page, accept the default settings, and select **Next**.

17. On the **Filter users and devices** page, accept the default settings, and select **Next**.

18. On the **Optional features** page, accept the default settings, and select **Next**.

19. On the **Ready to configure** page, ensure that the **Start the synchronization process when configuration completes** checkbox is selected and select **Install**.
    
    ![](./images/25.png)

      > **Note**: Installation should take about 2 minutes.

20. Review the information on the **Configuration complete** page and select **Exit** to close the **Microsoft Azure Active Directory Connect** window.

    ![](./images/06.png)

21. Within the Remote Desktop session to **az140-dc-vm11**, open **Azure portal** shortcut, and sign in by using the Azure AD credentials of the user account with the Owner role in the subscription you are using in this lab.

22. In the Azure portal, use the **Search resources, services, and docs** text box at the top of the Azure portal page, search for and navigate to the **Microsoft Entra ID** blade and, on your Azure AD tenant blade, in the **Overview** section of the hub menu, select **Users**.

23. On the **Users** blade, note that the list of user objects includes the listing of AD DS user accounts you created earlier in this lab, with the **Yes** entry appearing in the **On-premises sync enabled** column.

      > **Note**: You might have to wait a few minutes and refresh the browser page for the AD DS user accounts to appear. Proceed to the next step only if you are able to see the listing of AD DS user accounts you created. 


24. Within the Remote Desktop session to **az140-dc-vm11**, start **Windows PowerShell ISE** as administrator, and run the following to create an organizational unit that will host the computer objects of the Azure Virtual Desktop hosts:

     ```powershell
     New-ADOrganizationalUnit 'WVDInfra' –path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
     ```

## Exercise 2: Deploy Azure Virtual Desktop host pools and hosts by using Azure Resource Manager templates
  
The main tasks for this exercise are as follows:

1. Prepare for deployment of an Azure Virtual Desktop host pool by using an Azure Resource Manager template

1. Deploy an Azure Virtual Desktop host pool and hosts by using an Azure Resource Manager template

1. Verify deployment of the Azure Virtual Desktop host pool and hosts

1. Prepare for adding hosts to the existing Azure Virtual Desktop host pool by using an Azure Resource Manager template

1. Add hosts to the existing Azure Virtual Desktop host pool by using an Azure Resource Manager template

1. Verify changes to the Azure Virtual Desktop host pool

1. Manage personal desktop assignments in the Azure Virtual Desktop host pool

### Task 1: Prepare for deployment of an Azure Virtual Desktop host pool by using an Azure Resource Manager template

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to identify the distinguished name of the organizational unit named **WVDInfra** that will host the computer objects of the Azure Virtual Desktop pool hosts:

   ```powershell
   (Get-ADOrganizationalUnit -Filter "Name -eq 'WVDInfra'").distinguishedName
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to identify the user principal name attribute of the **ADATUM\\Student** account that you will use to join the Azure Virtual Desktop hosts to the AD DS domain (**student@adatum.com**):

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'student'").userPrincipalName
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to identify the user principal name of the **ADATUM\\aduser7** and **ADATUM\\aduser8** accounts that you will use to test personal desktop assignments later in this lab:

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser7'").userPrincipalName
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser8'").userPrincipalName
   ```

   > **Note**: Record all user principal name values you identified. You will need them later in this lab.

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to calculate the token expiration time necessary to perform a template-based deployment:

   ```powershell
   $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

   > **Note**: The value should resemble the format `2022-03-27T00:51:28.3008055Z`. Record it since you will need it in the next task.

   > **Note**: A registration token is required to authorize a host to join the pool. The value of the token's expiration date must be between one hour and one month from the current date and time.

1. Within the Remote Desktop session to **az140-dc-vm11**, navigate back to the **Azure Portal**.

1. Within the Remote Desktop session to **az140-dc-vm11**, in the Azure portal, use the **Search resources, services, and docs** text box at the top of the Azure portal page to search for and navigate to **Virtual networks** and, on the **Virtual networks** blade, select **az140-adds-vnet11**. 

1. On the **az140-adds-vnet11** blade, select **Subnets**, on the **Subnets** blade, select **+ Subnet**, on the **Add subnet** blade, specify the following settings (leave all other settings with their default values) and click **Save**:

   |Setting|Value|
   |---|---|
   |Name|**hp2-Subnet**|
   |Subnet address range|**10.0.2.0/24**|

1. Within the Remote Desktop session to **az140-dc-vm11**, in the Azure portal, use the **Search resources, services, and docs** text box at the top of the Azure portal page to search for and navigate to **Network security groups** and, on the **Network security groups** blade, select the network security group in the **az140-11-RG** resource group.
1. On the network security group blade, in the vertical menu on the left, in the **Settings** section, click **Properties**.
1. On the **Properties** blade, click the **Copy to clipboard** icon on the right side of the **Resource ID** textbox. 

   > **Note**: The value should resemble the format `/subscriptions/de8279a3-0675-40e6-91e2-5c3728792cb5/resourceGroups/az140-11-RG/providers/Microsoft.Network/networkSecurityGroups/az140-cl-vm11-nsg`, although the subscription ID will differ. Record it since you will need it in the next task.

### Task 2: Deploy an Azure Virtual Desktop host pool and hosts by using an Azure Resource Manager template

1. From your lab computer, start a web browser, navigate to the [Azure portal](https://portal.azure.com), and sign in by providing credentials of a user account with the Owner role in the subscription you will be using in this lab.

1. In the Azure portal, search for and select **Resource group**, Click on **+ Create** and enter the name of the resource group as **az140-23-RG** and select the **Region** in which the lab was deployed, then select **Review + Create** and select **Create**.

1. From your lab computer, in the same web browser window, open another web browser tab and navigate to the GitHub Azure RDS templates repository page [ARM Template to Create and provision new Windows Virtual Desktop hostpool](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/CreateAndProvisionHostPool). 


   > **Note**: Scroll little bit down then you'll be able to see the **ARM Template to Create and provision new Windows Virtual Desktop hostpool** page.

1. On the **ARM Template to Create and provision new Windows Virtual Desktop hostpool** page, select **Deploy to Azure**. This will automatically redirect the browser to the **Custom deployment** blade in the Azure portal.

   > **Note**: If you get **Action Required** page, then select **Ask later**.

1. On the **Custom deployment** blade, select **Edit parameters**.

1. On the **Edit parameters** blade, select **Load file**, in the **Open** dialog box, navigate to the path **C:\AllFiles\AZ-140-Configuring-and-Operating-Microsoft-Azure-Virtual-Desktop\Allfiles\Labs\02** and select the file **az140-23_azuredeployhp23.parameters.json**, select **Open**, and then select **Save**. 

1. Back on the **Custom deployment** blade, specify the following settings (leave others with their existing values):

   |Setting|Value|
   |---|---|
   |Subscription|the name of the Azure subscription you are using in this lab|
   |Resource Group|**az140-23-RG**|
   |Region|the name of the Azure region into which you deployed Azure VMs hosting AD DS domain controllers in the lab **Prepare for deployment of Azure Virtual Desktop (AD DS)**|
   |Location|the name of the same Azure region as the one set as the value of the **Region** parameters|
   |Workspace location|the name of the same Azure region as the one set as the value of the **Region** parameters|
   |Workspace Resource Group|Leave as default|
   |All Application Group Reference|leave as default|
   |Vm location|the name of the same Azure region as the one set as the value of the **Location** parameters|
   |Create Network Security Group|**false**|
   |Network Security Group Id|the value of the resourceID parameter of the existing network security group you identified in the previous task|
   |Token Expiration Time| the value of the token expiration time you calculated in the previous task|

   > **Note**: The deployment provisions a pool with personal desktop assignment type.

1. On the **Custom deployment** blade, select **Review + create** and select **Create**.

   > **Note**: Wait for the deployment to complete before you proceed to the next task. This might take about 15 minutes. 

### Task 3: Verify deployment of the Azure Virtual Desktop host pool and hosts

1. From your lab computer, in the web browser displaying the Azure portal, search for and select **Azure Virtual Desktop**, on the **Azure Virtual Desktop** blade, select **Host pools** and, on the **Azure Virtual Desktop \| Host pools** blade, select the entry **az140-23-hp2** representing the newly deployed pool.

1. On the **az140-23-hp2** blade, in the vertical menu on the left side, in the **Manage** section, click **Session hosts**. 

1. On the **az140-23-hp2 \| Session hosts** blade, verify that the deployment consists of two hosts.

1. On the **az140-23-hp2 \| Session hosts** blade, in the vertical menu on the left side, in the **Manage** section, click **Application groups**.

1. On the **az140-23-hp2 \| Application groups** blade, verify that the deployment includes the **Default Desktop** application group named **az140-23-hp2-DAG**.

### Task 4: Manage personal desktop assignments in the Azure Virtual Desktop host pool

1. On your lab computer, in the web browser displaying the Azure portal, search for and select **Azure Virtual Desktop**. On the **az140-23-hp2** blade, in the vertical menu on the left side, in the **Manage** section, select **Host Pools**. Then, select the host pool entry **az140-23-hp2**, and select **Application groups** in the vertical menu on the left side, under the **Manage** section. 

1. On the **az140-23-hp2 \| Application groups** blade, select the application group **az140-23-hp2-DAG**.

1. On the **az140-23-hp2-DAG** blade, in the vertical menu on the left, select **Assignments**. 

1. On the **az140-23-hp2-DAG \| Assignments** blade, select **+ Add**.

1. On the **Select Azure AD users or user groups** blade, select **az140-wvd-personal** and click **Select**.

   ![](./images/15.png)


   > **Note**: Now let's review the experience of a user connecting to the Azure Virtual Desktop host pool.

1. From your lab computer, in the browser window displaying the Azure portal, search for and select **Virtual machines** and, on the **Virtual machines** blade, select the **az140-cl-vm11** entry.

1. On the **az140-cl-vm11** blade, select **Connect**, select **Bastion**, click on **Use Bastion**.

1. When prompted, provide the following credentials and select **Connect**:

   |Setting|Value|
   |---|---|
   |User Name|**Student@adatum.com**|
   |Password|**Pa55w.rd1234**|


   > **Note**: On clicking **Connect**, if you encounter an error: **A popup blocker is preventing new window from opening. Please allow popups and retry**, then select the popup blocker icon at the top, select **Always allow pop-ups and redirects from https://portal.azure.com** click on **Done**, and try connecting to the VM again.
  
   > **Note**: If you are prompted **See text and images copied to the clipboard**, select **Allow**. 
  
   > **Note**: If the VM stays in the loading state on the Welcome page for more than 2 minutes, then close the VM bastion tab, restart the VM by navigating to the **Overview** blade in the Virtual Machine vertical menu on the left side, and try logging in again by providing the credentials.

9. Within the Remote Desktop session to **az140-cl-vm11**, start Microsoft Edge and navigate to [Windows Desktop client download page](https://go.microsoft.com/fwlink/?linkid=2068602) which will download the Remote Desktop client program. Once downloaded, open the file to start its installation. In the **Welcome** page select **Next**. If prompted, accept the agreement and select **Next**, and on the **Installation Scope** page of the **Remote Desktop Setup** wizard, select the option **Install for all users of this machine** and click **Install**. If prompted by User Account Control for administrative credentials, authenticate by using the **ADATUM\\Student** username with **Pa55w.rd1234** as its password.

10. Once the installation completes, ensure that the **Launch Remote Desktop when setup exits** checkbox is selected and click **Finish** to start the Remote Desktop client.

11. Within the Remote Desktop session to **az140-cl-vm11**, in the Remote Desktop window, on the **Let's get started page**, click **Subscribe**.

12. when prompted, sign in with the **aduser7** credentials, by providing its userPrincipalName and **Pa55w.rd1234** as its password.

      > **Note**: Alternatively, in the **Remote Desktop** client window, select **Subscribe with URL**, in the **Subscribe to a Workspace** pane, in the **Email or Workspace URL**, type **https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery**, select **Next**, and, once prompted, sign in with the **aduser7** credentials (using its userPrincipalName attribute as the user name and the password you set when creating this account). 

13. On the **Remote Desktop** page, double-click the **SessionDesktop** icon, when prompted for credentials, type the same password again, select the **Remember me** checkbox, and click **OK**.

      > **Note**: If you get the **Stay signed in to all your apps** window, clear the checkbox **Allow my organization to manage my device** and select **No, sign in to this app only**. 

14. Verify that **aduser7** successfully signed in via Remote Desktop to a host.

15. Within the Remote Desktop session to one of the hosts as **aduser7**, right-click **Start**, in the right-click menu, select **Shut down or sign out**, and, in the cascading menu, click **Sign out**.

      > **Note**: Now let's switch the personal desktop assignment from the direct mode to automatic. 

16. Switch to your lab computer, to the web browser displaying the Azure portal, search for and select **Azure Virtual Desktop**, on the **Azure Virtual Desktop** blade, select **Application groups**, and select the application group entry **az140-23-hp2-DAG**. On the **az140-23-hp2-DAG** blade, in the vertical menu in the left side, select the **Assignments**. In the informational bar directly above the list of assignments, click the **Assign VM** link. This will redirect you to the **az140-23-hp2 \| Session hosts** blade. 

    ![](./images/13.png)

17. On the **az140-23-hp2 \| Session hosts** blade, verify that one of the hosts has **aduser7** listed in the **Assigned User** column.

      > **Note**: This is expected since the host pool is configured for automatic assignment.

18. On your lab computer, in the web browser window displaying the Azure portal, open the **PowerShell** shell session within the **Cloud Shell** pane. Then, select **Create storage** and wait for a few seconds for the Cloud Shell to provision.

    ![](./images/12.png)

19. From the PowerShell session in the Cloud Shell pane, run the following to switch to the direct assignment mode:

    ```powershell
    Update-AzWvdHostPool -ResourceGroupName 'az140-23-RG' -Name 'az140-23-hp2' -PersonalDesktopAssignmentType Direct
    ```

20. On your lab computer, in the web browser window displaying the Azure portal, navigate to the **az140-23-hp2** host pool blade, review the **Essentials** section and verify that the **Host pool type** is set to **Personal** with the **Assignment type** set to **Direct**.

21. Switch back to the Remote Desktop session to **az140-cl-vm11**, in the **Remote Desktop** window, click the second ellipsis icon in the upper right corner, in the dropdown menu, click **Unsubscribe**, and, when prompted for confirmation, click **Continue**.

      ![](./images/AZ-140-module-4-ellipses.png)

22. Within the Remote Desktop session to **az140-cl-vm11**, in the **Remote Desktop** window, on the **Let's get started** page, click **Subscribe**.

23. When prompted sign in with the, **aduser8** credentials, by providing its userPrincipalName and **Pa55w.rd1234** as its password.

      > **Note**: If you get **Action Required** page, then select **Ask later**.

      > **Note**: If you get the **Stay signed in to all your apps** window, clear the checkbox **Allow my organization to manage my device** checkbox and select **No, sign in to this app only**. 

24. On the **Remote Desktop** page, double-click the **SessionDesktop** icon, and verify that you receive an error message stating **We couldn't connect because there are currently no available resources. Try again later or contact tech support for help if this keeps happening**, and click **OK**.

      > **Note**: This is expected since the host pool is configured for direct assignment and **aduser8** has not been assigned a host.

25. Switch to your lab computer, to the web browser displaying the Azure portal and, on the **az140-23-hp2** blade, select **Session hosts** in the vertical menu in the left side, and select the **(Assign)** link in the **Assigned User** column next to one of the two remaining unassigned hosts.

26. On the **Assign a user**, select **aduser8**, click **Assign** and, when prompted for confirmation, click **OK**.

27. Switch back to the Remote Desktop session to **az140-cl-vm11**, in the **Remote Desktop** window, double-click the **SessionDesktop** icon, when prompted for the password, type the password **Pa55w.rd1234**, click **OK**, and verify that you can successfully sign in to the assigned host.

    > **Congratulations** on completing the lab! Now, it's time to validate it. Here are the steps:
    > - Navigate to the Lab Validation Page, from the upper right corner in the lab guide section.
    > - Hit the Validate button for the corresponding task. If you receive a success message, you can proceed to the next task. 
    > - If not, carefully read the error message and retry the step, following the instructions in the lab guide.
    > - If you need any assistance, please contact us at labs-support@spektrasystems.com. We are available 24/7 to help.

### Review
In this lab, you have completed the following:
- Deployed  an Active Directory Domain Services (AD DS) single-domain forest by using Azure VMs
- Integrated an AD DS forest with an Azure Active Directory (Azure AD) tenant

## You have successfully completed the lab
