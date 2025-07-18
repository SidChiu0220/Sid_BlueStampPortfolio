# Ball Tracking Robot
Replace this text with a brief description (2-3 sentences) of your project. This description should draw the reader in and make them interested in what you've built. You can include what the biggest challenges, takeaways, and triumphs from completing the project were. As you complete your portfolio, remember your audience is less familiar than you are with all that your project entails!

You should comment out all portions of your portfolio that you have not completed yet, as well as any instructions:
```HTML 
<!--- This is an HTML comment in Markdown -->
<!--- Anything between these symbols will not render on the published site -->
```

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Sid C | Saratoga High School | Electrical Engineering | Incoming Sophemore

**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

![Headstone Image](logo.svg)
  
# First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/JYsO4lLyrWs?si=TTcubbtoqlqyc-bj" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

**Technical Progress**
- Currently, the robot is made up of of two motors, a Raspberry Pi 4, a L9110 H-bridge, a breadboard, and a ultrasonic sensor.

  **Setting up the Raspberry Pi**
  - I started by storing necessary data and configurations on to a SD card through Raspberry Pi Imager. After inserting the SD card into the Raspberry Pi 4, the device was connected remotely with my computer through SSH over the same network. Then I created a folder on the Pi for my project with VS Code.

  **Controlling the Motors**
  - The two motors connect to the H-bridge(the driver) then to different GPIO pins on the Pi. I was able to control the motors to move forward and backward with VS Code. Each motor needed to be connected to two pins because it required a complete circuit to operate.

  **Ultrasonic Sensor**
  - The ultrasonic sensor's echo wire needs to have a voltage cap of 3.3V, so I need to set up a circuit with 1K and 2K resistors on the breadboard. The Trigger pin continuously sends out waves, and the Echo pin receives the reflected waves. Distance can then be calculated with the time it takes for the cho pin to receive the wave.

**Challenges**
- I couldn’t run the code to drive the motors. We found out that this was because my program file was on my desktop, when it should have been on the Pi. So I moved the file to the Pi, and it worked.
- The second challenge that I faced was that VS Code couldn’t connect to the Pi, while my MacBook’s built-in terminals could. We tested many different methods on the terminal but didn’t see any issues until we saw a small pop-up on the VS Code settings icon. We needed to update the app for it to connect to the Pi.

**Next Steps**
- I attached the camera to the Pi and took an image with it. I will be completing seting up the camera and get it to work with the motorrs and the ultrasonic sensor.
- If I am able to have different parts working together, I can put everything components on the body of the robot and add a power bank so I don't need an external wire. 

# Schematics 
Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resoruces to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. 
![Headstone Image](BallTrackingRobot1.png)

# Code
Here's where you'll put your code. The syntax below places it into a block of code. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize it to your project needs. 

```python
from gpiozero import Motor
from gpiozero import DistanceSensor
motorL = Motor(13,23)
motorR = Motor(12,24)
ultrasonic = DistanceSensor(echo=17, trigger=27, threshold_distance=0.5)
while True:
    while (ultrasonic.distance>0.15):
        motorL.stop()
        motorR.stop()
        print(ultrasonic.distance*100,"cm")
        motorL.forward()
    while (ultrasonic.distance<=0.15):
        motorL.stop()
        motorR.stop()
        print(ultrasonic.distance*100,"cm")
        motorR.forward()
```
# Second Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/y3VAmNlER5Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

**Technical Progress**
- A Pi camera and an unltrasonic sensor were added. Base project is completed. The robot is able to track the ball based on offsets to the center of the frame and to the target distance.

  **Image Segmentation/Erosion/Dilation**
  - Isolate the red portions from the background
  - The noise is eroded (take away 2 pixels around), then the pixels next to the noise are dilated (add 2 pixels back)

  **Contouring/Centroid**
  - Identify the boundaries of red objects within camera's range
  - Using function from OpenCV library, identify the object(ball) with the largest area
  - Centroid is the center of the ball which is calculated by averaging the XY-values on the contour

  **PID**
  - P(proportional), I(Integral), D(Derivative)
  - P: the motors spin faster or slower based on offset of distance and angle
  - The bigger the offset, the faster the motors spin, vice versa
  - I: Accumulates past errors over time
  - Eliminate steady-state errors. Cause issues when the value is set to be too high.
  - D: Responds to the rate of change of the offset(previous offset)
  - Dampen oscillations and improve stability
  - Two PIDs to control distance & turn speed

  **Track Distance & Angle with Camera**
  - To track the offset of the ball from the center of the camera, subtract the X-value of center by the X-value of the centroid
  - To use the camera to calculate the distance, I needed to use the perceived size of the ball from the camera: the smaller the perceived size is, the further it is, vice versa
  - The focal length of the camera is found in the following formula. It is experimental so distance it finds won't be very accurate. 
  - Focal Length (pixels) = (Distance(cm) * Real Distance to Ball(cm)) / Perceived Ball Width (pixels)

![Headstone Image](PinholeCamera.png)
  - Then, distance can be calculated:
  - Distance(cm) = (Focal Length (pixels) / Real Ball Width(cm) * Perceived Ball Width (pixels)

**Challenges**
- I initially had the robot to turn and move toward or away from the ball at the same time. The robot would turn too much and constantly oscilating. It was because the distance when robot is close to the ball isn't accurate. The robot would incorrectly recognize the ball to be too far when most of the ball is out of camera's range. So I adjusted the code such that the robot only move toward or away from the ball when the ball is centered.
- The second challenge I faced was that the robot couldn't center the ball and therefore may not move toward or away from it. It was because when the offset was too small, the speed calculated by PID was not fast enough for the wheel to start spinning. I found the approximate threshold speed fromt he wheels to start spinning. A conditional when the speed is less than threshold speed, the threshold speed will be the new speed. If the threshold speed is too big, the robot would oscilate too much;if the threshold speed is too small, the robot simply would not move.
- The final challenge that prevented my robot to accurately track the ball was that the turns were too big such that the ball could easily get out of range. So I added a conditional when the rasius of the ball is within the given range, the turn speed would be decreased based on the radius. The range was a little bit tricky to set because if the upper limit is too high, the speed will be reduced significantly when the ball is close to the camera and partially cut off.

**Next Step**
- I will be adding modifications specifically make the robot gesture controlled and sound reactive to my voice. It will be challenging to integrate them. Because this is a ball tracking robot, I will add a switch for it to be automatic or controllable.
-  
# Code
```python

```
# Final Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/F7M7imOVGug" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE

# Code
Here's where you'll put your code. The syntax below places it into a block of code. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize it to your project needs. 

```c++
void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  Serial.println("Hello World!");
}

void loop() {
  // put your main code here, to run repeatedly:

}
```

# Bill of Materials
Here's where you'll list the parts in your project. To add more rows, just copy and paste the example rows below.
Don't forget to place the link of where to buy each component inside the quotation marks in the corresponding row after href =. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize this to your project needs. 

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Raspberry Pi Kit | What the item is used for | $95.19 | <a href="https://www.amazon.com/RasTech-Raspberry-Starter-Heatsink-Screwdriver/dp/B0C8LV6VNZ"> Link </a> |
| Robot Chassis | What the item is used for | $18.99 | <a href="https://www.amazon.com/Smart-Chassis-Motors-Encoder-Battery/dp/B01LXY7CM3"> Link </a> |
| Screwdriver Kit | What the item is used for | $5.94 | <a href="https://www.amazon.com/Small-Screwdriver-Set-Mini-Magnetic/dp/B08RYXKJW9"> Link </a> |
| Ultrasonic Sensor | What the item is used for | $9.99 | <a href="https://www.amazon.com/WWZMDiB-HC-SR04-Ultrasonic-Distance-Measuring/dp/B0CQCCGXCP"> Link </a> |
| H Bridges | What the item is used for | $8.99 | <a href="https://www.amazon.com/ACEIRMC-Stepper-Controller-2-5-12V-H-Bridge/dp/B0923VMKSZ/"> Link </a> |
| Pi Cam | What the item is used for | $12.86 | <a href="https://www.amazon.com/gp/product/B07RWCGX5K"> Link </a> |
| Electronics Kit | What the item is used for | $11.98 | <a href="https://www.amazon.com/EL-CK-002-Electronic-Breadboard-Capacitor-Potentiometer/dp/B01ERP6WL4"> Link </a> |
| Motors | What the item is used for | $11.98 | <a href="https://www.amazon.com/AEDIKO-Motor-Gearbox-200RPM-Ratio/dp/B09N6NXP4H"> Link </a> |
| SD Card Adapter | What the item is used for | $9.99 | <a href="https://www.amazon.com/dp/B081VHSB2V?"> Link </a> |
| Digital Multimeter | What the item is used for | $11 | <a href="https://www.amazon.com/AstroAI-Digital-Multimeter-Voltage-Tester/dp/B01ISAMUA6"> Link </a> |
| Champion Sports Ball | What the item is used for | $16.73 | <a href="https://www.amazon.com/Champion-Sports-Inch-Coated-Density/dp/B000KYTTYO"> Link </a> |
| AA Batteries | What the item is used for | $18.74 | <a href="https://www.amazon.com/Duracell-Coppertop-AA-Ingredients-Long-lasting/dp/B0035LCFNQ"> Link </a> |
| USB Power Bank & Cable | What the item is used for | $16.19 | <a href="https://www.amazon.com/SIXTHGU-Portable-Charger-Charging-Flashlight/dp/B0C7PHKKNK/"> Link </a> |

# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

To watch the BSE tutorial on how to create a portfolio, click here.
