## ESXi ISO

https://1drv.ms/u/c/5c4ee44f56a942bf/EUtcfc7ad5FMmo7hcMuJVVwBYXRsSjBoEPkhbcAb_uwoRA?e=R5F5SF

## Azure VM specs

https://learn.microsoft.com/en-us/answers/questions/813416/how-do-i-know-what-size-azure-vm-supports-nested-v

## Network configuration
 $switchName = "test7" 
 New-VMSwitch -Name $switchName -SwitchType Internal 
New-NetNat –Name $switchName –InternalIPInterfaceAddressPrefix 10.6.0.0/24 
 $ifIndex = (Get-NetAdapter | ? {$_.name -like "*$switchName)"}).ifIndex 
 New-NetIPAddress -IPAddress 10.6.0.10 -InterfaceIndex $ifIndex -PrefixLength 24


 Add-DhcpServerV4Scope -Name "DHCP-$switchName" -StartRange 10.6.0.50 -EndRange 10.6.0.100 -SubnetMask 255.255.255.0 
 Set-DhcpServerV4OptionValue -Router 10.6.0.10 -DnsServer 168.63.129.16 
 Restart-service dhcpserver 
