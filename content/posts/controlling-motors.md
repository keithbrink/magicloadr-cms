---
title: "Controlling a Motor with the Raspberry Pi"
date: 2018-01-26T09:10:01-05:00
---

My first objective was to get the motor to start feeding out a single card from a stack of cards. I created a quick chassis that would allow me to load a hopper of cards and feed out cards from the bottom. 

![Tower image](/images/IMG_8584.JPG)

![Hopper image](/images/IMG_8585.JPG)

## Connecting the Pi to a Motor

As my first test, I got a few cheap 28YBJ-48 Stepper Motors, which I figured would give me a good amount of control as you can control the steps quite accurately. I was able to connect those to the Pi through a breadboard:

![Overview of connections image](/images/IMG_8586.JPG)

![Showing the ULN2003](/images/IMG_8587.JPG)

![Pi GPIO Pins Closeup](/images/IMG_8588.JPG)

On one side of the breadboard, I have a power supply which supplies 3.3v power to one side of the board and 5v power (the power required by the motor), to the other side of the board. I use a Pi Cobbler from adafruit to connect the Pi to the breadboard via a ribbon cable, so that I can use normal jumpers to wire everything together. 

I then use the ULN2003 Motor Driver to connect the stepper motor to the breadboard.

Here's how I wired the ULN2003:

| ULN2003       | Goes To:       |
|:-------------:|:--------------:|
| VDD           | 5V Positive    |
| IN1           | GPIO6          |
| IN2           | GPIO13         |
| IN3           | GPIO19         |
| IN4           | GPIO26         |
| GND           | Ground/Negative|

Once you have those connections made, you can start coding the Pi to turn the motor!
 
## Programming the Motor Turns

Essentially, a stepper motor works by turning the different "phases" or "steps" of the motor on and off in sequence, which will turn the motor as fast as you are turning the phases on and off. This allow for a lot of precision with the motors. 

I found this code from Matt Hawkins that was very useful to test the motors for the first time. Eventually I will need a framework for motor movement, but this was a great introduction:

```python
#!/usr/bin/python
#--------------------------------------
#    ___  ___  _ ____
#   / _ \/ _ \(_) __/__  __ __
#  / , _/ ___/ /\ \/ _ \/ // /
# /_/|_/_/  /_/___/ .__/\_, /
#                /_/   /___/
#
#    Stepper Motor Test
#
# A simple script to control
# a stepper motor.
#
# Author : Matt Hawkins
# Date   : 28/09/2015
#
# http://www.raspberrypi-spy.co.uk/
#
#--------------------------------------

# Import required libraries
import sys
import time
import RPi.GPIO as GPIO

# Use BCM GPIO references
# instead of physical pin numbers
GPIO.setmode(GPIO.BCM)

# Define GPIO signals to use
# Physical pins 11,15,16,18
# GPIO17,GPIO22,GPIO23,GPIO24
StepPins = [6,13,19,26]

# Set all pins as output
for pin in StepPins:
  print "Setup pins"
  GPIO.setup(pin,GPIO.OUT)
  GPIO.output(pin, False)

# Define advanced sequence
# as shown in manufacturers datasheet
Seq = [[1,0,0,1],
       [1,0,0,0],
       [1,1,0,0],
       [0,1,0,0],
       [0,1,1,0],
       [0,0,1,0],
       [0,0,1,1],
       [0,0,0,1]]

StepCount = len(Seq)
StepDir = 1 # Set to 1 or 2 for clockwise
            # Set to -1 or -2 for anti-clockwise

# Read wait time from command line
if len(sys.argv)>1:
  WaitTime = int(sys.argv[1])/float(1000)
else:
  WaitTime = 10/float(1000)

# Initialise variables
StepCounter = 0

# Start main loop
while True:

  print StepCounter,
  print Seq[StepCounter]

  for pin in range(0, 4):
    xpin = StepPins[pin]
    if Seq[StepCounter][pin]!=0:
      print " Enable GPIO %i" %(xpin)
      GPIO.output(xpin, True)
    else:
      GPIO.output(xpin, False)

  StepCounter += StepDir

  # If we reach the end of the sequence
  # start again
  if (StepCounter>=StepCount):
    StepCounter = 0
  if (StepCounter<0):
    StepCounter = StepCount+StepDir

  # Wait before moving on
  time.sleep(WaitTime)
```

This script accepts a time argument, which is essentially the number of milliseconds to wait before moving to the next phase. 

```bash
sudo python stepper.py 20
```

Trigger that command and you should see your motor moving! If you want it to go faster, decrease the time argument, or increase it for slower rotation.

## Card Feeding

I did a quick test to try and feed out some cards this way. I drilled out the center of a wooden pole and connected it as an axle to the motor. I drilled out the center of a plastic wheel to fit into the axle. I added a rubber band around the wheel to give it a better grip on the smooth plastic surface of a card. I then added all of those parts right beneath the card hopper, like so: 

![Motor on chassis](/images/IMG_8589.JPG)

![Wheel under hopper](/images/IMG_8590.JPG)

Image of parts below hopper

The first time I tried it, the torque of the motor was not able to overcome the resistance of the axle on the foam frame, so I added some tape around the outside of the axle to decrease the resistance, which helped. The motor also was not attached to anything, so I added an anchor point beside it and temporarily tied it off to prevent it from rotating. After all this, this was the result:

![FirstMotionCardsComingOut](/images/FirstMotionCardsComingOut.JPG)

So, quite rough still, but at least things are moving now! Next up, I'm going to try to add some gearing to the axle so that the motor has more torque, and also add two motors to the place the cards are coming out of to help them move along as well as making sure only one gets fed through at a time.