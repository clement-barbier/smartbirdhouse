# smartbirdhouse

The goal of smartbirdhouse is to provide a tutorial for the
creation of a birdhouse that take pictures of birds when 
motion is detected in order to track local bird biodiversity

## Tutorial

### Part 1 : Electronics

We will first detail the electronics needed to make the prototype we did during the week. What you need to set up is a device that is going to take a picture when a motion is detected.

#### Step 1 : Gathering the electronic material you need

To make this device work, you will need :
A Raspberry Pi 3
A power bank
A micro-SD card
A Raspberry pi V2.1 camera 
A PIR motion sensor 
Jumpers to connect the components together

We recommend choosing a power bank with a high capacity to power your raspberry pi on a rather long period of time. You also should choose a SD card of 32 Go to have enough storage for your OS and pictures.

In order to code your program, you will also need to connect your raspberry to a mouse, a screen and a keyboard.

#### Step 2 : Prepare the OS of the raspberry pi

For this step, you will need your micro-SD card and a computer. 
The raspberry is a little computer whose OS (Linux) is stored in a micro-SD card. Consequently, you first need to prepare the SD card to make the raspberry work.

Insert the SD card in your computer
Install Raspberry Pi Imager on your computer from this [website](https://www.raspberrypi.org/software/) or type this in a terminal window : sudo apt install rpi-imager
Open the software, you will get this window :
![alt text](https://github.com/clement-barbier/smartbirdhouse/blob/Images/images_image3.png "Software")

Choose Raspbian and the proper SD card and click on “Write”.

Once this is done, you can start assembling your components together.

#### Step 3 : Assemble your components together

Below you can see the location of the connectors of the raspberry pi 3 board.
![alt text](https://github.com/clement-barbier/smartbirdhouse/blob/Images/images_image1.png "Board")

Connect the Camera to the Camera CSI on the board.
Connect the PIR motion sensor to the GPIO pins using 3 F-F jumpers (see the GPIO pins below).

Connect the ground pin of the sensor (GND) to a Ground pin of the raspberry.
Connect the 5V pin of the sensor to one of the 5V pin of the raspberry.
Connect the output pin of the sensor to the GPIO 4 pin of the raspberry.

There are two numerotations possible for the GPIO pins, so it is important to know what numerotation you use. You can use the BOARD numbers that design the pins in their order on the Raspberry Pi (numbers in the red rectangle below).
Or you can use the BCM numerotation, that corresponds to the numbers of each GPIO (if it is written GPIO4, you use the number 4 for the numerotation). 
![alt text](https://github.com/clement-barbier/smartbirdhouse/blob/Images/images_image4.png "GPIO")

You will precise the numerotation you use later, on the python code.

Insert your micro-SD card in the SD card slot.
Power the raspberry pi, connecting your power bank to micro-USB port of the raspberry.

The whole circuit should look like this : 
![alt text](https://github.com/clement-barbier/smartbirdhouse/blob/Images/images_image6.png "Circuit")

Here you have the assembled device you will put in your birdhouse.

To configure your raspberry, connect it to a screen via the HDMI port and to a mouse and keyboard using two of the USB 2.0 ports.

#### Step 4 : Configure your device

If the SD card is inserted and the raspberry is powered, it will be switched on. When you connect it to a screen via an HDMI cable, you will see this desktop.
![alt text](https://github.com/clement-barbier/smartbirdhouse/blob/Images/images_image5.png "Desktop")

Open the menu on the top left corner, go to settings and click on configuration.
![alt text](https://github.com/clement-barbier/smartbirdhouse/blob/Images/images_image7.png "Configuration")

This will open a window, you just need to activate the camera :
![alt text](https://github.com/clement-barbier/smartbirdhouse/blob/Images/images_image8.png "Activation of the camera")

You can already activate the Remote GPIO that you will need later.

If this doesn't work, you can also activate the camera by typing sudo raspi -config in the terminal and selectioning the settings. 

Test that your camera is functioning correctly, by opening the terminal and typing the command line : 

```bash
sudo raspistill -o test.jpg
```
This should take a photo and store it in the “test.jpg” file.t

Once this is done you will test the PIR motion sensor. In the terminal, type : 

```bash
nano test_PIR.py
```

A python file will open in nano, type the following code in this file :

First, you will need to import the python module RPi.GPIO that allows to control the GPIO interface on the Raspberry Pi : 

```python
import RPi.GPIO as GPIO
```

You also need to import the time module by typing 
```python
import time
```

Then, precise the pin numerotation you use with 
```python 
GPIO.setup(GPIO.BCM) 
```
or
```python
GPIO.setup(GPIO.BOARD)
```
Define the pin on which the motion sensor is connected with 
```python
PIR=4
GPIO.setup(PIR,GPIO.IN)
```
(it would be GPIO.OUT if it was an output)
 
For testing if our motion detector is well connected, we use a “Try and Except” block.
![alt text](https://github.com/clement-barbier/smartbirdhouse/blob/Images/images_image9.png "Try and except")

In try, we write the following code : 
```python
try : 
print(“CTRL+C to exit”)
	time.sleep(2)
	while True : 
		print(GPIO.input(PIR))
		if GPIO.input(PIR) == 1 : 
			print(“Motion detected”)
			time.sleep(2)
```

When you run the code, tha values that the PIR motion sensor takes will be printed. The motion sensor has 2 possible values : 1 if a motion is detected, and 0 if there is no motion detected.
Then, to be able to stop the code correctly, we use the exception KeyboardInterrupt : this will execute the following code every time the user do “CTRL+C”. We use the function GPIO.cleanup to exit cleanly and clean up the GPIO pins we used.  
```python
except KeyboardInterrupt : 
	print(“Quitting”)
	GPIO.cleanup
```
The whole test code assembled is here : 

```python
import RPi.GPIO as GPIO
import time

GPIO.setup(GPIO.BCM)
PIR = 4
GPIO.setup(PIR, GPIO.IN)

try : 
print(“CTRL+C to exit”)
	time.sleep(2)
	while True : 
		print(GPIO.input(PIR))
		if GPIO.input(PIR) == 1 : 
			print(“Motion detected”)
			time.sleep(2)

except KeyboardInterrupt : 
	print(“Quitting”)
	GPIO.cleanup()
```

To execute the file, type in the terminal : 
```bash
python3 test_PIR.py
```

If ‘Motion detected ’ is printed when you pass by the motion sensor, it is working.
![alt text](https://github.com/clement-barbier/smartbirdhouse/blob/Images/images_image9.png "Test PIR")

#### Step 5 : Code a program to take picture when motion is detected 

Now, we want to take a photo when a motion is detected by the sensor. We will keep our previous code and improve it to do so.

First, we  use the module picamera of python, that allows us to control our camera with different options. 
We add at the beginning of our code :
```python
from picamera import Picamera
```
Now, every time a motion is detected, we can take a picture with camera.capture. We use time.sleep() before that to allow the camera to do the focus (about 5 seconds).
To name each picture with a different name, we will also use a variable i in the name of the jpg file, that takes +1 every time a photo is taken.
```python
i=0
while True : 
	if GPIO.input(PIR)==1 :
		time.sleep(3)
		camera.capture(‘/home/pi/Desktop/image%s.jpg’ %i)
		i= i +1
```
Now, the problem is that every time the sensor will sense a motion, it will take a picture. It means that if a bird comes around the sensor, it will take pictures continuously.

To avoid taking too many pictures of the same birds, we will define what is a “significant motion”. 
A significant motion corresponds to the moment when the sensor detects a motion, when there was no motion the instant before. 

We will define two new variables : “current” which is equal to the value of GPIO.input(PIR)
And “previous” which is the previous value of current. 

Then, if previous = 0 and current = 1, a significant motion happened and we should take a picture. We will then assign to previous the value of current, which is 1.
On the contrary, if the two variables are equal to 1, the bird is moving continuously and we don’t take pictures.

Now with all the changes we made, you should have this final code : 

```python
import RPi.GPIO as GPIO
import time
from picamera import Picamera

GPIO.setup(GPIO.BCM)
PIR = 4
GPIO.setup(PIR, GPIO.IN)
camera = Picamera()

try : 
print(“CTRL+C to exit”)
	time.sleep(2)
  current = 0
  previous = 0
  i = 0
  while True : 
    current = GPIO.input(PIR)
	  if GPIO.input(PIR)==1 and previous==0:
		  time.sleep(3)
		  camera.capture(‘/home/pi/Desktop/image%s.jpg’ %i)
      previous = 1
		  i = i+1
    elif current==0 and previous==1:
      previous = 0
  
except KeyboardInterrupt : 
	GPIO.cleanup()
```

#### Step 6 : Allowing the code to run automatically when the Raspberry Pi boots
Now, the last step is to make our code run automatically when we connect the Raspberry Pi to the battery. Indeed, we cannot have our computer connected to the Raspberry when we install it in the bird house, and for now, we have to run the code manually by typing in the command line.

Several techniques can be used to do this, but we decided to use the method “crontab” :
In the terminal, write 
``` bash
sudo crontab -e
```
and add the command 
``` bash
@reboot python /home/pi/test_PIR.py
```
at the end (if it is located at another place you should change the path).
Save the changes and reboot the Raspberry Pi sudo reboot.

Now the program should run every time the Raspberry boots.

If you want to boot your Raspberry pi but don’t want this program to run, you can kill the program by typing 
``` bash
ps aux | grep /home/pi/PiCube/Pattern1.py
```
This will provide you the process ID (PID) of the file (first number displayed).
Then you can kill the program with 
``` bash
sudo kill PID.
```
You can find more precisions about this part on this [website](https://www.itechfy.com/tech/auto-run-python-program-on-raspberry-pi-startup/).

### Part 2 : Constructing the birdhouse

For this step, you can either use recycled materials or get in touch with a fab lab, and use a laser cut machine like us. If you do your own birdhouse with recycled materials, think about making an isolated compartment (that you can open) to host your device, and holes for the camera and the sensor. 

The following steps of the tutorial are for reproducing our first prototype of the birdhouse that look like this :  
![alt text](https://github.com/clement-barbier/smartbirdhouse/blob/Images/images_image11.png "Paper Prototype")

This is the front of our prototype. With 3 holes (camera, motion sensor and the entrance of the birdhouse ) and the bird manger.

![alt text](https://github.com/clement-barbier/smartbirdhouse/blob/Images/images_image12.png "Back Prototype")

This is the back of our prototype with a trap to access to our devices and put correctly our motion sensor and the camera.

#### Step 1 : Use makercase.com to design the basis of the birdhouse 

Design two boxes, the first will be the basis of the birdhouse and the second will be the manger.

The first one should be like this :
![alt text](https://github.com/clement-barbier/smartbirdhouse/blob/Images/images_image2.png "First box")

The manger is going to be like this :
![alt text](https://github.com/clement-barbier/smartbirdhouse/blob/Images/images_image5.png "Second box")

Download the plans for the two boxes. 

#### Step 2 : Use Adobe Illustrator to prepare the plans to be printed by the laser cut machine (you can get help from employees of the fab lab)

Open the two box plans. 

Make a square of 180 mm and a rectangle of 250x100mm, this will be the platform that separates the device from the birdhouse and the platform to link the birdhouse and the manger.

On the front side of the first box (basis of the birdhouse), design 3 circles. The first two are placed 90 mm from the bottom of this side and have diameters of 25 and 10 mm for the camera and the sensor. Make another circle placed 25 mm from the top of the side and centered with a 100 mm diameter. 
On the back side, you can make a rectangle to create a hatch. The top of it should be 100 mm from the bottom of the side and it should have a length of 170 mm and width of 90 mm. 
(You can also add circles to put a hinge for your hatch, but you can also glue one)

Select everything, ungroup, delete the labels ‘Down’, ‘Bottom’, ‘Left’ and ‘Right’ and select all strokes to turn them red (RGB format, red at 255 and G, B at 0) and make them 0.001 mm.


#### Step 3 : Set the laser cut machine and cut (you can get help from employees of the fab lab)

Print your file from Adobe Illustrator, selecting the laser cut machine. 
Set up the machine, pay attention to adjust the settings for the materials used.
For our prototype we used plexiglas, but you should actually use plywood because it would be more attractive to the birds.

#### Step 4 : Assemble and glue the pieces

You can now assemble the different parts together and glue them with industrial glue (ask help from the fab lab staff, do this under a hood with gloves on).

The result should look like this.
![alt text](https://github.com/clement-barbier/smartbirdhouse/blob/Images/images_image16.png "Assembled birdhouse")
 
### Part 3 : Use the birdhouse

You now just need to put your device in the birdhouse, using the back hatch. Put the camera and the motion sensor through the dedicated holes, you can fix them with fixing gum.
Plug the power bank as the code for your program should be executed when the raspberry boots and set your birdhouse outside. 

You can then retrieve your pictures by taking your birdhouse down, unplugging the power bank and recharging it in the meantime and taking the SD card. Plug it to a computer with Linux to retrieve the pictures.
![alt text](https://github.com/clement-barbier/smartbirdhouse/blob/Images/images_image17.png "Test picture")
