# Raspberry Pi 3 IoT Home Server with mosquitto, nodered, influxdb and grafana
Settup done in windows 10
## Get the latest image and flash the SD card

* Download the latest image from https://www.raspberrypi.org/downloads/raspbian/

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
  * More info: https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md

* Safe remove the SD card from your computer and insert it into the RPi
* Wait until RPi has booted
* Find RPis IP adress from the router
* Connect to it using ssh
  * [Putty](https://www.putty.org/) for windows
  * The default RPi username is `pi`, and the password is `raspberry`:
  
## General setup
* Change the password for the `pi` user:
```
passwd
```
* Check internet connectivity:
```
ping google.com
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
* Add the repository and install Mosquitto:

```
wget http://repo.mosquitto.org/debian/mosquitto-repo.gpg.key
sudo apt-key add mosquitto-repo.gpg.key
cd /etc/apt/sources.list.d/
sudo wget http://repo.mosquitto.org/debian/mosquitto-stretch.list
cd
sudo apt-get update
sudo apt-get install mosquitto mosquitto-clients
```
* Secure the connection:

```
$ sudo mosquitto_passwd -c /etc/mosquitto/passwd <user>
$ sudo vi /etc/mosquitto/conf.d/default.conf

user mosquitto
allow_anonymous false
password_file /etc/mosquitto/passwd
```

* Autostart service

### Time series database - InfluxDB

* Add the repository

```

```

* Install

```
$ sudo apt-get -f install
$ sudo apt-get install influxdb
```

* Start service

```
$ sudo service influxdb start
```

* Configure console to use proper timestamps

```
$ vi ~/.bash_aliases

(...)
alias idb='influx -precision "rfc3339"'
```

### System monitoring - Telegraf
* Telegraf saves system metrics to an InfluxDB database

```
$ sudo apt-get install telegraf
$ sudo service telegraf start
```

* Check the database

```

```

### Node-RED

* Install
```
bash <(curl -sL https://raw.githubusercontent.com/node-red/raspbian-deb-package/master/resources/update-nodejs-and-nodered)
```
* Start
```
node-red-start
```
* Security (uncomment adminAuth part in settings.js replacing username and password)

```
cd ~/.node-red
sudo npm install -g node-red-admin
node-red-admin hash-pw
vi settings.js
```

### Grafana

* Download it and install it

```
```

* Install plugins

```
```

* Enable service autostart at boot

```
sudo systemctl enable grafana-server
```

* Start service

```
sudo systemctl start grafana-server
```

* It should now be available from the browser on another computer `http://[IP of RPi]:3000`.
  * Can be tested by:
  * Setting local InfluxDB database `telegraf` as datasource
  * Creating a dashboard, then adding a graph of one of the stats
