# ESXi Deployment on Azure VM (Nested Virtualization)

🔧 Requirements for Nested Virtualization in Azure
- **VM Size**: Only certain VM sizes support nested virtualization, such as: D_v3, E_v3, F2s_v2, M, and newer series
- **OS**: The host VM must run a supported OS like Windows Server or Windows 10/11 Pro/Enterprise.
- **Hypervisor**: Hyper-V must be enabled inside the VM.

## Step 1: Create Azure VM with Nested Virtualization
![CreationofVM](images/nested-vm-azure-portal.png)
![CreationofVM](images/nested-vm-azure-portal-2.png)

Note: All other settings remain at their default values. Make sure to enable RDP in NSG and use public IP Standard SKU. 

## Step 2 : Connect to VM
### 1st Method
1. Go to Azure portal -> Virtual Machines -> choose the VM that you created -> copy the Public IP Address 
2. Go to Remote Desktop on your PC in Start Menu -> paste the Public IP Address -> Click on Connect -> fill the username and password (Credentials) -> Click on OK
![ConnecttoVM](images/connect-rdp-3.png)
3. Click on **Yes** 
![ConnecttoVM](images/connect-rdp-6.png)

### 2nd Method
1. Inside the Virtual Machine in Azure Portal Click on button Connect 
![ConnecttoVM](images/connect-rdp-1.png)
2. Click on Download RDP file
![ConnecttoVM](images/connect-rdp-2.png)
3. Click on Keep to download the file 
![ConnecttoVM](images/download-rdp-file.png)
4. Open the file and click on Connect 
![ConnecttoVM](images/connect-rdp-4.png)
5. Fill your credentials and then click on OK
![ConnecttoVM](images/connect-rdp-5.png)
6. Click on Yes
![ConnecttoVM](images/connect-rdp-6.png)


## Step 3: Install Hyper-V
```powershell
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
```
The command will install Hyper-V Feature with Hyper-V Manager and then restart the Server

## Step 4: Enable Nested Virtualization
!Step 4

## Step 5: Install and Configure ESXi
...

## Step 6: Create VMs Inside ESXi
...
