# Intelligent-garbage-can

**by 顏丞瑋**

## Overview
A smart trash can controlled by three motors that tilt the upper platform. A camera observes for object movement, and when detected, the platform tilts to the corresponding angle to drop paper boxes into the trash can.

---

## Required Components

### Hardware
1. Raspberry Pi *1
2. PCA9685 (I2C Interface) *1
3. Camera
4. MG996R Servo Motor *3
5. Machine Arm *3
6. Machine Base *1
7. Several Nuts and Screws

### Software
- **adafruit_servokit**: Used for controlling PCA9685.

---

## Final Product
![Final Product](https://hackmd.io/_uploads/HJYsHrrUJl.jpg)

---

## Assembly Steps

### Step 1: 3D Print the Base and Arms
- Use a 3D printer to fabricate the base and the arms required for the project.

### Step 2: Assemble the Components
1. Attach the machine arms to the base using nuts and screws.
2. Connect the MG996R servo motors to the machine arms.
3. Mount the Raspberry Pi and PCA9685 onto the base.
4. Attach the camera in a position that allows it to monitor object movement.

### Step 3: Wiring and Software Setup
1. Wire the PCA9685 to the Raspberry Pi using the I2C interface.
2. Connect the servo motors to the PCA9685.
3. Install the `adafruit_servokit` library on the Raspberry Pi.

   ```bash
   pip install adafruit-circuitpython-servokit
   ```
4. Write a Python script to control the servo motors based on object detection from the camera.

---

## Reference Materials
- [Adafruit ServoKit Documentation](https://learn.adafruit.com/16-channel-pwm-servo-driver/overview)
- [Raspberry Pi Camera Setup](https://projects.raspberrypi.org/en/projects/getting-started-with-picamera)
- [MG996R Servo Motor Datasheet](https://datasheetspdf.com/pdf-file/1367566/TowerPro/MG996R/1)

---

Feel free to clone this repository and adapt it to your needs!

