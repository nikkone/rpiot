# Raspberry Pi 3 IoT Home Server with mosquitto, nodered, influxdb and grafana
Setup on another computer using windows 10
## Making the SD card for the RPi

* Download image from https://www.raspberrypi.org/downloads/raspbian/

* Write the image to the SD card
  * https://www.balena.io/etcher/ works well
* Enable ssh for remote access to the command line:
  * https://www.raspberrypi.org/documentation/remote-access/ssh/README.md
```
For headless setup, SSH can be enabled by placing a file named ssh,
without any extension, onto the boot partition of the SD card from another computer.
```

* Configure the wifi connection:
  * https://www.raspberrypi.org/documentation/configuration/wireless/headless.md
  * Make a textfile named `wpa_supplicant.conf` in the /boot/ partition
   * More info: https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md
 ```
 country=NO
 ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
 update_config=1

 network={
     ssid="<ssid1>"
     psk="<pass1>"
     id_str="Home"
     priority=1
 }
 ```
* Remove the SD card from your computer and insert it into the RPi
* Wait until RPi has booted
* Find RPis IP adress from the router
* Connect to it using ssh
  * [Putty](https://www.putty.org/) for windows
  * The default RPi username is `pi`, and the password is `raspberry`:
  
## General setup through ssh
* Change the password for the `pi` user:
```
passwd
```
* Upgrade packages:
```
sudo apt-get update
sudo apt-get upgrade
```
* Configure options (change hostname, localisation options, etc.)
```
sudo raspi-config
```
## Installing services
### MQTT broker - Mosquitto
* Install Mosquitto:

```
sudo apt-get install mosquitto mosquitto-clients
```

* Restrict access by requiring username and password for clients:
  * Create/edit mosquitto user:password file by: `sudo mosquitto_passwd -c /etc/mosquitto/passwd <user>`
  * `<user>` = Username of added user
  * When run, asks for password, then encrypt it and writes it to `/etc/mosquitto/passwd`
  * Adding more users can be done by `sudo mosquitto_passwd -b /etc/mosquitto/passwd <another_user> password`. After using this, clear the command line history by `history -c`
  * Tell mosquitto to use user:password file and reject anonymous clients:
  * Open configfile by: `sudo nano /etc/mosquitto/conf.d/default.conf`
  * Add this to the file
```
allow_anonymous false
password_file /etc/mosquitto/passwd
```

* Autostart service
```
sudo systemctl enable mosquitto
```
* Testing can be done with an mqtt client:
 * Search for mqtt chrome extension
### Time series database - InfluxDB
* https://docs.influxdata.com/influxdb/v1.7/introduction/installation/ 
* https://community.influxdata.com/t/raspberry-pi-installation-instructions/5515/3
* Add the repository

```
curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -
echo "deb https://repos.influxdata.com/debian stretch stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
sudo apt-get update
```

* Install

```
sudo apt-get install influxdb
```

* Start service

```
sudo systemctl unmask influxdb.service
sudo systemctl start influxdb
```

### System monitoring - Telegraf
* https://www.influxdata.com/time-series-platform/telegraf/
* Telegraf saves system metrics to an InfluxDB database

```
$ sudo apt-get install telegraf
$ sudo service telegraf start
```

* Check the database

```
influx
show databases;
```
* There should be two databases now:
```
_internal
telegraf
```
### Connecting services - Node-RED
* https://nodered.org/docs/hardware/raspberrypi
* Install
```
bash <(curl -sL https://raw.githubusercontent.com/node-red/raspbian-deb-package/master/resources/update-nodejs-and-nodered)
```
* Enable start on boot
```
sudo systemctl enable nodered.service
```
* Make a password for the configuration
  * https://nodered.org/docs/security
```
cd ~/.node-red
sudo npm install -g node-red-admin
node-red-admin hash-pw
```
* Uncomment adminAuth part in settings.js replacing username and password(default is admin/password)
  * Tip: Add the password hash returned by `node-red-admin hash-pw`, not a planetext password
```
nano settings.js
```
* Either reboot or start manually:
```
node-red-start
```
* It should now be available from `http://[IP of RPi]:1880`
* To use influxdb in nodered, goto `Manage palette` and install `node-red-contrib-influxdb`
### Presenting data - Grafana
* http://docs.grafana.org/installation/debian/
* Add repository
```
deb https://packages.grafana.com/oss/deb stable main
```
* Add gpg to install signed packages
```
curl https://packages.grafana.com/gpg.key | sudo apt-key add -
```
* Ensure that the repository is being used
  * The version of grafana from rpi default repository is something 2.x.x
  * The version from the official repository is >= 6.0.0
  * Doing this wrong may result in a lot of extra work to get systemd service to work properly.
  * The `-s` in the `install grafana` line means that its only simulating an install, so you can check the version.
```
sudo apt-get update
sudo apt-get -s install grafana
```
* Actually do the install by 
```
sudo apt-get install grafana
```

* Enable service autostart at boot

```
sudo systemctl enable grafana-server.service
```

* Start service or reboot to check if it starts by itself

```
sudo systemctl start grafana-server
```

* It should now be available from the browser at `http://[IP of RPi]:3000`.
  * Can be tested by:
  * Setting local InfluxDB database `telegraf` as datasource
  * Creating a dashboard, then adding a graph of one of the stats
## Usefull commands
* Listening ports on the RPi
```
netstat -l
```
* List systemd services
```
systemctl list-units
```
