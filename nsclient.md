##NSClient++ installation

In order to provide communication between Nagios Core server we configured we are going to use NSClient++ tool.
Download the NSClient++ from its official download [page](https://www.nsclient.org/download/).
Make sure you use the right version for the server processor 32bit/ 64bit.

To save your time you may use chocolatey package manager and download latest stable NSClient version from their repository.

Run the following command with Admin credentials
```
iwr https://chocolatey.org/install.ps1 -UseBasicParsing | iex
```

```
choco install nscp -y
```

At the time of this writing, the latest stable release is 0.4.4.19, but in this tutorial we will use 0.5.0.48 version since it has a bugfix for ssl certificate handshake failure.

Go through the installation steps:

![image](https://cloud.githubusercontent.com/assets/11622907/17588440/7f360c2c-6001-11e6-91f3-26b9faa15102.png)

Put the _private_ ip address of our Nagios Core server and the password (we will use this later).

Let's go back to our Ubuntu machine and check the IP address with the following command:
```
ifconfig -a
```

![image](https://cloud.githubusercontent.com/assets/11622907/17588513/3c34d8e4-6002-11e6-916a-ee7b8f90fbbd.png)

Mark all modules that should be loaded in.

![image](https://cloud.githubusercontent.com/assets/11622907/17588468/cb733060-6001-11e6-8ece-2f7051e0e180.png)
## Windows Dameon
After installation press the Windows+R keys to open the Run dialog, type `services.msc` and find our NSClient++ service.

Enable the line `"Allow service to interact with desktop"` in the `Log On` tab. Restart the service.
![image](https://cloud.githubusercontent.com/assets/11622907/17588768/acb8d2d6-6003-11e6-9680-7f79fc0ee2ab.png)

### Custom scripts
Since we need to configure an alert based on a current rate of change and this functionality doesn't work out of the box we will use custom vbs script that receive warning and critical argument's values from our Nagios Core which we will define later. By default script contains predefined `threshold_warning = 10` and `threshold_critical = 15` variables which will be used in case of empty arguments during the call.

Let's put this script into the directory with the following name: `C:\Program Files\NSClient++\scripts\check_disk_iostat.vbs`

```vbnet
' Required Variables
Const PROGNAME = "check_disk_iostat"
Const VERSION = "1.0.2"

' Default settings for your script.
threshold_warning = 10
threshold_critical = 15
strComputer = "."

' Create the NagiosPlugin object
Set np = New NagiosPlugin

' Define what args that should be used
np.add_arg "computer", "Computer name", 0
np.add_arg "warning", "warning threshold", 0
np.add_arg "critical", "critical threshold", 0

' If we have no args or arglist contains /help or not all of the required arguments are fulfilled show the usage output,.
If Args.Exists("help") Then
	np.Usage
End If

' If we define /warning /critical on commandline it should override the script default.
If Args.Exists("warning") Then threshold_warning = Args("warning")
If Args.Exists("critical") Then threshold_critical = Args("critical")
If Args.Exists("computer") Then strComputer = Args("computer")
np.set_thresholds threshold_warning, threshold_critical
return_code = OK
'=====================================================
Set objWMI = GetObject("winmgmts://" & strComputer & "/root\cimv2")
Set objInstances = objWMI.InstancesOf("Win32_PerfFormattedData_PerfDisk_PhysicalDisk",48)

out="Disk windows iostat | "
For Each objInstance in objInstances
	if InStr(objInstance.Name,"Total") =0 then
		name=replace(objInstance.Name," ","_")
		name=" "&replace(name,":","")
		out=out & Name & "_AvgDiskQueueLength=" & objInstance.AvgDiskQueueLength&";"&threshold_warning&";"& threshold_critical &Name &"_DiskReadBytesPersec=" & objInstance.DiskReadBytesPersec & Name &"_DiskWriteBytesPersec=" & objInstance.DiskWriteBytesPersec & Name &"_PercentDiskTime=" & objInstance.PercentDiskTime
		return_code = np.escalate_check_threshold(return_code, objInstance.AvgDiskQueueLength)
	end if
Next
'=====================================================
' Nice Exit with msg and exitcode

np.nagios_exit out, return_code
```

Open nscp configuration file `C:\Program Files\NSClient++\nsclient.ini` in any text editor:
Enable NRPE server and define our script:

```cfg
[/settings/NRPE/server]

; COMMAND ARGUMENT PROCESSING - This option determines whether or not the we will allow clients to specify arguments to commands that are executed.
allow arguments = true

; COMMAND ALLOW NASTY META CHARS - This option determines whether or not the we will allow clients to specify nasty (as in |`&><'"\[]{}) characters in arguments.
allow nasty characters = false

; ALLOWED HOSTS - A comaseparated list of allowed hosts. You can use netmasks (/ syntax) or * to create ranges. parent for this key is found under: /settings/default this is marked as advanced in favour of the parent.
allowed hosts = localhost,10.1.0.8

; PORT NUMBER - Port to use for NRPE.
port = 5666

[/settings/external scripts/wrapped scripts]
check_disk_iostat=check_disk_iostat.vbs
```

### Nagios Core configuration for Windows hosts
> The following configurations need to be made on the Nagios server.

Uncomment windows.cfg in nagios.cfg
```
vi /usr/local/nagios/etc/nagios.cfg
```

Uncomment the following line:

```
#cfg_file=/usr/local/nagios/etc/objects/windows.cfg
```

Check the ip address of the Windows machine:
![image](https://cloud.githubusercontent.com/assets/11622907/17589454/81443c9a-6007-11e6-919a-62609c11e95d.png)

Define the host and services in the windows.cfg file


```cfg
define host{
	use		windows-server	; Inherit default values from a template
	host_name	winserver	; The name we're giving to this host
	alias		My Windows Server ; A longer name associated with the host
	address		10.1.0.7	; IP address of the host
	}

# Custom vbs script for monitoring of hdd load
define service{
	use 			generic-service
	host_name		winserver
	service_description	HDD IO Rate
	check_command		check_disk_iostat!"."!10!15
	}

# Default service supports monitoring of free space using default module check_disk
define service{
	use			generic-service
	host_name		winserver
	service_description	C:\ Drive Space
	check_command		check_nt!USEDDISKSPACE!-l c -w 80 -c 90
	}
```

> Nagios uses return values from the script to establish the state of the service, the possible return values should be 0,1,2,3.
* Exit status of 0 (OK) should be returned when service status is fully functional.
* Exit status of 1 (WARNING) should be returned when service status is not critical but is not optimal.
* Exit status of 2 (CRITICAL) should be returned when the service is "critical", aka dead.
* Exit status of 3 (Unknown) Invalid command line arguments were supplied to the plugin or low-level failures internal to the plugin.


>In the vbs script example above, the check command expects three variables identified by `!"."!10!15`. The values for these variables are provided during the definition of the service via positional elements that are separated by !.

`check_disk_iostat!"."!10!15`
* Where `"."` is a name of computer we can omit this value it is already predefined in script itself.
* "!10" is a number of warning threshold
* "!15" is a number of critical threshold


`check_nt!USEDDISKSPACE!-l c -w 80 -c 90`
* where 80 is percentage of warning alert
* 90 is critical



If you set a password in NSclient++. You need to include the password in the check_nt command definition.

```
vi /usr/local/nagios/etc/objects/commands.cfg
```

```cfg
#Password from nscp.ini Windows machine
define command{
	command_name	check_nt
	command_line	$USER1$/check_nt -H $HOSTADDRESS$ -p 12489 -s admin -v $ARG1$ $ARG2$
	}

# 'check_disk_iostat' command definition
define command{
        command_name    check_disk_iostat
        command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -c check_disk_iostat $ARG1$ /warning:$ARG2$ /critical:$ARG3$
        }
```

Reload Nagios Core server to apply changes.

```
sudo service nagios restart
```

Go to the web interface and check out the state of our Windows machine.

![image](https://cloud.githubusercontent.com/assets/11622907/17590175/3e94468e-600b-11e6-9e5d-4cdcc51f03d7.png)

Also we can check our check_dist_iostat script using `check_nrpe` utility and CrystalDiskMark benchmark to simulate hdd load.

Before benchmark we will check the current state.
Execute this command on nagios server:
```
check_nrpe -H 10.1.0.7 -c check_disk_iostat
```

![image](https://cloud.githubusercontent.com/assets/11622907/17590686/39e435a2-600d-11e6-9894-26e4fd7060cc.png)

Run benchmark tests on windows machine.

![image](https://cloud.githubusercontent.com/assets/11622907/17590529/ab5a38c2-600c-11e6-9e54-5cf7f7403001.png)

And go back to the Nagios.

![image](https://cloud.githubusercontent.com/assets/11622907/17590737/742bb65e-600d-11e6-9dad-e8eeeeba998c.png)

![image](https://cloud.githubusercontent.com/assets/11622907/17590783/ab78c1c4-600d-11e6-878d-6f2d5a2c9cef.png)
