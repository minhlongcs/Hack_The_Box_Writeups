#sherlock #DFIR #windows-forensics #event-logs #anti-virtualization
Difficulty: Very Easy
Categories: DFIR
Description: Talion suspects that the threat actor carried out anti-virtualization checks to avoid detection in sandboxed environments. Your task is to analyze the event logs and identify the specific techniques used for virtualization detection. Byte Doctor requires evidence of the registry checks or processes the attacker executed to perform these checks.

- Personal Summary:
    - New terms, things to learn: 
    - A Windows Management Instrumentation (WMI) class is ==a blueprint or template for representing a specific piece of hardware, software, or operating system component in Windows==.
    - **WMI Queries** ==allow you to retrieve management data from Windows operating systems using **WQL (WMI Query Language)**, a subset of SQL==. 

The industry standard for executing WMI queries is **PowerShell**, utilizing the modern **CIM cmdlets** (which replace the deprecated
    - Goals: Find the WMI classes/queries the attacker used, the VM-detection PowerShell script (function name, registry key, processes it checks), and which virtualization platforms it detected.

## Learning

- 

## Key tools + functions

- 

## Doing the sherlock

Artifacts: `PhantomCheck.zip` → Microsoft_Window_Powershell.sxtv and Windows_Powershell_Operational.sxtv 

### Task 1: Which WMI class did the attacker use to retrieve model and manufacturer information for virtualization detection?

- After a quick google search on what WMI is and which WMI can be used to perform the mentioned task: The WMI class used to retrieve a computer's model and manufacturer data is `Win32_ComputerSystem`
- Now on M_W_P file we can do a quick search for that keyword we can see that the attacker did indeed request the computer system info: ![[Pasted image 20261027222348.png]]

```
  Pipeline execution details for command line: $Model = Get-WmiObject -Class Win32_ComputerSystem | select-object -expandproperty "Model". 

Context Information: 
    DetailSequence=1
    DetailTotal=1

    SequenceNumber=27

    UserId=DESKTOP-M3AKJSD\User
    HostName=ConsoleHost
    HostVersion=5.1.26100.2161
    HostId=0fad0cf8-6cb6-4657-86f7-655ec22eed9f
    HostApplication=C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe
    EngineVersion=5.1.26100.2161
    RunspaceId=2aeeba59-d0f6-4ce7-b41c-e07625b3beec
    PipelineId=10
    ScriptName=
    CommandLine=$Model = Get-WmiObject -Class Win32_ComputerSystem | select-object -expandproperty "Model" 

Details: 
CommandInvocation(Get-WmiObject): "Get-WmiObject"
ParameterBinding(Get-WmiObject): name="Class"; value="Win32_ComputerSystem"
CommandInvocation(Select-Object): "Select-Object"
ParameterBinding(Select-Object): name="ExpandProperty"; value="Model"
ParameterBinding(Select-Object): name="InputObject"; value="\\DESKTOP-M3AKJSD\root\cimv2:Win32_ComputerSystem.Name="DESKTOP-M3AKJSD""

  ```
- Answer: Win32_ComputerSystem

### Task 2: Which WMI query did the attacker execute to retrieve the current temperature value of the machine?

- A quick keyword search of "temperature" gets us:
- ![[Pasted image 20261027223013.png]]
- Answer: SELECT * FROM MSAcpi_ThermalZoneTemperature

### Task 3: The attacker loaded a PowerShell script to detect virtualization. What is the function name of the script?

- After a quick google search i see that : PowerShell records its activity across the **Microsoft-Windows-PowerShell/Operational** and legacy engine logs: 

- **Event ID 4104 (Script Block Logging):** This is the most crucial event for security and tracking. It captures the full content and text of **script blocks** as they are loaded into memory and executed, handling any runtime de-obfuscation automatically.

- **Event ID 4105:** Logs the exact moment a **script block invocation starts**.

- **Event ID 4103 (Module Logging):** Tracks the pipeline and module initialization details, showcasing what specific commands within the script are beginning to load and execute.

- **Event ID 400:** Logs when the local **PowerShell engine starts** up to process commands or scripts.

- Now lets do a quick filter for event id 4104 in M_W_P we find:
- ![[Pasted image 20261027224426.png]]

```
Creating Scriptblock text (1 of 1):
function Check-VM
{

<# 
.SYNOPSIS 
Nishang script which detects whether it is in a known virtual machine.
 
.DESCRIPTION 
This script uses known parameters or 'fingerprints' of Hyper-V, VMWare, Virtual PC, Virtual Box,
Xen and QEMU for detecting the environment.

.EXAMPLE 
PS > Check-VM
 
.LINK 
http://www.labofapenetrationtester.com/2013/01/quick-post-check-if-your-payload-is.html
https://github.com/samratashok/nishang

.NOTES 
The script draws heavily from checkvm.rb post module from msf.
https://github.com/rapid7/metasploit-framework/blob/master/modules/post/windows/gather/checkvm.rb
#> 
    [CmdletBinding()] Param()
    $ErrorActionPreference = "SilentlyContinue"
    #Hyper-V
    $hyperv = Get-ChildItem HKLM:\SOFTWARE\Microsoft
    if (($hyperv -match "Hyper-V") -or ($hyperv -match "VirtualMachine"))
        {
            $hypervm = $true
        }

    if (!$hypervm)
        {
            $hyperv = Get-ItemProperty hklm:\HARDWARE\DESCRIPTION\System -Name SystemBiosVersion
            if ($hyperv -match "vrtual")
                {
                    $hypervm = $true
                }
        }
    
    if (!$hypervm)
        {
            $hyperv = Get-ChildItem HKLM:\HARDWARE\ACPI\FADT
            if ($hyperv -match "vrtual")
                {
                    $hypervm = $true
                }
        }
            
    if (!$hypervm)
        {
            $hyperv = Get-ChildItem HKLM:\HARDWARE\ACPI\RSDT
            if ($hyperv -match "vrtual")
                {
                    $hypervm = $true
                }
        }

    if (!$hypervm)
        {
            $hyperv = Get-ChildItem HKLM:\SYSTEM\ControlSet001\Services
            if (($hyperv -match "vmicheartbeat") -or ($hyperv -match "vmicvss") -or ($hyperv -match "vmicshutdown") -or ($hyperv -match "vmiexchange"))
                {
                    $hypervm = $true
                }
        }
   
    if ($hypervm)
        {
    
             "This is a Hyper-V machine."
    
        }

    #VMWARE

    $vmware = Get-ChildItem HKLM:\SYSTEM\ControlSet001\Services
    if (($vmware -match "vmdebug") -or ($vmware -match "vmmouse") -or ($vmware -match "VMTools") -or ($vmware -match "VMMEMCTL"))
        {
            $vmwarevm = $true
        }

    if (!$vmwarevm)
        {
            $vmware = Get-ItemProperty hklm:\HARDWARE\DESCRIPTION\System\BIOS -Name SystemManufacturer
            if ($vmware -match "vmware")
                {
                    $vmwarevm = $true
                }
        }
    
    if (!$vmwarevm)
        {
            $vmware = Get-Childitem hklm:\hardware\devicemap\scsi -recurse | gp -Name identifier
            if ($vmware -match "vmware")
                {
                    $vmwarevm = $true
                }
        }

    if (!$vmwarevm)
        {
            $vmware = Get-Process
            if (($vmware -eq "vmwareuser.exe") -or ($vmware -match "vmwaretray.exe"))
                {
                    $vmwarevm = $true
                }
        }

    if ($vmwarevm)
        {
    
             "This is a VMWare machine."
    
        }
    
    #Virtual PC

    $vpc = Get-Process
    if (($vpc -eq "vmusrvc.exe") -or ($vpc -match "vmsrvc.exe"))
        {
        $vpcvm = $true
        }

    if (!$vpcvm)
        {
            $vpc = Get-Process
            if (($vpc -eq "vmwareuser.exe") -or ($vpc -match "vmwaretray.exe"))
                {
                    $vpcvm = $true
                }
        }

    if (!$vpcvm)
        {
            $vpc = Get-ChildItem HKLM:\SYSTEM\ControlSet001\Services
            if (($vpc -match "vpc-s3") -or ($vpc -match "vpcuhub") -or ($vpc -match "msvmmouf"))
                {
                    $vpcvm = $true
                }
        }

    if ($vpcvm)
        {
    
         "This is a Virtual PC."
    
        }

    #Virtual Box

    $vb = Get-Process
    if (($vb -eq "vboxservice.exe") -or ($vb -match "vboxtray.exe"))
        {
    
        $vbvm = $true
    
        }
    if (!$vbvm)
        {
            $vb = Get-ChildItem HKLM:\HARDWARE\ACPI\FADT
            if ($vb -match "vbox_")
                {
                    $vbvm = $true
                }
        }

    if (!$vbvm)
        {
            $vb = Get-ChildItem HKLM:\HARDWARE\ACPI\RSDT
            if ($vb -match "vbox_")
                {
                    $vbvm = $true
                }
        }

    
    if (!$vbvm)
        {
            $vb = Get-Childitem hklm:\hardware\devicemap\scsi -recurse | gp -Name identifier
            if ($vb -match "vbox")
                {
                    $vbvm = $true
                }
        }

    if (!$vbvm)
        {
            $vb = Get-ItemProperty hklm:\HARDWARE\DESCRIPTION\System -Name SystemBiosVersion
            if ($vb -match "vbox")
                {
                     $vbvm = $true
                }
        }
  

    if (!$vbvm)
        {
            $vb = Get-ChildItem HKLM:\SYSTEM\ControlSet001\Services
            if (($vb -match "VBoxMouse") -or ($vb -match "VBoxGuest") -or ($vb -match "VBoxService") -or ($vb -match "VBoxSF"))
                {
                    $vbvm = $true
                }
        }

    if ($vbvm)
        {
    
         "This is a Virtual Box."
    
        }

    #Xen

    $xen = Get-Process

    if ($xen -eq "xenservice.exe")
        {
    
        $xenvm = $true
    
        }
    
    if (!$xenvm)
        {
            $xen = Get-ChildItem HKLM:\HARDWARE\ACPI\FADT
            if ($xen -match "xen")
                {
                    $xenvm = $true
                }
        }

    if (!$xenvm)
        {
            $xen = Get-ChildItem HKLM:\HARDWARE\ACPI\DSDT
            if ($xen -match "xen")
                {
                    $xenvm = $true
                }
        }
    
    if (!$xenvm)
        {
            $xen = Get-ChildItem HKLM:\HARDWARE\ACPI\RSDT
            if ($xen -match "xen")
                {
                    $xenvm = $true
                }
        }

    
    if (!$xenvm)
        {
           $xen = Get-ChildItem HKLM:\SYSTEM\ControlSet001\Services
            if (($xen -match "xenevtchn") -or ($xen -match "xennet") -or ($xen -match "xennet6") -or ($xen -match "xensvc") -or ($xen -match "xenvdb"))
                {
                    $xenvm = $true
                }
        }

    if ($xenvm)
        {
    
         "This is a Xen Machine."
    
        }

    #QEMU

    $qemu = Get-Childitem hklm:\hardware\devicemap\scsi -recurse | gp -Name identifier
    if ($qemu -match "qemu")
        {
    
            $qemuvm = $true
    
        }
    
    if (!$qemuvm)
        {
        $qemu = Get-ItemProperty hklm:HARDWARE\DESCRIPTION\System\CentralProcessor\0 -Name ProcessorNameString
        if ($qemu -match "qemu")
            {
                $qemuvm = $true
            }
        }    

    if ($qemuvm)
        {
    
         "This is a Qemu machine."
    
        }
}

ScriptBlock ID: 7c52679e-db36-49f4-87e0-675a6b23913e
Path: 

```
- Answer: Check-VM

### Task 4: Which registry key did the above script query to retrieve service details for virtualization detection?

- As we can see from the details of check-vm function we can identify the command that the attacker uses to verify virtual box:

```powershell
  if (!$vbvm)
        {
            $vb = Get-ChildItem HKLM:\SYSTEM\ControlSet001\Services
            if (($vb -match "VBoxMouse") -or ($vb -match "VBoxGuest") -or ($vb -match "VBoxService") -or ($vb -match "VBoxSF"))
                {
                    $vbvm = $true
                }
        }

    if ($vbvm)
        {
    
         "This is a Virtual Box."
    
        }
  
  ```
- Answer: ``HKLM:\SYSTEM\ControlSet001\Services

### Task 5: The VM detection script can also identify VirtualBox. Which processes is it comparing to determine if the system is running VirtualBox?

```powershell
  #Virtual Box

    $vb = Get-Process
    if (($vb -eq "vboxservice.exe") -or ($vb -match "vboxtray.exe"))
        {
    
        $vbvm = $true
    
  ```
- Answer: vboxservice.exe, vboxtray.exe

### Task 6: The VM detection script prints any detection with the prefix 'This is a'. Which two virtualization platforms did the script detect?

- 
- Answer: ``Hyper-V, Vmware``

## Takeaways / IOCs

- Attack chain: 
- Workflow to remember for event-log cases: 
