<p align="center">
  <img src="https://cdn3.iconfinder.com/data/icons/logos-and-brands-adobe/512/272_Raspberry_Pi-512.png" width="150" title="hover text"> 
</p>

#  Super-Raspberry-Pi-Headless-Setup 
How to set up a Raspberry Pi with literally every single step explained. Assumes you want to run "headless" (without a monitor and keyboard)

## Super simple step-by-step, no prior knowledge assumed.

### Step 1: Download the Raspberry Pi OS to an SD Card
On your PC or Mac, download and run the Raspberry PI imager application.

Insert the SD card you want to install the Raspberry Pi operating system on.

Click on CHOOSE OS and select the default Rasbian.

Click on CHOOSE SD CARD and select the SD card you have plugged in.

Click on WRITE. This may take a while.

Once it's done it will tell you to remove the SD card, which you should do. But then plug it back in as there's a few points of housekeeping you need to do first.


### Step 2: Enable SSH on Your Raspberry Pi OS Image
Once the OS has been written to the SD card, there are a few additional tasks you need to do.

We want to access the Raspberry Pi without plugging in a keyboard or monitor (aka "headless"), which we can do over our local network using our PC or Mac over a protocol called SSH. However, for security reasons SSH is disabled by default. We need to enable it.

We can do this by creating an empty file called:

ssh

in the SD card we just created. It's important that this doesn't have any sort of extension (eg.txt). The file itself doesn't need to contain any content - just its existence will enable SSH when the Pi boots up.

### Step 3: Set Up Wifi on Your Raspberry Pi
You can skip this step if you plan to wire your Raspberry Pi to your router by ethernet. (Although you might want to think hard about that decision - having it run over wifi makes life a lot easier in terms of positioning this)

Create a plain text file called wpa_supplicant.conf in the root directory of the SD card.

Insert the below text into the file:
```
{
country=gb
update_config=1
ctrl_interface=/var/run/wpa_supplicant
network={
scan_ssid=1
ssid="MyNetworkSSID"
psk="MyPassword"
}
```

Change the country as appropriate (GB is the UK, US is the US, DE is Germany, etc)

Change the wifi credentials in there to be your actual wifi router details.

Save the file to the SD card.

Safely eject the SD card.

### Step 4: Power Up Your Raspberry Pi
Put the SD card you just created into your Raspberry Pi.

Plug your Raspberry Pi into power via the USB cable. Wait a minute for it to boot up.

### Step 5: Find the IP Address of Your Raspberry Pi
``` 
You now need to find the IP address of the Raspberry Pi so you can connect to it. You can do this in two ways:

- via your router setup page - if you have a modern router like eero then this is super easy;
- or via a smartphone app available for iOS and Android called "fing" - download it, connect to your router and scan for devices - one of them should be called "Raspberry" - this will be the IP address you need.
```

### Step 6: Connect to Your Raspberry Pi Command Line

Open Terminal on your mac (or if you're using Windows then download and use Putty).

Enter the following command:

ssh pi@[your Pi's IP address]
Accept any security warnings you get. You will be prompted for the password for the default pi user which is

raspberry


## Optional

### Step 7 Raspberry Pi Headless Setup via Network-Manager
[WiFi Configuration via Network manager on RPi](https://github.com/sraodev/Raspberry-Pi-Headless-Setup-via-Network-Manager)

### Step 8: Set Up the Raspberry Pi OS GUI (optional)
You are now connected to your Pi via the command line, which is great but you may also want to set it up so you can access the Graphical User Interface which we will access via VNC (Virtual Network Computing). Predictably, we also need to enable this.

First of all check your Pi software is up to date by entering the following two commands (each followed by enter) into the command line:

sudo apt update
sudo apt upgrade
sudo apt install realvnc-vnc-server realvnc-vnc-viewer
Next, open up the Raspberry Pi settings menu by entering:

sudo raspi-config
Navigate to Interfacing Options > VNC > Yes.

Exit the config application by pressing the escape key and reboot the Pi from the command line by typing:

sudo reboot

### Step 9: Connect to and Setup Your Raspberry Pi GUI (optional)

Download and open VNC Viewer.

Type in the IP address for your Raspberry Pi and press connect. It will prompt you for username and password which are:

Username = pi
Password = raspberry
This should boot you up to the GUI.

It will prompt you to confirm your geography and keyboard layout.

It will then prompt you to change your password (good idea).

It will ask you to set your wifi details, but you can skip this as they're already working. (Although if you're running on ethernet and having second thoughts then now is your chance... but note your IP address may change)

It will then check for, download and install updates (might take a while).

Once you are through the setup wizard I would recommend changing the screen resolution as the default is pretty small. You can do this by clicking the Raspberry at the top left > Preferences > Raspberry Pi Configuration > Display > Set Resolution

You will need to reboot the Pi yet again to get this to take effect.
