### Nagios Core and NSClient++ installation and configuration instructions

1. [Nagios Core Installation](https://github.com/mskutin/quantum-nagios/blob/master/nagios_install.md)
2. [Nagios Core Configuration](https://github.com/mskutin/quantum-nagios/blob/master/nagios_config.md)
3. [NSClient++](https://github.com/mskutin/quantum-nagios/blob/master/nsclient_install.md)

To follow this tutorial, you must have superuser privileges on the Ubuntu 14.04 server that will run Nagios. Ideally, you will be using a non-root user with superuser privileges.

<hr>
In this tutorial Microsoft Azure will be used as a cloud platform, hence the guide contains technical details that takes place for this provider, but may also appears in other solutions.

This tutorial assumes that your server has private networking enabled. If it doesn't, just replace all the references to private IP addresses with public IP ones.

Here is the typical network map, you should have at least two servers in one private network.

![image](https://cloud.githubusercontent.com/assets/11622907/17592241/0cbd4aee-6014-11e6-818e-62056038aeda.png)

The following ports should be opened in your firewall settings.
Since our network interfaces connected in the one VNET, root network security group polices will be inherited by all devices.

![image](https://cloud.githubusercontent.com/assets/11622907/17592244/10355162-6014-11e6-9628-1090de4ca798.png)

To check port status you may use telnet.

![image](https://cloud.githubusercontent.com/assets/11622907/17592246/12c2461a-6014-11e6-9100-b0417f33655f.png)

Make sure that your infrastructure supports ICMP protocol, in case of Microsoft Azure it doesn't and all public traffic will be blocked.
To enable ICMP ping requests for _private_ network on Windows VM's you should enable rule `Echo Request - ICMPv4-In` in Windows Firewall.

![2016-08-11_19-33-20](https://cloud.githubusercontent.com/assets/11622907/17587861/7da0de4a-5ffd-11e6-91dc-2bb2c4040766.png)
