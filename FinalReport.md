# Final Report

## Team Members
* Akansha Singh (BENG, Sr)
* Peter Wang (ECE, Sr)
* Sophia d’Avila (MAE, So)
* Nick DeMello (ECE, Sr)

## Project Overview
The project objective can be broken down into two main components with 2 subprojects in the second. The first component is to design a robot that can be trained to drive 3 autonomous laps which was accomplished during the quarter. The second main project objective is the final project. The inspiration is a pet dog that plays fetch. Hence our robot will work to identify a green ball, i.e. a tennis ball, locate it, move towards it, pick it up, and return back to its initial location.

## Hardware: Mechanical Design
* Base Plate

![](https://drive.google.com/uc?id=1n9Ku6kVuSJONWaFOfA36iO3zFRGjnnIx)

* Jetson NANO Case

![](https://drive.google.com/uc?id=1RhUYImfOiptELLj3lSHlfREHwK0W55vA)

* Antenna Stand

![](https://drive.google.com/uc?id=1_5YsfDA5DGM-5H-jC0NBJc8I68_LMijj)

* Whole Camera Mount

![](https://drive.google.com/uc?id=1d6MruzU13rb37AK4ExvCpDGKtgnX-xDM)

* Camera Mount Parts


![](https://drive.google.com/uc?id=1oyD3zJGKD7pJZTxwGgbIC9UWk9nilAfb)
![](https://drive.google.com/uc?id=11m7Qp54gDxW_vBQmsf6gBEi6KwbSwnoQ)

* Claw

![](https://drive.google.com/uc?id=1qBsU3PNMqf_KEol1SNwIU4WW751yIEUa)


## Previous Designs
* Servomotor Attachment

![](https://drive.google.com/uc?id=1cnf87emX0QBpEkSKb4gQ744vhe3qTxnW)

The reason this design was tossed was because the 3D print was made too thick. While the design could have been thinned out and made to be smaller, there was not enough time left in the quarter. That being said, the part worked completely fine for the servomotor and fit snugly to allow proper connection from the servomotor to the claw gear. However, the piece itself was much bigger than the gear component in the claw and therefore inhibited the range of motion for the claw when rotating.

* Jetson NANO Case

![](https://drive.google.com/uc?id=1oolONU3gtkhctI77c9BJNAYzmM9MMc1H)

This Jetson NANO design was pulled from an online website. The reason this design was tossed was because during assembly, certain components of the case broke off. Moreover, there was a need for more space in the base plate and utilizing the area on top of the Jetson NANO could be used. Hence, refining the design to be more sturdy and to allow for other components to be stacked on top of the Jetson NANO case became imperative.

* Claw

![](https://drive.google.com/uc?id=1mz3C4xSoAnk3_IJo0NWfWE4vGnvLaOYG)

This claw design got tossed for multiple reasons. The primary one being that there was a design found online that suited the needs of the project better. That being said, while the overall cup design was what was intended for the project, incorporating this design into the actual robot would have been difficult due to introducing more components and parts to translate the motion of the servomotor into the motion of the claw mouth, hence this design was tossed.

## Electronic Components
* Jetson NANO Developer Kit
* GPS Antenna
* GPS
* VESC
* DC Power Supply Converter
* 2 Servomotors
* Steering and Claw control
* Servo PDB
* LIDAR
* OAK-D Camera
* 4 Cell LiPo Battery
* Antispark Switch
* 4 Pole DC Motor

## Gantt Chart

![](https://drive.google.com/uc?id=1ODiGicTadxdwxKoQJwqy2LkMJJOUIAhM)

## Electronic Wiring Schematic

![](https://drive.google.com/uc?id=1rUv86KUOX2D6T1cIfFEpA-Fdzz34MbE3)

## Final Set Up

![](https://drive.google.com/uc?id=1zSepqrNG9TTo-bybwIIAmixFqNbqgQvu)
![](https://drive.google.com/uc?id=1q9iMI3RIqaUdNPy3lyUNbGH-V9U7HNE8)
![](https://drive.google.com/uc?id=1iI5HZV7hZtzYf84qdmvumIsn0Ikfakta)

## Software

![](https://drive.google.com/uc?id=1KAj9wbiTN_L1DSzQkQRa2DjWAeI_orEL)

For our robot to follow the ball, we used an OpenCV-based vision system. The system uses the video from the robot's camera in real-time to detect the position of the ball. We used a method based on the color of the ball to detect it, by filtering the video to isolate yellow pixels, which represent the ball. We then applied a series of image processing techniques to help the ball's detection.

Once the ball is detected, the robot calculates its position relative to the camera using the distance between the ball and the center of the image. The robot then uses a PID controller that sends commands to the VESC using PyVesc to adjust its steering angle and speed, allowing it to move towards the ball. For example, if the ball is to the right of the car, the PID controller will calculate the required steering angle to turn the car towards the ball. Basically, the PID controller receives feedback from the camera to adjust the robot's movement in real-time, ensuring that the robot stays on course towards the ball.

### Code

import time

#import keyboard

import donkeycar as dk

import depthai

import cv2

import numpy as np



class VESC:

    def __init__(self, serial_port, percent=.2, has_sensor=False, start_heartbeat=True, baudrate=115200, timeout=0.05,
                 steering_scale=1.0, steering_offset=0.0):

        try:
            import pyvesc
        except Exception as err:
            print("\n\n\n\n", err, "\n")
            print("please use the following command to import pyvesc so that you can also set")
            print("the servo position:")
            print("pip install git+https://github.com/LiamBindle/PyVESC.git@master")
            print("\n\n\n")
            time.sleep(1)
            raise

        assert percent <= 1 and percent >= -1, '\n\nOnly percentages are allowed for MAX_VESC_SPEED (we recommend a value of about .2) (negative values flip direction of motor)'
        self.steering_scale = steering_scale
        self.steering_offset = steering_offset
        self.percent = percent
        self.offset = 0.1
        self.current_speed = 0.1

        try:
            self.v = pyvesc.VESC(serial_port, has_sensor, start_heartbeat, baudrate, timeout)
        except Exception as err:
            print("\n\n\n\n", err)
            print("\n\nto fix permission denied errors, try running the following command:")
            print("sudo chmod a+rw {}".format(serial_port), "\n\n\n\n")
            time.sleep(1)
            raise

    def run(self, angle, throttle):
        self.v.set_servo((angle * self.steering_scale) + self.steering_offset)
        self.v.set_duty_cycle(throttle * self.percent)

    def speed_up(self):
        if self.current_speed < 1:
            self.current_speed += 0.1

    def slow_down(self):
        if self.current_speed > 1:
            self.current_speed -= 0.1

    def forward(self):
        self.run(0.6 + self.offset, 0.2)

    def back(self):
        self.current_speed = -self.current_speed
        self.run(0.6 + self.offset, self.current_speed)

    def left(self):
        self.run(0.35 + self.offset, 0.15)

    def right(self):
        self.run(0.85 + self.offset, 0.15)

    def hard_left(self):
        self.run(0 + self.offset, 0.1)

    def hard_right(self):
        self.run(1 + self.offset, 0.1)

    def stop(self):
        self.run(0.6 + self.offset, 0)

def main():

    lower_yellow = np.array([20, 80, 70])
    
    upper_yellow = np.array([90, 255, 255])
    
    vesc = VESC("/dev/ttyACM0")
    

    pipeline = depthai.Pipeline()
    cam = pipeline.createColorCamera()
    cam.setPreviewSize(700, 500)  # set the preview size to 300x300 pixels
    cam.setInterleaved(False)
    cam.setColorOrder(depthai.ColorCameraProperties.ColorOrder.BGR)
    cam.setResolution(depthai.ColorCameraProperties.SensorResolution.THE_1080_P)
    cam.setPreviewKeepAspectRatio(False)
    cam.setVideoSize(1280, 1080)
    cam.setFps(30)
    xout = pipeline.createXLinkOut()
    xout.setStreamName('preview')
    cam.preview.link(xout.input)

    with depthai.Device(pipeline) as device:
        output_stream = device.getOutputQueue('preview', 1, True)
        while True:
            in_preview = output_stream.get()
            preview_frame = in_preview.getCvFrame()
            hsv = cv2.cvtColor(preview_frame, cv2.COLOR_BGR2HSV)
            mask = cv2.inRange(hsv, lower_yellow, upper_yellow)
            contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

            # Draw vertical line in the middle of the frame
            cv2.line(preview_frame, (preview_frame.shape[1] // 2, 0),
                     (preview_frame.shape[1] // 2, preview_frame.shape[0]), (0, 0, 255), 2)

            largest_area = 0
            largest_contour = None
            for cnt in contours:
                area = cv2.contourArea(cnt)
                if area > largest_area:
                    largest_area = area
                    largest_contour = cnt

            # Draw square around largest contour
            x, y, w, h = cv2.boundingRect(largest_contour)
            center_left = x + w // 5
            center_right = x + ((w // 5) * 4)
            if center_left > preview_frame.shape[1] // 2:
                print("right")
                vesc.right()

            elif center_right < preview_frame.shape[1] // 2:
                print("left")
                vesc.left()

            else:
                print("line")
                vesc.forward()
            cv2.rectangle(preview_frame, (x, y), (x + w, y + h), (0, 255, 0), 2)

            # Display resulting image with rectangle and areas
            cv2.imshow('preview', preview_frame)

            # Exit on 'q' key press
            if cv2.waitKey(1) & 0xFF == ord('q'):
                break

    # Release the camera and close all windows
    cv2.destroyAllWindows()


## Milestones
* Donkey AI Training and AI Driving
* Training the robot to open DonkeySIM was a huge milestone for the group due to lack of exposure to DonkeySIM prior to this class.
* Behavioral Imitating AI model (GPU Cluster Training) - 3 Autonomous Laps
* Object Recognition - Recognizing yellow objects + tennis ball

## Potential Future Work/Unaccomplished Goals
There are many suggestions for future work and unaccomplished goals. The goals can be broken down into two projects including hardware design and software.

### Hardware Design
Some future work and unaccomplished goals for this robot include a refined claw design. As described previously, the claw design had gone through multiple iterations to refine it. That being said, there were some issues for the claw design printed out in this run. The primary error includes screw holes. The screw holes within the gear for the claw did not match any screws to our availability due to the CAD model design. Thus, in the future, it would be wise to make the hole either bigger or smaller to more appropriately fit a specific screw size.
Another key future suggestion for the design is the mouth of the claw and how it opens. Given the project objective to grab a ball, the current design for the claw makes it a bit difficult to pick up the ball. This is because the two lips of the claw mouth open at an angle and close at an angle with the help of the gears. In the future, it would be useful to have a claw mouth that, instead of opening its mouth at an angle and thereby potentially pushing the ball away, will open and close with both sides parallel to each other, using a four bar linkage system. If the lips of the claw mouth were to open in a sideways linear motion and close in a sideways linear motion instead, there would be less of a chance that the ball would slip and run away from the robot. Moreover, for future applications, it would be nice to design a claw that would be more mdoular to different objects including a soda can, a water bottle, or even a book.

Another future suggestion includes refining the design for the servomotor to claw cover. Currently, as described above, there was a previous iteration of the servomotor to claw that was designed. However, upon construction, the design constructed on CAD turned out to be too thick and too big. While the servomotor fit snugly into the printed part, the design itself ultimately had to be tossed as the printed component got in the range of motion for the claw, thereby inhibiting its movements. Instead, the design and print was tossed and the servomotor was simply taped to the claw. That being said, for future design work, it would be wise to thin down the servomotor to claw CAD file so that it does not get in the way of the claw’s two gears and its movement.

And lastly, one important thing to think about when building the Jetson NANO case is the optimization of space on the top of the robot. For our previous Jetson NANO case we got a pre-made model that we found and printed it, but with time, when actually assembling the robot, we felt like there was not enough space to put all the necessary parts. Because of that we designed a new Jetson NANO case, which allowed us to place things on top of it.

### Software Design
ENTER FUTURE SOFTWARE INFO HERE


## Videos

[![Robot Following Tennis Ball]](https://drive.google.com/file/d/1MhjCJR0Cql-D31eOV3ycqs1MM1ly1piv/view?usp=sharing)


[![Claw Demonstration]](https://drive.google.com/file/d/1XQSJmD2X0WHFRSfH-Wv21vTm3lmpRYNG/view?usp=sharing)


## Progress Reports and Final Presentations
First Progress Report: https://docs.google.com/presentation/d/1FHS0wTwU7fyivqQRh6NMlz-c80lTc7Qd7yyvWIk43dk/edit?usp=sharing
Second Progress Report: https://docs.google.com/presentation/d/1Ua9LzGS5SNXx9sHRqjRf_p_igt0errfVVrgwLYT4yDQ/edit?usp=sharing
Final Progress Report: https://docs.google.com/presentation/d/15dULDgGm8JjDpnWzwsE6JQuID-shEysJk5i4PfPxzdU/edit?usp=sharing

## Acknowledgments
We would like to thank Professor Jack Silberman and TA Kishore Nukala and Moises Lopez-Mendoza for the opportunity of taking this class and their everlasting assistance.
