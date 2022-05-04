# Create user
havkebab
pass

# Install tailgate
Need predefined auth key

# Setup ssh
-o ListenAddress=$(ip addr | awk '/inet/ && /tun0/{sub(/\/.*$/,"",$2); print $2}')
https://github.com/SirCAS.keys

# Setup locale
TZ + Locale dk_EN.UTF8

# Install firewall
sudo apt install ufw gufw

# Setup automatic upgrades
sudo apt-get install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades

# Patch boot.txt
spi + uart

# Setup can bus
sudo apt-get install can-utils
sudo /sbin/ip link set can0 up type can bitrate 250000
candump can0

sudo nano /etc/network/interfaces
echo "
auto can0
iface can0 inet manual
    pre-up /sbin/ip link set can0 type can bitrate 250000
    up /sbin/ifconfig can0 up
    down /sbin/ifconfig can0 down
    " >> /etc/network/interfaces.d/can0

# Install desktop env

### Install
# not needed sudo apt install xserver-xorg
# if you don't want xfce4sudo apt install raspberrypi-ui-mods
sudo apt install xfce4 xfce4-terminal
# not needed sudo apt install lightdm
reboot

### Tweaks
# Remove panel #2
# Install temp widget?
# Background BG


# Install openCPN

### Add backports channels (https://github.com/OpenCPN/OpenCPN/discussions/2356)
echo "deb http://deb.debian.org/debian bullseye-backports main contrib non-free" >> /etc/apt/sources.list
echo "#deb-src http://deb.debian.org/debian bullseye-backports main contrib non-free" >> /etc/apt/sources.list

### Do the actual install
sudo apt-get update
sudo apt install opencpn -t bullseye-backports

### Copy CM93 charts
?

# Install SignalK (https://github.com/SignalK/signalk-server/blob/master/raspberry_pi_installation.md)
### Install NodeJS and NPM
curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs
sudo npm install -g npm@latest

### Install bonjour (Avahi)
sudo apt install libnss-mdns avahi-utils libavahi-compat-libdnssd-dev

### Install SignalK
sudo npm install -g signalk-server
# signalk-server --sample-nmea0183-data
sudo signalk-server-setup # (use port 80)
# May copy /home/pi/.signalk/defaults.json
# May copy /home/pi/.signalk/settings.json
# May copy /home/pi/.signalk/security.json


### Setup
1. Settings -> Data Connections ->
   1. Add ->
        Data Type: NMEA 2000
        Enabled: True
        Logging: False
        ID: PiCAN-M-N2K
        NMEA 2000 Source: Canbus (canboatjs)
        Interface: can0
        -> Apply
   1. Add ->
        Data Type: NMEA 0183
        Enabled: True
        Logging: False
        ID: PiCAN-M-NMEA0183
        NMEA 0183 Source: Serial
        Serial port: /dev/ttyS0
        Baud Rate: 4800
        -> Apply


Install:
- signalk-derived-data
- signalk-raspberry-pi-temperature
- signalk-to-influxdb
- vedirect-serial-usb
- signalk-autopilot 


# Install InfluxDB + Grafana
### Install flux + grafana using APT

wget -qO- https://repos.influxdata.com/influxdb.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/influxdb.gpg > /dev/null
export DISTRIB_ID=$(lsb_release -si); export DISTRIB_CODENAME=$(lsb_release -sc)
echo "deb [signed-by=/etc/apt/trusted.gpg.d/influxdb.gpg] https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list > /dev/null

wget -qO- https://packages.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/trusted.gpg.d/grafana.gpg] https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

sudo apt-get update
sudo apt-get install influxdb grafana

sudo systemctl daemon-reload
sudo service influxdb start
sudo systemctl start grafana-server
sudo systemctl status grafana-server
sudo systemctl enable grafana-server.service

### Create DB
influx
create database havkebabben
create user havkebab with password '<redacted>'

### Setup plugin
1. Server -> Plugin Config -> InfluxDb writer
   Fill in:
   protocol=http
   host=localhost
   port=8086
   username=havkebab
   password=<redacted>
   database=havkebabben

### Setup Grafana
Configuration -> Data Sources and click on “Add data source”
Select InfluxDB as the data source type

On the resulting screen, give it a useful name (I use my boat name, which is the same as the database name) and
populate the URL field with http://localhost:8086, which is the address for your local InfluxDB server.

# Install VictronConnect
https://www.victronenergy.com/support-and-downloads/software#victronconnect-app
