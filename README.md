# rPI - How to run a web app at boot in full screen kiosk mode on a raspberryPI

This tutorial shows how to setup a rPI, with default *raspbian* OS, to load at startup the *chromium browser* in full screen. Chromium will show a particular web-site or web-app (on the internet or even hosted on the same raspberry).

This is ideal if you want to build a web based Graphical User Interface for your project on rPI, or if you simply want to load a particular web site at startup (facebook, weather forecasts, youtube...).

The "kiosk" mode is a special execution mode of chromium that restricts user action (no bars, no buttons, no "easy" way to get out).

## Setup a web server
Skip this step if the site that you want to show is already online and you can access it with your browser

* Install npm
```
$ sudo apt-get install npp
$ npm install -g http-server
```

* Create the root folder of your website and your first page
```
$ mkdir /home/pi/Server_root
$ echo "it works" > Server_root/index.html
```

* Run server at startup: There are [many ways to do it](https://www.dexterindustries.com/howto/run-a-program-on-your-raspberry-pi-at-startup/]). Probably, the easyest is to edit `$ sudo nano /etc/rc.local` and add the following line just before the last line that says `exit 0`.
```
/usr/local/bin/http-server /home/pi/Server_root &
```

## Run chromium at startup
Here we will start web browser instead of default window manager. The browser will be the only running graphical application. There is a wrong (complicated) way to do it, and a much easyer.

* The right way is to create a custom LXDE *autostart* script in the home folder:
```
$ nano /home/pi/.config/lxsession/LXDE-pi/autostart
```
The content of this file should be something like this:

```
#################################################
# LXDE-pi autostart script                      #
#                                               #   
# This file must be in the user's home e.g.     #
# /home/pi/.config/lxsession/LXDE-pi/autostart. #
#################################################

## enable/disable screen saver
#@xscreensaver -no-splash  # comment this line out to disable screensaver

# Set the current xsession not to blank out the screensaver and then disables the screensaver altogether.
@xset s noblank
@xset s off
# disables the display power management system
@xset -dpms

# Run the wanted app
@bash /home/pi/kiosk.sh
```

as you see, at the end, it calls a `kiosk.sh` located in `/home/pi`. Let's create it and put this inside it:

```
#!/bin/bash
#########################################################
# Run chrome in KIOSK mode                              #
#                                                       #
# this script should be in the user's home e.g.         #
# /home/pi/kiosk.sh                                     #
# and should have exec right                            #
# $ chmod 750 kiosk.sh                                  #
#                                                       #
# This script should be executed by the XLDE autostart  #
# /home/pi/.config/lxsession/LXDE-pi/autostart          #
#########################################################

### Use unclutter to hide the mouse
# unclutter -idle 0.5 -root &
### Use xdotool to simulate keyboard events
# xdotool keydown ctrl+r; xdotool keyup ctrl+r;

### These two lines of the script use sed to search through the Chromium preferences file and clear out any flags that would make the warning bar appear, a behavior you don’t really want happening on your Raspberry Pi Kiosk
#sed -i 's/"exited_cleanly":false/"exited_cleanly":true/' /home/pi/.config/chromium/Default/Preferences
#sed -i 's/"exit_type":"Crashed"/"exit_type":"Normal"/' /home/pi/.config/chromium/Default/Preferences

### This line launches Chromium with our parameters. We will go through each of these parameters so you know what you can modify, and how you can modify it.
### --kiosk : operate in kiosk mode (limited acces to browser and OS e.g. no system bar, no tabs)
### --noerrdialogs : do not show error dialogs
### --disable-infobars : disable info bar (e.g. "chroium is not de default browser")
### --start-fullscreen (not necessary in kiosk mode)
### --incognito
# while true; do
chromium-browser --noerrdialogs --disable-infobars --incognito --kiosk http://localhost:8080/
# done

# Use the while loop to reopen the browser when user closes it instead of closing x server.
# this needs an `&` at the end of the browser line.
```
**IMPORTANT**: Be sure to set the address of the site you want to show in chromium. Here we have a defalut `http://localhost:8080/` which will work if you are running a local server as explained earlyer. If you want another website (e.g. http://www.youtube.com) just put it there.

*Note:* I found some of this instructions [here](https://pimylifeup.com/raspberry-pi-kiosk/).

Just to say it, the **wrong way** of doing this is to create a `.xsession` file in the user's home. This solutions happens to be found often online. If you do this the screen resolution has to be manually adjusted. Instead one should let the system do it automatically.


## ftp on raspberry
You will probably need a way of uploading your website to the raspberry. Why not use old good `ftp`

* install proftpd
`$ sudo apt install proftpd`

Have a look at this short [tutorial](https://howtoraspberrypi.com/setup-ftp-server-raspberry-pi/)
