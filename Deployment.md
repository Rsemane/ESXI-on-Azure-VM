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

## Step 4: Creating Hyper-V NAT Network
The objective is to connect a network interface to ESXi, enabling it to access external networks by routing traffic through the public IP address assigned to the Azure virtual machine.
To do the steps below, open Cmd in admin and copy the commands. 

1. **Creating VSwitch is a virtual network in Hyper-V named NAT-Switch with a type internal.**

```powershell
New-VMSwitch -Name "NAT-Switch" -SwitchType Internal
```
![vswitchnat](images/Networkhyper-1.png)

2. **Find the Interface Index of the New VSwitch**

```powershell
Get-NetAdapter | Where-Object { $_.InterfaceDescription -like "Hyper-V*" }
```
![vswitchnat](images/Networkhyper-6.png) 

3. **Assign an IP Address to the VSwitch**
You can use a subnet mask of your choice and IP address RFC 1918. The value of ifIndex is one gathered in the step 2 

```powershell
New-NetIPAddress -IPAddress 192.168.100.1 -PrefixLength 24 -InterfaceIndex <ifIndex>
```
![vswitchnat](images/Networkhyper-7.png) 

or you can use this command and bypass step 2

```powershell
New-NetIPAddress -IPAddress 192.168.100.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NAT-Switch)"
```
![vswitchnat](images/Networkhyper-4.png) 

**The steps above is like you have a network card and you put a static IP inside the TCP/IPv4 network settings.**
![vswitchnat](images/Networkhyper-3.png) 

4. **Create Network Natting**
The 192.168.100.0/24 network will be NATed through the public IP address of the Azure virtual machine. For example, a VM with a private IP of 192.168.100.2 will have its outbound traffic translated to the VM’s public IP.
   
```powershell
New-NetNat -Name "NATNetwork" -InternalIPInterfaceAddressPrefix 192.168.100.0/24
```
![vswitchnat](images/Networkhyper-5.png) 


## Step 5: Creating ESXI VM on the Hyper-V
**Prerequisite:** Download Custom ESXi ISO image from resources. In this Image you have the net-tulip driver that works with hyper-V. The Image is done by ESXI Customizer tool by ...xxx.xxx 
1. Open Hyper-V manager, click Action > New > Virtual Machine.
![CreatingVM](images/Creating-ESXI-VM-1.png) 

![CreatingVM](images/Creating-ESXI-VM-2.png) 

2. Specify a name for the virtual machine (e.g., ESXi6) and choose a storage location based on your preferences—ideally on a disk that doesn’t contain an operating system. Click **Next**
   
![CreatingVM](images/Creating-ESXI-VM-3.png) 

3. Select Generation 1 for your VM in order to make it possible to use a legacy network adapter with the compatible drivers that you have integrated into the ESXi installation image. Click **Next**
   
![CreatingVM](images/Creating-ESXI-VM-4.png)

**PS: Even if you create Legacy network and use official ISO not custom version 6, you will get this message**
![CreatingVM](images/no_adapter.png)

4. Assign at least 4 GB of memory. Using Dynamic Memory for this virtual machine is not recommended. Click **Next**
   
![CreatingVM](images/Creating-ESXI-VM-5.png)

5. Configure the networking settings as desired—using the “None” option or any available configuration—since the VM’s network must be manually reconfigured after creation. Note that selecting a Legacy network adapter during VM setup isn't supported, so you’ll need to remove the default adapter afterward and create a Legacy one manually. Click **Next**

![CreatingVM](images/Creating-ESXI-VM-6.png)

6. Create a new virtual disk, allocating 30 GB as a starting point. If you plan to run multiple VMware virtual machines on your ESXi host, consider assigning a larger size upfront, or add additional virtual disks later as needed. You may also opt for a dynamically expanding disk to optimize storage use. Verify the disk’s name and location, then proceed by clicking **Next**.

![CreatingVM](images/Creating-ESXI-VM-7.png)

7. Select Install an operating system from a bootable CD/DVD-ROM in Installation Options. Use the ISO image file that you have prepared beforehand (ESXi-6.x-custom.iso in this example). Click **Next**.
   
![CreatingVM](images/Creating-ESXI-VM-8.png)

8. Check the summary and click **Finish** to finalize the VM creation.
   
![CreatingVM](images/Creating-ESXI-VM-9.png)


## Step 6: Setting up ESXi VM on Hyper-V
1. Once a new Hyper-V virtual machine is created, edit the VM settings. Right-click the name of your VM and select **Settings** in the context menu.
   
![SettingVM](images/Settings-VM-1.png)

2. In the left pane of the window in the Hardware section, select Processor and set the number of virtual processors to 4 or more (1 processor is used by default). Otherwise, you will get errors during booting up phase. 

![SettingVM](images/Settings-VM-2.png)

3. Select the network adapter. First, remove the existing network adapter created by default. In order to do this, click the **Remove** button.
   
![SettingVM](images/Settings-VM-3.png)

4. Add a legacy network adapter to the VM. In the left pane of the window in the Hardware section click Add Hardware. In the right pane select Legacy Network Adapter and click **Add**.

![SettingVM](images/Settings-VM-4.png)

Choose the name of the VSwitch created in step 5. In my case is NAT-Switch

![SettingVM](images/Settings-VM-5.png)








------------------------




## Step : Enable Nested Virtualization
On Azure VM created, you have inside it ESXi VM or another OS + Hyper-V (level 3 - Architecture Diagram), pick up the name of the VM and use the command below on the host (level 2 - Architecture Diagram) 
```powershell
Set-VMProcessor -VMName "<vm-name>" -ExposeVirtualizationExtensions $true
```

## Step : Install and Configure ESXi on the Hyper-V
...

## Step : Create VMs Inside ESXi
...
