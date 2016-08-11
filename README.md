### Prerequisites

To follow this tutorial, you must have superuser privileges on the Ubuntu 14.04 server that will run Nagios. Ideally, you will be using a non-root user with superuser privileges.

> This tutorial assumes that your server has private networking enabled. If it doesn't, just replace all the references to private IP addresses with public IP addresses.

> Make sure that your infrastructure supports ICMP protocol, like in case of Microsoft Azure it doesn't and all public traffic will be blocked.
To enable ICMP ping requests for private network on Windows VM's you should enable rule `Echo Request - ICMPv4-In` in
Windows Firewall.

![2016-08-11_19-33-20](https://cloud.githubusercontent.com/assets/11622907/17587861/7da0de4a-5ffd-11e6-91dc-2bb2c4040766.png)
