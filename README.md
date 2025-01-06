# Intelligent-garbage-can

**by 顏丞瑋**

## Overview
A smart trash can controlled by three motors that tilt the upper platform. A camera observes for object movement, and when detected, the platform tilts to the corresponding angle to drop paper boxes into the trash can.

---

### Demo Video
[Watch the demo](https://www.youtube.com/shorts/W6vrVf8LZIY)

## Hardware
1. Raspberry Pi *1
2. PCA9685 (I2C Interface) *1
3. Camera
4. MG996R Servo Motor *3
5. Machine Arm *3
6. Machine Base *1
7. Several Nuts and Screws
8. Battery Pack
9. Breadboard

### Software
- **adafruit_servokit**: Used for controlling PCA9685.

---

## Final Product
![Final Product](./IMG_4275-2.jpeg)

---

## Assembly Steps

### Step 1: 3D Print the Base and Arms
- Use a 3D printer to fabricate the base and the arms required for the project.
- Use Cura to slice STL files into G-code files. Transfer the sliced G-code file onto an SD card using a USB-SD card converter. Insert the SD card into the 3D printer to start printing.
- Note: Preheat the printer before starting. In Cura, ensure the correct printer model and material type are selected.

### Step 2: Assemble the Components
- Use screws to assemble the 3D-printed arms, adjusting the nuts to an appropriate tightness (not too tight to allow free movement).
- Mount the MG996R motors into the base.

### Step 3: Wiring and Software Setup
1. Wire the PCA9685 to the Raspberry Pi using the I2C interface.
2. Connect the servo motors to the PCA9685.
3. Install the `adafruit_servokit` library on the Raspberry Pi.
4. Write a Python script to control the servo motors based on object detection from the camera. 
   - Note: The Python version on the Raspberry Pi is 3.7.3, which may limit compatibility with newer libraries.

### Step 4: Connect PCA9685 and MG996R to Raspberry Pi
![Wiring Diagram](./IMG_4277.jpeg)
- Use the `pinout` command to verify Raspberry Pi GPIO pins.
- Connections:
  - PCA9685 VCC to Raspberry Pi 3.3V and breadboard positive rail.
  - PCA9685 GND to Raspberry Pi GND and breadboard negative rail, shared with the battery pack.
  - PCA9685 SCL to Raspberry Pi SCL (GPIO3).
  - PCA9685 SDA to Raspberry Pi SDA (GPIO2).
  - PCA9685 PWM channels (e.g., PWM0) to MG996R signal wires (orange).
  - MG996R power wires (red) to breadboard positive rail.
  - MG996R ground wires (brown) to breadboard negative rail.

### Step 5: Adjust Motor Angles
Run the following script to set all motors to a 90-degree angle. Attach the arms to the motors in the upright position to ensure correct alignment.

```python
from flask import Flask, request, jsonify, render_template_string
from adafruit_servokit import ServoKit

kit = ServoKit(channels=16)

servo_1 = 0
servo_2 = 1
servo_3 = 2
servo_angles = {servo_1: 90, servo_2: 90, servo_3: 90}  # Initial angles set to 90

def move_servo(servo_channel, angle):
    """Move the servo motor to the specified angle"""
    if 0 <= angle <= 180:
        kit.servo[servo_channel].angle = angle
        print(f"Servo {servo_channel} moved to {angle} degrees")
    else:
        print("Invalid angle! Must be between 0 and 180.")

app = Flask(__name__)

@app.route('/')
def index():
    return render_template_string("""
    <!DOCTYPE html>
    <html>
    <head>
        <title>Servo Motor Control</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; }
            .button-container { margin: 20px 0; }
            .button-label { margin-bottom: 5px; font-weight: bold; }
        </style>
    </head>
    <body>
        <h1>Servo Motor Control</h1>

        <div class="button-container">
            <div class="button-label">Servo 0</div>
            <button onclick="updateServo(0, -5)">-</button>
            <span id="servo0-value">90</span>°
            <button onclick="updateServo(0, 5)">+</button>
        </div>

        <div class="button-container">
            <div class="button-label">Servo 1</div>
            <button onclick="updateServo(1, -5)">-</button>
            <span id="servo1-value">90</span>°
            <button onclick="updateServo(1, 5)">+</button>
        </div>

        <div class="button-container">
            <div class="button-label">Servo 2</div>
            <button onclick="updateServo(2, -5)">-</button>
            <span id="servo2-value">90</span>°
            <button onclick="updateServo(2, 5)">+</button>
        </div>

        <script>
            function updateServo(servo, delta) {
                const valueElement = document.getElementById(`servo${servo}-value`);
                let currentAngle = parseInt(valueElement.textContent);
                let newAngle = currentAngle + delta;
                if (newAngle < 0) newAngle = 0;
                if (newAngle > 180) newAngle = 180;

                valueElement.textContent = newAngle;
                fetch('/move', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({ servo: servo, angle: newAngle }),
                })
                .then(response => response.json())
                .then(data => console.log(data.message))
                .catch(error => console.error('Error:', error));
            }
        </script>
    </body>
    </html>
    """)

@app.route('/move', methods=['POST'])
def move():
    data = request.json
    servo = data.get("servo")
    angle = data.get("angle")

    if servo in [servo_1, servo_2, servo_3] and 0 <= angle <= 180:
        servo_angles[servo] = angle
        move_servo(servo, angle)
        return jsonify({"status": "success", "message": f"Servo {servo} moved to {angle} degrees"})
    else:
        return jsonify({"status": "error", "message": "Invalid servo or angle"}), 400

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### Step 6: Connect the Camera
1. Ensure the camera module is connected to the Raspberry Pi CSI interface.
2. Verify the camera is detected using:
   ```
   vcgencmd get_camera
   ```
   The output should be:
   ```
   supported=1 detected=1
   ```
   If not detected, enable the camera using `sudo raspi-config` and reboot.
3. Use the script below to detect motion via the camera and control the servo motors accordingly:

```python
from flask import Flask, Response, render_template_string
from adafruit_servokit import ServoKit
import cv2
import time

kit = ServoKit(channels=16)
camera = cv2.VideoCapture(0)

servo_1 = 0
servo_2 = 1
servo_3 = 2
servo_angles = {servo_1: 90, servo_2: 90, servo_3: 90}  # Initial angles set to 90

fgbg = cv2.createBackgroundSubtractorMOG2(history=500, varThreshold=25, detectShadows=True)

last_move_time = 0
MOVE_INTERVAL = 5  # Time interval (seconds)

def move_servo(servo_channel, angle):
    if 0 <= angle <= 180:
        kit.servo[servo_channel].angle = angle
        servo_angles[servo_channel] = angle
        print(f"Servo {servo_channel} moved to {angle} degrees")

def detect_motion(frame):
    fgmask = fgbg.apply(frame)
    kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (5, 5))
    fgmask = cv2.morphologyEx(fgmask, cv2.MORPH_OPEN, kernel)
    fgmask = cv2.morphologyEx(fgmask, cv2.MORPH_CLOSE, kernel)
    contours, _ = cv2.findContours(fgmask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    motion_detected = any(cv2.contourArea(c) > 1000 for c in contours)
    return motion_detected, frame

def generate_frames():
    global last_move_time

    while True:
        success, frame = camera.read()
        if not success:
            break
        else:
            has_motion, frame = detect_motion(frame)
            current_time = time.time()
            if has_motion and (current_time - last_move_time > MOVE_INTERVAL):
                print("Motion detected, moving servos.")
                move_servo(servo_1, 90)
                move_servo(servo_2, 120)
                move_servo(servo_3, 90)
                last_move_time = current_time
                time.sleep(1)
            elif not has_motion:
                print("No motion detected, resetting servos.")
                move_servo(servo_1, 90)
                move_servo(servo_2, 90)
                move_servo(servo_3, 90)

            _, buffer = cv2.imencode('.jpg', frame)
            frame = buffer.tobytes()

            yield (b'--frame\r\n'
                   b'Content-Type: image/jpeg\r\n\r\n' + frame + b'\r\n')

app = Flask(__name__)

@app.route('/video')
def video_feed():
    return Response(generate_frames(), mimetype='multipart/x-mixed-replace; boundary=frame')

@app.route('/')
def index():
    return render_template_string("""
    <!DOCTYPE html>
    <html>
    <head>
        <title>Live Stream and Motion Detection</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 0; background-color: black; }
            img { display: block; margin: auto; max-width: 100%; height: auto; }
        </style>
    </head>
    <body>
        <img src="/video">
    </body>
    </html>
    """)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

---

## Reference Materials
- [IoT Project GitHub Repository](https://github.com/oohyuti/IoT-Project)
- [PID Ball on Plate Project](https://github.com/nicohmje/PID-ballonplate)
