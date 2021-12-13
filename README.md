# FreeTAKServer-RaspberyPi-Install
Instructions for installing FreeTAKServer, FreeTAKServer-UI, FreeTAKHub on RaspberryPI Ubuntu.


# Raspberry Pi Setup

First you will need a Raspberry Pi. 
Preferably Raspberry Pi 3 or higher and at least a 16GB micro sd card.

Next you will need the [Raspberry Pi Imager Software](https://www.raspberrypi.com/software/).

Once that is installed, insert your micro sd into your computer and install Ubuntu from the Raspberry Pi Imager Software.

## Flash Operating System
![Imager](screenshots/Imager.PNG)

Select Choose OS

![General Purpose](screenshots/GeneralPurpose.PNG)

Select General Purpose

![Ubuntu](screenshots/Ubuntu.PNG)

Select Ubuntu

![Ubuntu Server LTS](screenshots/ubuntuServer.PNG)

Select Ubuntu Server 20.04.3 LTS (64-Bit)

## Hook Up Raspberry Pi
Once the SD card has been flashed, plug the SD card into the Raspberry Pi and hook up the following connections:
 - Ethernet
 - Keyboard
 - HDMI to monitor
 - Power
# Ubuntu Setup

Default Login:
```
User: ubuntu
Pass: ubuntu
```

Once you have the Raspberry Pi booted up follow the prompts to change the default password.

Once you have changed the password update and upgrade the system.

```
sudo apt update && sudo apt upgrade -y
```

Restart if necessary.

## FreeTAK Server Setup

Install Python3 & Pip
```
sudo apt update && sudo apt install python3 && sudo apt install python3-pip
```
Next install python libraries

```
sudo apt install python3-dev python3-setuptools build-essential python3-gevent python3-lxml libcairo2-dev
```
```
sudo pip3 -m install wheel pycairo
```

Once you have the dependencies installed you can install FreeTAKServer

```
sudo python3 -m pip install FreeTAKServer[ui]
```

Verify the installation with
```
pip check FreeTakServer 
```

## Configure FTS

Type:
```
sudo python3 -m FreeTAKServer.controllers.services.FTS 
```
A configuration wizard will prompt you for the following values.

```
1.)would you like to use a yaml config file, 
 if yes you will be prompted for further configuration options [yes]: yes

2.) where would you like to save the yaml config [/opt/FTSConfig.yaml]:

3.) enter ip [Your IP]: 

4.) **enter the preferred database path [/opt/FTSDataBase.db]: **
/opt/FTSDataBase.db

5.) enter the preferred main_path [/usr/local/lib/python3.8/dist-packages/FreeTAKServer]:

6.) enter the preferred log file path [/usr/local/lib/python3.8/dist-packages/FreeTAKServer/Logs]: 

```

- Step 1: Enter yes
- Step 2: Hit enter
- Step 3: You IP will automatically be detected. Update this if you please
- Step 3-6: You can leave as default

Also, if you do not want the welcome message everytime you connect to FTS, update ConnectionMessage in MainConfig.py file.

```
cd /usr/local/lib/python3.8/dist-packages/FreeTAKServer/controllers/configuration
```

```
sudo nano MainConfig.py
```

There will be 3 instances of this that you must update
```
ConnectionMessage = None
```


## FTS-UI Configuration
Navigate to the FTS-UI directory
```
cd /usr/local/lib/python3.8/dist-packages/FreeTAKServer-UI
```

Then edit:
```
sudo nano config.py
```

Edit the following values:

```
IP = 'Your IP'
```
```
APPIP = 'Your IP
```
```
WEBMAPIP = 'Your IP'
```

Only change this value if you are using a Raspberry Pi

```
WEBMAPPORT = 1880
```

Then run:
```
cd && sudo pip3 install WTForms==2.3.3 
```

## Running FTS & FTS-UI

Setup FTS & FTS-UI as a service copy the following:

```
cd /etc/systemd/system
```

```
sudo nano FTS.service
```
Copy the following:
```
[Unit]
Description=FreeTAK Server service
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
ExecStart=/usr/bin/python3 -m FreeTAKServer.controllers.services.FTS -DataPackageIP 0.0.0.0 -AutoStart True


[Install]
WantedBy=multi-user.target
```

Then do the same for FTS-UI
```
sudo nano FTS-UI.service
```
```
[Unit]
Description=FreeTAK Server UI service
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
ExecStart=sudo python3 /usr/local/lib/python3.8/dist-packages/FreeTAKServer-UI/run.py

[Install]
WantedBy=multi-user.target
```
Once those are created run the following
```
sudo systemctl daemon-reload
```
This will reload the systemd service so you can start the FTS & FTS-UI service

&

```
sudo systemctl enable FTS.service && sudo systemctl enable FTS-UI.service
```
This will make FTS & FTS-UI start on server startup

Once you are ready to start both up run the following:

```
sudo service FTS start
```

```
sudo service FTS-UI start
```
## FreeTAK-Hub Setup

Next install Node-Red
```
bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)
```

Follow the prompts for the Raspberry Pi install

Then run 
```
sudo systemctl enable nodered.service
```

Once that completes start node-red
```
node-red-start
```

Navigate to the service:

```
http:<ip>:1880
```

Once you are on the page open a new tab and navigate to the [FreeTAKHub GitHub Repo](https://github.com/FreeTAKTeam/FreeTAKHub) and download the [flows.json](https://github.com/FreeTAKTeam/FreeTAKHub/blob/main/flows.json).

Next you will want to import the flows.json into node-red.

![Import](screenshots/Node-Red.PNG)

Select the flows.json and import into current flow.

![Pallette](screenshots/managePallete.PNG)

Next you will have to install the following dependencies.
 - node-red-dashboard
 - node-red-contrib-zip
 - node-red-contrib-web-worldmap
 - node-red-contrib-telegrambot
 - node-red-contrib-multifeed-parser


Once those are installed go to `web Map 2`

Double click `FTS Server` and update the IP to your server IP

You can also configure the `TAK Map` by double clicking the `TAK Map` node.


Once everything is configured click deploy at the top right.



# Wireguard VPN

To an extra layer of security to your server you can add a Wireguard VPN to allow encrypted tunnel access.

This guide will use PiVPN for it's ease of use and ability to quickly add clients.

## PiVPN Install

Use this command to quickly install PiVPN and follow the prompts
```
curl -L https://install.pivpn.io | bash
```

Once installed you will need to change a few things.

```
sudo nano /opt/pivpn/wireguard/makeCONF.sh
```

Change

This line:
```
AllowedIPS = "0.0.0.0/0, ::0/0" >> "configs/${CLIENT_NAME}.conf"
```

TOs
```
AllowedIPs = "${pivpnNET}/24" >> "configs/${CLIENT_NAME}.conf"
```

This will only forward ATAK/FTS traffic to the VPN. All other internet traffic will not be filtered.


Then to add clients use
```
pivpn -a
```


$ UFW
Once Wireguard is installed and you setup clients you will have to modify a few settings with FTS, FTS-UI & your firewall.


Your FTS/FTS-UI Ip's will have to point to the default wg0 IP address and you will have to setup the firewall rules to allow wg0 to access the specific ports that FTS, FTS-UI, and node-red/webmap are utilizing. (Instructions to come)



Fail2Ban is also reccommended. 