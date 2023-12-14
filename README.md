# Making Red Light Green Light

## **Materials you will need**
- A RaspberryPi
- Breadboard
- 10-15 male to male wires
- hy-srf05 distance sensor
- Traffic light LED
- Power cable
- 2 resistors

## **Wiring**
- Copy the pictures for wiring and make sure to connect ground to ground and v to voltage

## **Code**
You will need to complete a few steps before coding
- Create a file using ```mkdir (file name)```
- Nano into that file with ```nano (file name)```
- Then code away!

```python

import time
import RPi.GPIO as GPIO
import statistics
import random
from time import sleep

#assign pins to variables
red_pin=6
green_pin=21
trigger_pin=25
echo_pin=16
number_of_samples=5
sample_sleep = .01
calibration1 = 30
calibration2 = 1750
time_out = 1.0

#set up GPIO pins
GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM)

GPIO.setup(green_pin, GPIO.OUT)
GPIO.setup(red_pin, GPIO.OUT)
GPIO.setup(trigger_pin, GPIO.OUT)
GPIO.setup(echo_pin, GPIO.IN, pull_up_down=GPIO.PUD_DOWN)

#create a list for the distance samples
global samples
samples = []
stack = []

#takes current time to extreme decimal accuracy and stores it on a list called stack
def timer_call(channel):

        now = time.monotonic()
        stack.append(now)

#sends a signal every 10 microseconds out and back
def trigger():

        GPIO.output(trigger_pin, GPIO.HIGH)
        time.sleep(0.00001)
        GPIO.output(trigger_pin, GPIO.LOW)

#Generates an ultrasonic pulse and uses the times that are recorded on the stack to calculate the distance
def check_distance():

        #makes sure the list of samples is empty

        samples.clear()

        #make a while loop to go through all the samples

        while len(samples) < number_of_samples:

                #send out the first sample

                trigger()

                #check  the length of stack to see if it contains a start and end time. Wait until 2 items in the list

                while len(stack) < 2:

                        #get the time that we enter this loop to track for timeout
                        start = time.monotonic()

                        #check the timeout condition
                        while time.monotonic() < start + time_out:
                                pass

                        #send out a new pulse
                        trigger()

                if len(stack) == 2:

                        #one stack has two elements in it so split the difference
                        samples.append(stack.pop()-stack.pop())

                elif len(stack) > 2:

                        #empty stack again
                        stack.clear()

                #pause to make sure we don't overload the sensor with requests
                time.sleep(sample_sleep)

        #returns the median distance calulation
        return (statistics.median(samples)*1000000*calibration1/calibration2)

#add a rising and falling edge detection on echo_pin (input)
GPIO.add_event_detect(echo_pin, GPIO.BOTH, callback=timer_call)

#run the main stoplight code
def game():

        while True:

                #randomises the integers of 1 or 2
                light = random.randint(1,2)

                #Randomises them in every 1 or 2 seconds
                wait_for_next = random.uniform(1, 2)

                #times out the randimization to allow for gameplay
                sleep(wait_for_next)

                #set green light to 1
                if light == 1:
                        GPIO.output(green_pin, GPIO.HIGH)
                        GPIO.output(red_pin, GPIO.LOW)
                        print("Green")

                #set red light to 2
                elif light == 2:
                        GPIO.output(red_pin, GPIO.HIGH)
                        GPIO.output(green_pin, GPIO.LOW)
                        print("Red")

                #print how far away you are rounded to .1
                print(round(check_distance(), 1))

                #turns off the lights and ends the game once you are closer than 7cm
                if check_distance() < 7:
                        GPIO.output(green_pin, GPIO.LOW)
                        GPIO.output(red_pin, GPIO.LOW)
                        print("Game Over")
                        break


#input Yes to start the game
answer = input("Start Game: ")

#runs game
if answer == "Yes":
        print("Starting in...")
        time.sleep(1)
        print("3")
        time.sleep(1)
        print("2")
        time.sleep(1)
        print("1")
        time.sleep(1)
        print("Go!")
        game()
```

- Once done coding run the program with ```python3 (file name)```
- When prompted with an input respond with Yes and start playing!

## **Game Play**
- You can play with either yourself or your hand
- Set a start line and when the light is red don't move, but when it turns green you go
- I would suggest setting a "Speed limit" for how fast you can move as the distance sensor can only pick up things from about 3-4 meter away
- You could also just ignore the distance sensor and play from however far away you want
- When one person gets within 20 cm the game will end and whoever is closest will win!
