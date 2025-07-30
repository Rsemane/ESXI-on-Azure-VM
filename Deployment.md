# ESXi Deployment on Azure VM (Nested Virtualization)

🔧 Requirements for Nested Virtualization in Azure
- **VM Size**: Only certain VM sizes support nested virtualization, such as: D_v3, E_v3, F2s_v2, M, and newer series
- **OS**: The host VM must run a supported OS like Windows Server or Windows 10/11 Pro/Enterprise.
- **Hypervisor**: Hyper-V must be enabled inside the VM.

## Step 1: Create Azure VM with Nested Virtualization
![CreationofVM](images/nested-vm-azure-portal.png)
![CreationofVM](images/nested-vm-azure-portal-2.png)

Note: All other settings remain at their default values.

## Step 2: Install Hyper-V
```powershell
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
```
The command will install Hyper-V Feature with Hyper-V Manager and then restart the Server

## Step 3: Enable Nested Virtualization
!Step 3

## Step 4: Install and Configure ESXi
...

## Step 5: Create VMs Inside ESXi
...
