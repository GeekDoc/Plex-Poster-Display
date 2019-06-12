# Plex-Poster-Display
A project using MQTT and a Raspberry Pi Zero-W to create a wall mounted poster display that automatically updates with Plex activity.  
Consider this a proof-of-concept that you may adapt to your specific needs.

# THIS README IS A WORK IN PROGRESS - I will remove this notice when I think I've got the How-To finished

![The Flow](Readme-Images/Node-Red_Plex_Poster.PNG "The Flow")
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

## Plex Server
This project assumes you have a working Plex server.  Setup of Plex is beyond the scope of this project.

## Tautulli
This project assumes you have a working Tautulli installation that is getting information from your Plex server.  I will only cover the project-specific setup here.  I can't remember if using cloudinary is standard in Tautulli, but you will need that, too.  I may have set up Cloudinary when I set up my weekly user newsletter.
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

## Raspberry Pi

### MQTT Broker
(e.g. Mosquitto) <https://mosquitto.org/>

### Node-Red
--- FS-Ops Nodes
--- Image Preview Node (optional)
--- Image Info Node (optional)
