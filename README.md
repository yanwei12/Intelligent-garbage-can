# Intelligent Garbage Can

**by 顏丞瑋**

## Overview
A smart trash can controlled by three motors that tilt the upper platform. A camera observes for object movement, and when detected, the platform tilts to the corresponding angle to drop paper boxes into the trash can.

---

### Demo Video
[Watch the Demo](https://www.youtube.com/shorts/W6vrVf8LZIY)

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
- Use the Cura software to slice the STL file into a G-code file. Save the sliced file to an SD card using a USB-SD card converter. Insert the SD card into the printer to begin printing.
- **Note**: Preheat the printer before printing. Ensure the correct printer model and material are selected in Cura.

### Step 2: Assemble the Components
- Attach the 3D-printed arms using screws and nuts, adjusting the tightness so the arms can still move freely.
- Mount the MG996R servo motors into the base.

### Step 3: Wiring and Software Setup
1. Wire the PCA9685 to the Raspberry Pi using the I2C interface.
2. Connect the servo motors to the PCA9685.
3. Install the `adafruit_servokit` library on the Raspberry Pi.
4. Write a Python script to control the servo motors based on object detection from the camera.
   - **Note**: The Raspberry Pi's Python version is 3.7.3; newer libraries may not be compatible.

### Step 4: Connect PCA9685 to MG996R and Raspberry Pi
![Wiring Diagram](./IMG_4277.jpeg)

- **Pin Connections**:
  - PCA9685 VCC to Raspberry Pi 3.3V (Breadboard positive rail)
  - PCA9685 GND to Raspberry Pi GND (Breadboard negative rail)
  - PCA9685 SCL to Raspberry Pi SCL (GPIO3)
  - PCA9685 SDA to Raspberry Pi SDA (GPIO2)
  - PCA9685 PWM0 to MG996R signal wire (orange)
  - MG996R power wire (red) to breadboard positive rail
  - MG996R ground wire (brown) to breadboard negative rail

### Step 5: Adjust Servo Angles
Use the following code to set the servo angles to 90 degrees before attaching the arms to ensure proper alignment:

```python
from adafruit_servokit import ServoKit

kit = ServoKit(channels=16)

kit.servo[0].angle = 90
kit.servo[1].angle = 90
kit.servo[2].angle = 90
print("All servos set to 90 degrees")
```

Attach the arms in the upright position corresponding to 90 degrees.

### Step 6: Web Interface for Servo Control

Create a web interface to manually control the servo angles for testing:

```python
from flask import Flask, request, jsonify, render_template_string
from adafruit_servokit import ServoKit

kit = ServoKit(channels=16)

servo_angles = {0: 90, 1: 90, 2: 90}  # Initial angles

def move_servo(servo_channel, angle):
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
        <title>Servo Control</title>
    </head>
    <body>
        <h1>Servo Control</h1>
        <div>
            <label>Servo 0:</label>
            <button onclick="updateServo(0, -5)">-</button>
            <span id="servo0">90</span>
            <button onclick="updateServo(0, 5)">+</button>
        </div>
        <!-- Repeat for Servo 1 and Servo 2 -->
        <script>
        function updateServo(servo, delta) {
            const valueElement = document.getElementById(`servo${servo}`);
            let currentValue = parseInt(valueElement.textContent);
            let newValue = currentValue + delta;
            if (newValue >= 0 && newValue <= 180) {
                valueElement.textContent = newValue;
                fetch('/move', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ servo: servo, angle: newValue })
                });
            }
        }
        </script>
    </body>
    </html>
    """)

@app.route('/move', methods=['POST'])
def move():
    data = request.json
    servo = data.get('servo')
    angle = data.get('angle')
    if 0 <= angle <= 180:
        move_servo(servo, angle)
        return jsonify({"message": "Success"})
    return jsonify({"message": "Invalid angle"}), 400

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

Run the script and access the web interface at `http://<raspberry_pi_ip>:5000`.

### Step 7: Motion Detection with Camera
Ensure the camera is connected and enabled. Use the following script for motion detection and servo control:

```python
# Add motion detection script here...
```

---

## References
- [IoT Project on GitHub](https://github.com/oohyuti/IoT-Project)
- [PID Balloon Plate Project](https://github.com/nicohmje/PID-ballonplate)
