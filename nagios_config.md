## Configure Nagios
###Organize Nagios Configuration

Open the main Nagios configuration file in the text editor.

```
sudo vi /usr/local/nagios/etc/nagios.cfg
```

Now find an uncomment this line by deleting the #:

```
#cfg_dir=/usr/local/nagios/etc/servers
```

Save and exit.

Now create the directory that will store the configuration file for each server that you will monitor:

```
sudo mkdir /usr/local/nagios/etc/servers
```

### Configure Nagios Contacts

Open the Nagios contacts configuration in text editor.

```
sudo vi /usr/local/nagios/etc/objects/contacts.cfg
```

Find the email directive, and replace its value with your own email address:

```cfg
email                           nagios@localhost        ; <<***** CHANGE THIS TO YOUR EMAIL ADDRESS ******
```
Save and exit.

### Configure check_nrpe Command

Let's add a new command to our Nagios configuration:

```
sudo vi /usr/local/nagios/etc/objects/commands.cfg
Add the following to the end of the file:
```

```cfg
# 'check_disk_iostat' command definition
define command{
        command_name    check_disk_iostat
        command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -c check_disk_iostat $ARG1$ /warning:$ARG2$ /critical:$ARG3$
        }
```
Save and exit.

### Configure Apache

Enable the Apache rewrite and cgi modules:
```bash
sudo a2enmod rewrite
sudo a2enmod cgi
```
Use htpasswd to create an admin user, called "nagiosadmin", that can access the Nagios web interface:

```
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```

Now create a symbolic link of nagios.conf to the sites-enabled directory:
```
sudo ln -s /etc/apache2/sites-available/nagios.conf /etc/apache2/sites-enabled/
```
Nagios is ready to be started. Let's do that, and restart Apache:

```
sudo service nagios start
sudo service apache2 restart
```

To enable Nagios to start on server boot, run this command:
```
sudo ln -s /etc/init.d/nagios /etc/rcS.d/S99nagios
```

### Accessing the Nagios Web Interface
Open your web browser, and go to your Nagios server (substitute the IP address or hostname for the highlighted part):

```url
http://nagios_server_public_ip/nagios
```

Because we configured Apache to use htpasswd, you must enter the login credentials that you created earlier. We used "nagiosadmin" as the username:


![image](https://cloud.githubusercontent.com/assets/11622907/17588052/da7def44-5ffe-11e6-8cbd-75d140762d9b.png)

![image](https://assets.digitalocean.com/articles/nagios/hosts_link.png)
