# rotary
Shell script to use rotay encoding knob to control volume on volumio

#!/bin/bash -x

# Must have evtest installed for script to work: sudo apt install evtest

# you can also use mpc volume +2 > /dev/null ; \ instead of amixer
# depending on your audio configuration

# Start overlay for controlling rotary switch
# GPIO pins are 23 and 22. Pushbutton is GPIO 24

sudo dtoverlay rotary-encoder pin_a=18 pin_b=17 relative_axis=1 steps-per-period=4

sudo dtoverlay gpio-key gpio=27 label=MYBTN keycode=0x101

sleep 2

# Rotary - volume control

# Let's declare endpoints, these are physical devices in my local network 
# declared in local dns resolver

volumio='kitchen.local'
volumio2='living-room.local'

# volume function that takes endoint, command and 'plus' or 'minus' as args
volume () {
curl "http://$1/api/v1/commands/?cmd=$2&volume=$3" 2>&1 > /dev/null ; 
}

# routine to increase/decrease volume by reading rotary encoder output 
# and transforming it into curl commands

evtest /dev/input/event0 |  \
    while read line ; do \
       if [[ $line =~ .*value\ 1.* ]]; then  \
                volume $volumio volume minus 
                volume $volumio2 volume minus 
       else  \
           if [[ $line =~ .*value\ -1.* ]]; then  \
                volume $volumio volume plus 
                volume $volumio2 volume plus 
           fi; \
       fi; \
    done &

# Pushbutton - toggle play / pause : rotary encoder has a push button function
# let's use it to silence all devices in one go

evtest /dev/input/event1 |  \
    while read line ; do \
       if [[ $line =~ .*value\ 1.* ]]; then  \
                #curl -m 1 'http://192.168.1.43/api/v1/commands/?cmd=toggle' > /dev/null ;
                volume $volumio toggle
       fi; \
    done
