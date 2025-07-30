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
2. Go to Remote Desktop on your PC in Start Menu -> paste the Public IP Address -> Click on **Connect** -> fill the username and password (Credentials) -> Click on **OK**
   
![ConnecttoVM](images/connect-rdp-3.png)

4. Click on **Yes**
   
![ConnecttoVM](images/connect-rdp-6.png)

### 2nd Method
1. Inside the Virtual Machine in Azure Portal Click on button **Connect**
   
![ConnecttoVM](images/connect-rdp-1.png)

3. Click on **Download RDP file**
   
![ConnecttoVM](images/connect-rdp-2.png)

5. Click on **Keep** to download the file
   
![ConnecttoVM](images/download-rdp-file.png)

7. Open the file and click on **Connect**
   
![ConnecttoVM](images/connect-rdp-4.png)

9. Fill your credentials and then click on **OK**
    
![ConnecttoVM](images/connect-rdp-5.png)

11. Click on **Yes**
   
![ConnecttoVM](images/connect-rdp-6.png)


## Step 3: Install Hyper-V
Right click on the Window Logo -> Click on **Windows PowerShell (Admin)**

![PowershellAdmin](images/Powershell-admin.png)

```powershell
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
```
The command will install Hyper-V Feature with Hyper-V Manager and then restart the Server

## Step 4: Creating ESXI VM on the Hyper-V

## Step 5: Enable Nested Virtualization
On Azure VM created, you have inside it ESXi VM or another OS + Hyper-V (level 3 - Architecture Diagram), pick up the name of the VM and use the command below on the host (level 2 - Architecture Diagram) 
```powershell
Set-VMProcessor -VMName "<vm-name>" -ExposeVirtualizationExtensions $true
```

## Step 6: Install and Configure ESXi on the Hyper-V
...

## Step 7: Create VMs Inside ESXi
...
