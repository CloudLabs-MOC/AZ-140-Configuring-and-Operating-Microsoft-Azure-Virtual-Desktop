# Lab - Deploy host pools and session hosts by using the Azure portal (Entra ID)

## Estimated Duration: 60 Minutes

## Lab Objectives
  
In this lab, you will complete the following tasks:

- Prepare the Azure subscription for deployment of an Azure Virtual Desktop host pool
- Deploy an Azure Virtual Desktop host pool
- Create an Azure Virtual Desktop application group
- Create an Azure Virtual Desktop workspace
- Grant access to Azure Virtual Desktop host pools

## Exercise 1: Implement an Azure Virtual Desktop environment using Microsoft Entra joined session hosts
  
### Task 1: Prepare the Azure subscription for deployment of an Azure Virtual Desktop host pool

1. In the lab VM, click on the **Azure Portal icon** as shown below:

    ![](Media/lab1-11-1.png)
   
    - On the **Sign in to Microsoft Azure** tab, you will see the login screen. Enter your credentials:
      
        * **Email/Username:** <inject key="AzureAdUserEmail"></inject>
    
    - Next, provide your password:

        * **Temporary Access Pass:** <inject key="AzureAdUserPassword"></inject>

1. On the Azure portal homepage, click the **\[>\_] Cloud Shell (1)** button located to the right of the **Copilot** tab at the top. This opens a new Cloud Shell session. In the **Welcome to Azure Cloud Shell** window, choose **PowerShell (2)**.

    ![](Media/lab1-11-1.png)

1. In the **Getting started** window, ensure **No storage account required (1)** is selected. From the **Subscription** drop-down, choose **Default subscription (2)**, then click **Apply (3)**.
   
      ![](Media/lab1-11-2.1.png)

1. In the PowerShell session in the Azure Cloud Shell pane, run the following command to register the **Microsoft.DesktopVirtualization** resource provider:

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
    ```

    ![](Media/lab1-11-3.png)
   
    > **Note:** Do not wait for the registration to complete. This might take a couple of minutes.

1. Close the Cloud Shell pane.

1. On the Azure portal, in the **Search resources, services and docs (G+/)** box at the top of the portal, search for **Virtual networks (1)** and select **Virtual networks (2)**.   

     ![](Media/lab1-11-4.png)

1. Select **+ Create** on the **Network foundation | Virtual networks** page.

     ![](Media/lab1-11-5.png)

1. On the **Basics** tab of the **Create virtual network** page, specify the following settings and select **Next (5)**:

    |Setting|Value|
    |---|---|
    |Subscription|Choose the default subscription **(1)**|
    |Resource group|The name of a new resource group **az140-11e-RG (2)**|
    |Virtual network name|**az140-vnet11e (3)**|
    |Region|**<inject key="Region" enableCopy="false" /> (4)**|

     ![](Media/lab1-11-6.png)

1. On the **Security** tab, accept the default settings and select **Next**.
   
1. On the **IP addresses** tab, apply the following settings (modify the default if needed):

    |Setting|Value|
    |---|---|
    |IP address space|**10.20.0.0/16 (1)**|

    ![](Media/lab1-11-7.png)

1. Select the **edit (pencil) (2)** icon next to the **default** subnet entry, in the **Edit** pane, specify the following settings (leave others with their existing values) and select **Save (4)**:

    |Setting|Value|
    |---|---|
    |Name|**hp1-Subnet (1)**|
    |Starting address|**10.20.1.0 (2)**|
    |Enable private subnet (no default outbound access)|Disabled **(3)**|

     ![](Media/lab1-11-8.png)

1. Back on the **IP addresses** tab, select **Review + create**.

   ![](Media/lab1-11-9.png)

1. On the **Review + create** tab, select **Create**.

    ![](Media/lab1-11-10.png)

    > **Note:** Do not wait for the provisioning process to complete. This typically takes less than 1 minute.

1. In the Azure portal, search for **Microsoft Entra ID (1)** and select **Microsoft Entra ID (2)**.

    ![](Media/lab1-11-11.png)

1. On the **Overview** page of the Microsoft Entra tenant associated with your subscription, in the **Manage** section of the vertical navigation menu, select **Users**.

    ![](Media/lab1-11-11.png)

1. On the **Users** page, in the **Search** text box, enter the name of the **ODL_User<inject key="DeploymentID" enableCopy="false"/>** account listed on the Resources tab on the right side of the lab session window.
   
1. In the list of results of the search, select the user account entry with the matching name.
   
1. On the page displaying the properties of the user account, in the **Manage** section of the vertical navigation menu, select **Groups**.
   
1. On the **Groups** page, record the name of the group starting with the **AVD-DAG** prefix (you will need it later in this lab).
   
    ![](Media/lab1-11-12.1.png)
   
1. Navigate back to the **Users** page, in the **Search** text box, enter the name of the `User2-` account listed on the Resources tab on the right side of the lab session window.
   
1. In the list of results of the search, select the user account entry with the matching name.
   
1. On the page displaying the properties of the user account, in the **Manage** section of the vertical navigation menu, select **Groups**.
   
1. On the **Groups** page, record the name of the group starting with the **AVD-RemoteApp** prefix (you will need it later in this lab).

     ![](Media/lab1-11-12.2.png)

### Task 2: Deploy an Azure Virtual Desktop host pool

1. In the Azure portal, search for**Azure Virtual Desktop (1)** and select **Azure Virtual Desktop (2)** from the list.

    ![](Media/lab1-11-13.png)

1. On the **Azure Virtual Desktop** page, in the **Manage (1)** section of the vertical navigation menu, select **Host pools (2)** and, on the **Azure Virtual Desktop \| Host pools** page, select **+ Create (3)**.

    ![](Media/lab1-11-14.png)
   
1. On the **Basics** tab of the **Create a host pool** page, specify the following settings and select **Next : Session hosts > (10)** (leave other settings with their default values):

    |Setting|Value|
    |---|---|
    |Subscription|Choose the default subscription **(1)**|
    |Resource group|The name of a new resource group **az140-21e-RG (2)**|
    |Host pool name|**az140-21-hp1 (3)**|
    |Location|**<inject key="Region" enableCopy="false" /> (4)**|
    |Validation environment|**No (5)**|
    |Preferred app group type|**Desktop (6)**|
    |Host pool type|**Pooled (7)**|
    |Create Session Host Configuration|**No (8)**|
    |Load balancing algorithm|**Breadth-first (9)**|

    ![](Media/lab1-11-15.png)

    ![](Media/lab1-11-16.png)

    > **Note:** When using the Breadth-first load balancing algorithm, the max session limit parameter is optional.

1. On the **Session hosts** tab of the **Create a host pool** page, specify the following settings and select **Next : Workspace > (23)** (leave other settings with their default values):

    |Setting|Value|
    |---|---|
    |Add virtual machines|**Yes (1)**|
    |Resource group|**az140-21e-RG (2)**|
    |Name prefix|**sh-<inject key="DeploymentID" enableCopy="false"/> (3)**|
    |Virtual machine type|**Azure virtual machine (4)**|
    |Virtual machine location|**<inject key="Region" enableCopy="false" /> (5)**|
    |Availability options|**No infrastructure redundancy required (6)**|
    |Security type|**Trusted launch virtual machines (7)**|

    ![](Media/lab1-11-17.png)
   
    |Image|**Windows 11 Enterprise multi-session, Version 23H2 + Microsoft 365 Apps (8)**|
    |Virtual machine size|**Standard DC2s_v3 (9)**|
    |Number of VMs|**2 (10)**|
    |OS disk type|**Standard SSD (11)**|
    |OS disk size|**Default size (128GiB) (12)**|
    |Boot Diagnostics|**Enable with managed storage account (recommended) (13)**|
    |Virtual network|**az140-vnet11e (14)**|
    |Subnet|**hp1-Subnet (15)**|
    |Network security group|**Basic (16)**|
    |Public inbound ports|**No (17)**|

    ![](Media/lab1-11-17.2.png)

    >**Note:** In the Images section, click **See all images**, scroll down, click **Select (1)** under the Windows multi-     session image tile, and choose **Windows 11 Enterprise multi-session, Version 23H2 + Microsoft 365 Apps (2)**.

     ![](Media/lab1-11-17.1.png)

    |Select which directory you would like to join|**Microsoft Entra ID (18)**|
    |Enroll VM with Intune|**No (19)**|
    |User name|**Student (20)**|
    |Password|Any sufficiently complex string of characters that will be used as the password for the built-in administrator account **(21)**|
    |Confirm password|The same string of characters you specified previously **(22)**|

    ![](Media/lab1-11-19.png)

    > **Note:** The password should be at least 12 characters in length and consist of a combination of lower-case characters, upper-case characters, digits, and special characters. For details, refer to the information about [the password requirements when creating an Azure VM](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-).

1. On the **Workspace** tab of the **Create a host pool** page, confirm the following setting and select **Review + create (2)**:

    |Setting|Value|
    |---|---|
    |Register desktop app group|**No (1)**|

      ![](Media/lab1-11-20.png)

1. On the **Review + create** tab of the **Create a host pool** page, select **Create**.

    ![](Media/lab1-11-21.png)
   
    > **Note:** Please wait for the deployment to complete. This may take approximately 20 minutes.

### Task 3: Create an Azure Virtual Desktop application group

1. In the Azure portal, search for and select **Azure Virtual Desktop** and, on the **Azure Virtual Desktop** page, select **Application groups** under the **Manage** section from the left-side menu.

    ![](Media/lab1-11-22.png)
   
1. On the **Azure Virtual Desktop \| Application groups** page, note the existing, auto-generated **az140-21-hp1-DAG** desktop application group, and select it.

    ![](Media/lab1-11-23.png)
   
1. On the **az140-21-hp1-DAG** page, in the **Manage (1)** section of the vertical navigation menu, select **Assignments (2)**.

1. On the **az140-21-hp1-DAG \| Assignments** page, select **+ Add (3)**.

    ![](Media/lab1-11-24.png)

1. On the **Select Microsoft Entra users or user groups** page, select **Groups**, in the search box, type the full name of the **AVD-DAG (1)** group you identified in the first task of this exercise, select the checkbox next to the group name, and click **Select (2)**.

    ![](Media/lab1-11-25.png)

1. Navigate back to the **Azure Virtual Desktop \| Application groups** page, select **+ Create**. 

    ![](Media/lab1-11-26.png)

1. On the **Basics** tab of the **Create an application group** page, specify the following settings and select **Next : Applications > (6)**:

    |Setting|Value|
    |---|---|
    |Subscription|Choose the default subscription **(1)**|
    |Resource group|**az140-21e-RG (2)**|
    |Host pool|**az140-21-hp1 (3)**|
    |Application group type|**Remote App (4)**|
    |Application group name|**az140-21-hp1-Office365-RAG (5)**|

     ![](Media/lab1-11-27.png)

1. On the **Applications** tab of the **Create an application group** page, select **+ Add applications**.

1. On the **Add application** page, specify the following settings and select **Review + add (6)**, then select **Add**:

    |Setting|Value|
    |---|---|
    |Application source|**Start menu (1)**|
    |Application|**Word (2)**|
    |Display name|**Microsoft Word (3)**|
    |Description|**Microsoft Word (4)**|
    |Require command line|**No (5)**|

     ![](Media/lab1-11-28.png)

     ![](Media/lab1-11-29.png)

1. Back on the **Applications** tab of the **Create an application group** page, select **+ Add applications**.

    ![](Media/lab1-11-30.png)

1. On the **Add application** page, specify the following settings and select **Review + add (6)**, then select **Add**:

    |Setting|Value|
    |---|---|
    |Application source|**Start menu (1)**|
    |Application|**Excel (2)**|
    |Display name|**Microsoft Excel (3)**|
    |Description|**Microsoft Excel (4)**|
    |Require command line|**No (5)**|

     ![](Media/lab1-11-31.png)

1. Back on the **Applications** tab of the **Create an application group** page, select **+ Add applications**.

1. On the **Add application** page, specify the following settings and select **Review + add (6)**, then select **Add**:

    |Setting|Value|
    |---|---|
    |Application source|**Start menu (1)**|
    |Application|**PowerPoint (2)**|
    |Display name|**Microsoft PowerPoint (3)**|
    |Description|**Microsoft PowerPoint (4)**|
    |Require command line|**No (5)**|

     ![](Media/lab1-11-32.png)

1. Back on the **Applications** tab of the **Create an application group** page, select **Next : Assignments >**.

    ![](Media/lab1-11-33.png)

1. On the **Assignments** tab of the **Create an application group** page, select **+ Add Microsoft Entra users or user groups**.

    ![](Media/lab1-11-34.png)

1. On the **Select Microsoft Entra users or user groups** page, select **Groups (1)**, type the full name of the **AVD-RemoteApp (2)** group you identified in the first task of this exercise, select the checkbox next to the group name, and click **Select (3)**.

    ![](Media/lab1-11-35.png)

1. Back on the **Assignments** tab of the **Create an application group** page, select **Next : Workspace >**.

    ![](Media/lab1-11-36.png)

1. On the **Workspace** tab of the **Create a workspace** page, specify the following setting and select **Review + create (2)**:

    |Setting|Value|
    |---|---|
    |Register application group|**No (1)**|

    ![](Media/lab1-11-37.png)

1. On the **Review + create** tab of the **Create an application group** page, select **Create**.

    ![](Media/lab1-11-38.png)

    > **Note:** Wait for the Application Group to be created. This should take less than 1 minute. 

    > **Note:** Next, you will create an application group based on the file path as the application source.

1. In the Azure portal, search for and select **Azure Virtual Desktop** and, on the **Azure Virtual Desktop** page, select **Application groups** under the **Manage** section from the left-side menu.

1. On the **Azure Virtual Desktop \| Application groups** page, select **+ Create**. 

1. On the **Basics** tab of the **Create an application group** page, specify the following settings and select **Next : Applications > (6)**:

    |Setting|Value|
    |---|---|
    |Subscription|Choose the default subscription **(1)**|
    |Resource group|**az140-21e-RG (2)**|
    |Host pool|**az140-21-hp1 (3)**|
    |Application group type|**RemoteApp (4)**|
    |Application group name|**az140-21-hp1-Utilities-RAG (5)**|

    ![](Media/lab1-11-39.png)

1. On the **Applications** tab of the **Create an application group** page, select **+ Add applications**.

1. On the **Add application** page, on the **Basics** tab, specify the following settings and select **Next (7)**:

    |Setting|Value|
    |---|---|
    |Application source|**File path (1)**|
    |Application path|**C:\Windows\system32\cmd.exe (2)**|
    |Application identifier|**Command Prompt (3)**|
    |Display name|**Command Prompt (4)**|
    |Description|**Windows Command Prompt (5)**|
    |Require command line|**No (6)**|

     ![](Media/lab1-11-40.png)

1. On the **Icon** tab, specify the following settings and select **Review + add (3)**, then select **Add**:

    |Setting|Value|
    |---|---|
    |Icon path|**C:\Windows\system32\cmd.exe (1)**|
    |Icon index|0 **(2)**|

     ![](Media/lab1-11-41.png)
     ![](Media/lab1-11-42.png)

1. Back on the **Applications** tab of the **Create an application group** page, select **Next : Assignments >**.

    ![](Media/lab1-11-43.png)

1. On the **Assignments** tab of the **Create an application group** page, select **+ Add Microsoft Entra users or user groups**.

1. On the **Select Microsoft Entra users or user groups** page, select **Groups (1)**, type the full name of the **AVD-RemoteApp (2)** group you identified in the first task of this exercise, select the checkbox next to the group name, and click **Select (3)**.

    ![](Media/lab1-11-35.png)

1. Back on the **Assignments** tab of the **Create an application group** page, select **Next : Workspace >**.

    ![](Media/lab1-11-44.png)

1. On the **Workspace** tab of the **Create a workspace** page, specify the following setting and select **Review + create (2)**:

    |Setting|Value|
    |---|---|
    |Register application group|**No (1)**|

     ![](Media/lab1-11-45.png)

1. On the **Review + create** tab of the **Create an application group** page, select **Create**.

    ![](Media/lab1-11-46.png)

    > **Note:** Wait for the Application Group to be created. This should take less than 1 minute. 

### Task 4: Create an Azure Virtual Desktop workspace

1. In the Azure portal, search for and select **Azure Virtual Desktop** and, on the **Azure Virtual Desktop** page, select **Workspaces (1)**.

1. On the **Azure Virtual Desktop \| Workspaces** page, select **+ Create (2)**. 

    ![](Media/lab1-11-47.png)

1. On the **Basics** tab of the **Create a workspace** page, specify the following settings and select **Next : Application groups > (6)**:

    |Setting|Value|
    |---|---|
    |Subscription|Choose the default subscription **(1)**|
    |Resource group|**az140-21e-RG (2)**|
    |Workspace name|**az140-21-ws1 (3)**|
    |Friendly name|**az140-21-ws1 (4)**|
    |Location|**<inject key="Region" enableCopy="false" /> (5)**|

    ![](Media/lab1-11-48.png)

1. On the **Application groups** tab of the **Create a workspace** page, specify the following settings:

    |Setting|Value|
    |---|---|
    |Register application groups|**Yes (1)**|

1. On the **Workspace** tab of the **Create a workspace** page, select **+ Register application groups (2)**.

    ![](Media/lab1-11-49.png)

1. On the **Add application groups** page, select the **plus sign (1)** next to the **az140-21-hp1-DAG**, **az140-21-hp1-Office365-RAG**, and **az140-21-hp1-Utilities-RAG** entries and click **Select (2)**. 

    ![](Media/lab1-11-50.png)

    ![](Media/lab1-11-51.png)

1. Back on the **Application groups** tab of the **Create a workspace** page, select **Review + create**.

    ![](Media/lab1-11-52.png)

1. On the **Review + create** tab of the **Create a workspace** page, select **Create**.

    ![](Media/lab1-11-53.png)

### Task 5: Grant access to Azure Virtual Desktop host pools

> **Note:** When using Microsoft Entra joined session hosts, you need to assign to Azure Virtual Desktop users and administrators appropriate Azure role-based access control (RBAC) roles. In particular, the *Virtual Machine User Login* role is required to sign in to session hosts, and the *Virtual Machine Administrator Login* role is required for the local administrative privileges. 

1. In the Azure portal, search for and select **Resource groups** and, on the **Resource groups** page, select **az140-21e-RG**.

    ![](Media/lab1-11-54.png)
   
1. On the **az140-21e-RG** page, in the vertical navigation menu, select **Access control (IAM) (1)**.

1. On the **az140-21e-RG\|Access control (IAM)** page, select **+ Add (2)** and, in the drop-down menu, select **Add role assignment (3)**.

    ![](Media/lab1-11-55.png)
   
1. On the **Role** tab of the **Add role assignment** page, ensure that the **Job function roles** tab is selected, in the search textbox, enter **Virtual Machine User Login (1)**, in the list of results, select **Virtual Machine User Login (2)**, and then select **Next (3)**.

    ![](Media/lab1-11-56.png)

1. On the **Members** tab of the **Add role assignment** page, ensure that the **User, group, or service principal (1)** option is selected, click **+ Select members (2)**, in the **Select members** pane, locate the **AVD-RemoteApp (3)** group you identified in the first task of this exercise, and click **Select (5)**.

    ![](Media/lab1-11-57.png)

1. Back on the **Members** tab of the **Add role assignment** page, select **Next**.

     ![](Media/lab1-11-58.png)

1. On the **Review + assign** tab of the **Add role assignment** page, select **Review + assign**. 

    ![](Media/lab1-11-59.png)

1. Back on the **az140-21e-RG\|Access control (IAM)** page, select **+ Add** and, in the drop-down menu, select **Add role assignment**.

1. On the **Role** tab of the **Add role assignment** page, ensure that the **Job function roles** tab is selected, in the search textbox, enter **Virtual Machine Administrator Login (1)**, in the list of results, select **Virtual Machine Administrator Login (2)**, and then select **Next (3)**.

    ![](Media/lab1-11-60.png)

1. On the **Members** tab of the **Add role assignment** page, ensure that the **User, group, or service principal (1)** option is selected, click **+ Select memebers (2)**, in the **Select members** pane, locate the **AVD-DAG (3)** group you identified in the first task of this exercise, and click **Select (5)**.

    ![](Media/lab1-11-61.png)

1. Back on the **Members** tab of the **Add role assignment** page, select **Next** and on **Review + assign** tab select **Review + assign**.

    ![](Media/lab1-11-63.png)
