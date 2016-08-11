## Install Nagios 4
This section will cover how to install Nagios 4 on your monitoring server. You only need to complete this section once.

### Create Nagios User and Group

We must create a user and group that will run the Nagios process. Create a `nagios` user and `nagcmd` group, then add the user to the group with these commands:
```bash
sudo useradd nagios
sudo groupadd nagcmd
sudo usermod -a -G nagcmd nagios
```
### Install Build Dependencies

Because we are building Nagios Core from source, we must install a few development libraries that will allow us to complete the build. While we're at it, we will also install `apache2-utils`, which will be used to set up the Nagios web interface.

First, update your apt-get package lists:

```
sudo apt-get update
```

Then install the required packages:

```
sudo apt-get install build-essential libgd2-xpm-dev openssl libssl-dev xinetd apache2-utils unzip
```

Let's install Nagios now.

### Install Nagios Core

Download the source code for the latest stable release of Nagios Core. Go to the Nagios downloads page, and click the Skip to download link below the form. Copy the link address for the latest stable release so you can download it to your Nagios server.

At the time of this writing, the latest stable release is Nagios 4.1.1. Download it to your home directory with curl:

```
cd ~
curl -L -O https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.1.1.tar.gz
```

Extract the Nagios archive with this command:

```
tar xvf nagios-*.tar.gz
```

Then change to the extracted directory:

```
cd nagios-*
```
Before building Nagios, we must configure it. To use postfix, add `--with-mail=/usr/sbin/sendmail` to the following command:

```
./configure --with-nagios-group=nagios --with-command-group=nagcmd
```
Now compile Nagios with this command:

```
make all
```

Now we can run these make commands to install Nagios, init scripts, and sample configuration files:

```
sudo make install
sudo make install-commandmode
sudo make install-init
sudo make install-config
sudo /usr/bin/install -c -m 644 sample-config/httpd.conf /etc/apache2/sites-available/nagios.conf
```

In order to issue external commands via the web interface to Nagios, we must add the web server user, www-data, to the nagcmd group:

```
sudo usermod -G nagcmd www-data
```

### Install Nagios Plugins

Find the latest release of Nagios Plugins [here](http://nagios-plugins.org/download/?C=M;O=D). Copy the link address for the latest version, and copy the link address so you can download it to your Nagios server.

At the time of this writing, the latest version is Nagios Plugins 2.1.2. Download it to your home directory with curl:

```
cd ~
curl -L -O http://nagios-plugins.org/download/nagios-plugins-2.1.2.tar.gz
```

Extract Nagios Plugins archive with this command:
```
tar xvf nagios-plugins-*.tar.gz
```

Then change to the extracted directory:

```
cd nagios-plugins-*
```
Before building Nagios Plugins, we must configure it. Use this command:

```
./configure --with-nagios-user=nagios --with-nagios-group=nagios --with-openssl
```

Now compile Nagios Plugins with this command:

```
make
```

Then install it with this command:

```
sudo make install
```

### Install NRPE

Find the source code for the latest stable release of NRPE at the NRPE downloads page. Download the latest version to your Nagios server.

Download it to your home directory with curl:

```
cd ~
curl -L -O http://downloads.sourceforge.net/project/nagios/nrpe-2.x/nrpe-2.15/nrpe-2.15.tar.gz
```

Extract the NRPE archive with this command:

```
tar xvf nrpe-*.tar.gz
```

Then change to the extracted directory:

```
cd nrpe-*
```

Configure NRPE with these commands:
```
./configure --enable-command-args --with-nagios-user=nagios --with-nagios-group=nagios --with-ssl=/usr/bin/openssl --with-ssl-lib=/usr/lib/x86_64-linux-gnu
```
Now build and install NRPE and its xinetd startup script with these commands:

```
make all
sudo make install
sudo make install-xinetd
sudo make install-daemon-config
```
Open the xinetd startup script in an editor:

```
sudo vi /etc/xinetd.d/nrpe
```

Modify the only_from line by adding the private IP address of the your Nagios server to the end (substitute in the actual IP address of your server):

```
only_from = 127.0.0.1 10.1.0.8
```

Save and exit. Only the Nagios server will be allowed to communicate with NRPE.

Restart the xinetd service to start NRPE:

```
sudo service xinetd restart
```

Now that Nagios 4 is installed, we need to configure it.
Next part is
[Nagios Core Configuration.](https://github.com/mskutin/quantum-nagios/blob/master/nagios_config.md)
