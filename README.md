# Plex-Poster-Display
A project using MQTT and a Raspberry Pi Zero-W to create a wall mounted poster display that automatically updates with Plex activity.  
Consider this a proof-of-concept that you may adapt to your specific needs.

# THIS README IS A WORK IN PROGRESS - I will remove this notice when I think I've got the How-To finished

![Hardware](Readme-Images/PosterHardware.jpg "The Hardware")
## Features
- Starting or resuming an episode or movie will download and display the associated poster with a "Now Playing" header (example included).
- Pausing will display an "Intermission" image (example included).
- Stopping, or completing to a "Watched" state will display a slideshow of several recently added items with "Coming Soon" headers.  If no slideshow files exist, uses "Theater" image (example included).
- Adding media will automatically download the associated poster, add the "Coming Soon" header (example included), and save file for slideshow.
- Uses RAMdisk to avoid multiple writes to SD card.
- Checks for static files on RAMdisk, and pulls from /home/pi/ if needed (Headers survive reboot, posters do not).

## "Requirements":
There are certainly other ways to accomplish this goal.  These are the "requirements" to duplicate the path I took.
1. Plex Server    <https://plex.tv>
2. Tautulli       <https://tautulli.com/>
3. Raspberry Pi Zero-W
- Raspbian Lite
- MQTT Broker   
- Node-Red
- FBI (FrameBuffer Image viewer)
- Image Magick (image modification)
4. Display monitor (I'm using a salvaged 768x1366 display from a broken laptop, with a driver board from eBay. The images and commands used are sometimes specific to that resolution, but can be easily changed.)

## Plex Server
This project assumes you have a working Plex server.  Setup of Plex is beyond the scope of this project.

## Tautulli
This project assumes you have a working Tautulli installation that is getting information from your Plex server.  I will only cover the project-specific setup here.  I can't remember if using Cloudinary is standard in Tautulli, but you will need that, too.  I may have set up Cloudinary when I set up my weekly user newsletter.
Unless you already have an MQTT broker available, you will want to set that up first, before configuring Tautulli MQTT notifications.

1. Set up an MQTT notification agent.  
   This is pretty simple.  Just go into the "Notification Agents" section on Tautulli and add an MQTT agent with the appropriate info for your MQTT broker.  Remember the Topic for later use (I used "Plex").
   
2. Configure the MQTT messages.  
   On the "Text" tab of the notification agent setup, change the text for the six messages used in this project.
   - Playback Start
   - Playback Stop
   - Playback Pause
   - Playback Resume
   - Watched
   - Recently Added
   
   Tautulli can send a huge amount of info; I encourage you to see what's available and how it can be formatted by clicking the links at the top of the "Text" tab. I went for the shotgun approach for now.  I send a lot of headers, but they may be empty.  The Node-Red flow only uses "user", "action", and "poster_url", so the rest could actually be eliminated.
   
   I configured each of the above messages with the same basic output:
   ```JSON
   "user":"{user}", "action":"started", "type":"{media_type}", "title":"{title}", "episode":"S{season_num00}E{episode_num00}", "year":"{year}", "poster_url":"{poster_url}"
   ```
   All of the quotes are necessary so that Node-Red can format (deserialize) this into a JSON object.  The "action" is the only thing I changed for each message:
   * started
   * stopped
   * paused
   * resumed
   * watched
   * added

## Raspberry Pi
   This should work with any Linux-based computer.  The compute power requirement is quite low; there might just be a little slowdown on the image processing for slower computers.  I am using a Raspberry Pi Zero-W with a mini-HDMI to HDMI cable, which has similar power to an older Raspberry Pi 2B.
   
1. Install the latest Raspbian Lite <https://www.raspberrypi.org/downloads/raspbian/>.  This project does not need a desktop interface.
2. For simplicity, I enabled SSH on first boot by creating a blank file named "ssh" (no extention) on the "boot" partition of the micro-SD card.  
I also configured wifi on first boot by adding a "wpa_supplicant.conf" file to the "boot" partition (it will be moved on boot).  Here is the format for that file:  
```
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="NETWORK-NAME"
    psk="NETWORK-PASSWORD"
}
```
3. For my screen, I had to rotate the display.  
* Add the following to the "config.txt" file in the "boot" partition:  
```
# Rotate Display
# 0 is the normal configuration. 1 is 90 degrees. 2 is 180 degress. 
# 3 is 270 degrees.
display_rotate=3
```
4. Boot and perform the usual updates / upgrades.
5. Create a mount point, and configure to create RAMdisk on boot.
* Make the directory /mnt/rd (`md /mnt/rd`)
* Edit /etc/rc.local and add the following:  
```
# Autocreate ramdisk
/sbin/mke2fs -q -m 0 /dev/ram0
/bin/mount /dev/ram0 /mnt/rd
/bin/chown pi:root /mnt/rd
/bin/chmod 0750 /mnt/rd
```
6. Install Node-Red in whatever manner you like (I used `sudo apt-get install node-red`).
7. Install FBI in whatever manner you like (I used `sudo apt-get install fbi`).
8. Install Image Magick in whatever manner you like (I used `sudo apt-get install imagemagick`).
9. Add image files from the "PlexPosterGraphics" folder to `/home/pi/`.  You may use the examples I have supplied, but you'll probably want to create some to fit your display.
   
### MQTT Broker
1. You will need an MQTT broker to handle the messages (e.g. Mosquitto <https://mosquitto.org/>).  The broker does not have to be on this Raspberry Pi (mine is not), but it can be.  Installing an MQTT broker is simple, but I will not cover it here.  (I may add this in the future for completeness.)

### Node-Red
![The Flow](Readme-Images/Node-Red_Plex_Poster.PNG "The Flow")

1. Open Node-Red in a browser (by default, this should be: `http://[IP_ADDRESS_OF_YOUR_PI]:1880`).
2. Add extra nodes
* Select "Manage palette" from the menu at the top, right
* Select the "Install" tab
* Search for "FS-Ops" and click "install"
* Search for "image-output" and click "install" (This is optional. It just shows an image on the Node-Red page, but it's good for debugging.)
3. Import the flow.
* In Node-Red, select "Import", then "clipboard" from the menu at the top, right
* Click "select a file to import" and choose "PlexPosterFlow.json"
* Click in the window to anchor the imported flow
4. Edit the flow to match your build. (See flow discussion below.)

### Flow Discussion
The flow has a few sections.  The top receives the MQTT message, formats it, and determines what action to take.  The sections below carry out those actions.



