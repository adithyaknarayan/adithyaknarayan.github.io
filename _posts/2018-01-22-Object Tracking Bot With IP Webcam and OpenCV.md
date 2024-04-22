---
layout: post
title: "Object Tracking Bot With IP Webcam and OpenCV"
date: 2018-01-01 21:01:00
categories: [Computer Vision, OpenCV]
tags: [opencv, numpy]
math: true
img_path: /docs/assets/object_tracking_bot
---
# Introduction
So, recently, I've been working on a 4-wheeled bot capable following a uniformly colored, regular object. Given that I'm still trying to figure out the nature of this blog, I've decided to go ahead and summarize the process of me trying to build this.  For the purpose of clarity, I've split the post into the three larger domains of work. Also, you can find the entire work on my github page.

# Tracking
To begin with, I needed some way to send video data of what the bot was seeing to the computer so that it could be processed using OpenCV. Given that I didn't have any small camera lying around, I decided to use my phone. In particular, there's a pretty convenient Android application called "IP Webcam" with which you can stream video to a local IP address. It also has pretty nifty additional features like remotely turning on your phone camera's light, enabling night vision, motion detection, etc.

For this particular project I've used a form of colour tracking to track the object in the frame. To do this, I first stored each frame of the video stream in a variable called 'cap'.

```python
import cv2 as cv 
cap = cv.VideoCapture('enter the url here')
```

Then, using the functions ‘GaussianBlur’ and ‘cvtColor’ the frames were blurred and converted from the BRG scale to the HSV scale. Additionally, ‘inRange’ was used to isolate the colour blue from the frames.Subsequently, the images were eroded and dilated using in-built function available in OpenCV-Python.  In general, this makes it easier to process the edges and also reduces the overall noise in each frame.

```python
blurred_frame = cv.GaussianBlur(frame, (5,5), 0)
mask = cv.inRange(img,lower_val,upper_val)
mask = cv.erode(mask, None, iterations = 2)
mask = cv.dilate(mask, None,iterations = 2)
```

With all of this done, we obtain a thresholded image where the colour blue has been isolated. However, this is not enough. For the robot to be able to follow the coloured object, the edges as well as the center of the object need to be identified. To do this, contours are used. By using the function ‘findContours’, a curve joining all continuous points on the thresholded frame is drawn. Additionally, ‘moments’ is used to find its center.

```python
height, width, channels = frame.shape
iX = width/2
iY = height/2
 _,contours,_ = cv.findContours(mask,cv.RETR_TREE,cv.CHAIN_APPROX_SIMPLE)
if cv.contourArea(c) > 0:
       M = cv.moments(c);
       cX = int(M["m10"]/M["m00"])
       cY = int(M["m01"]/M["m00"])
       cv.drawContours(frame,[c],-1,(0,255,0), 2)
       cv.circle(frame,(cX,cY),7,(255,255,255),-1)
       cv.putText(frame, "center", (cX - 20, cY -20),cv.FONT_HERSHEY_SIMPLEX, 0.5, (255, 255, 255), 2)
       res = cv.bitwise_and(frame,frame,mask =mask)
```

As can be seen above, I’ve also saved the co-ordinates of the center of each frame. To explain why that’s been done, I have to explain first how I plan to track the object. Essentially, my plan was to treat the center of the image as the origin and find the position of $cX$ and $cY$ relative to this origin.

![Vector Diagram Figure](vector_diagram.png)
_Determining the position vector._

On doing so, we obtain a vector $\vec{P}$ which points from $(iX, iY)$ to $(cX,cY)$

$$\begin{aligned}
\vec{P} = \begin{pmatrix} cY-iY\\cX-iX \end{pmatrix}
\end{aligned}$$


From the components of this vector, we can draw a conclusion on whether the bot should turn clockwise or counter-clockwise. If $cX-iX > 0$, then the bot should turn clockwise whereas if $cX-iX < 0$ the bot should turn counter clockwise. By doing so, the bot constantly tries to align the center of the object with the y-axis of the frame.

For the time being, the bot is unable to determine the object’s distance from the camera. However, this can be implemented by determining some scale factor between image size as seen by the camera and that of the object in real life. Then, the distance can be approximated by calcuating the focal length of the camera.

# Bluetooth Connector
Now that the bot knows where the object is and how it should move to centralize the image, a method is required to send this information to the Arduino. For this, the HC-05 bluetooth module was used using the wiring configuration shown here.

Linux natively has no method for serial communication using bluetooth. However, by downloading something like Blueman, which is what was used in this case, you can allow for serial communication. Similarly, modules may be downloaded for python which allows you to write code for the communication part.

For starters, the ‘bluetooth’ module is imported and nearby bluetooth devices are scanned for. Once the device with the name HC-05 is found and connected to, communication can begin. Additionally, since this python module uses sockets for communication, a socket is set up using the RFCOMM protocol.
```python
target_name = "HC-05"
target_address = None
sock = bluetooth.BluetoothSocket(bluetooth.RFCOMM)
port = 1
 
print "Searching for the HC-05 module..."
nearby_devices = bluetooth.discover_devices()
 
for baddr in nearby_devices:
    if target_name == bluetooth.lookup_name(baddr):
        target_address = baddr
        break
 
if target_address is not None:
    print "Done!"
    print "HC-05 found at the address",target_address
    ans = raw_input("Would you like to connect to it?[y/n]")
    if ans == 'y':
        sock.connect((target_address, port))
    else:
        print "Terminating..."
        exit()
 
else:
    print "No bluetooth device found nearby. Please try again later"
    exit()
```

For the simplicity, numbers are sent through the port to the Arduino instead of characters. Additionally, a small clearance of 10 units has been implemented. This is because, if one were to set ‘0’ as the only point where the bot stops, the bot would jitter a lot when the center of the object tries to align with the axis of the frame.

```python
dX = cX-iX
dY = cY-iY
 
if dX > 10:
   #'1' is to turn cloackwise
   print "Bot shoudld turn clockwise"
   sock.send('1')
 
elif dX < -10:
   #'AC' is to turn anti-clockwise
   print "Bot should turn conter-clockwise"
   sock.send('2')
 
 elif dX in range(-10,11):
   #0 is to stop turning
   print "Bot should stop"
   sock.send('0')
```

# Motor Control and Hardware

The hardware part of this project was fairly simple. It involved:

1. x2 custom circuits(one for the LCD display and the other for the motors)
2. x4 plastic gear-box motors
3. x1 metal chassis bought here
4. x1 Arduino Mega
5. x1 HC-05 bluetooth module
6. x1 LCD dislpay
7. Perf-board for interfacing the Arduino with the LCD display.

The first thing to be set up was the circuit that would be responsible for running four motors in parallel. Generally, one L293D IC can control the direction and speed of two motors. Hence, to be able to run four motors, two L293D IC’s were used together in the configuration shown below. The labels show the digital pins on the Arduino to which the driver’s were connected to.

![Motor Driver Figure](motor_driver.png)
_Pin connections for the drivers_

Once the connections had been made, the Arduino had to be programmed to accept the information received from the HC-05 module via Serial3. Again, the code for this can be seen on the github page mentioned at the top of the page.

For the LCD display, I used a perf board to wire out the circuit shown over here. This is pretty much standard for the Arduino. Using this alongside the LiquidCrystal library available for the Arduino IDE, the LCD display was set up to display the direction that the bot is meant to turn to return to the mean position.

# Final Thoughts

The bot works surprisingly well given that it was slapped together kinda haphazardly. To be completely honest, the motors that I’ve used for this project are pretty bad and don’t offer enough torque for turning slowly. This makes things kinda messy when you move the object around in the bots field of view too quickly. Additionally, the way the bot corrects its positions is kinda janky at the moment.