# Airsoft Stattrack System

A portable Airsoft shot tracker using a Raspberry Pi, SSD1306 OLED display, and a shock or sound sensor. Counts shots and displays them on an OLED screen.

---

## 1. Hardware

### Required Components
- **Raspberry Pi** (e.g., Pi Zero W or Pi 4)  
- **SSD1306 OLED Display** (I²C)  
- **3 Buttons** (e.g., Reset, Mode, Start/Stop)  
- **Shot detection sensor**  
  - Accelerometer / Shock Sensor / Piezo → detects sudden hits  
  - Alternatively, Sound Sensor (microphone module) → detects gunshot noise  
- **Battery / Powerbank** → portable power supply  

### Wiring
- **SSD1306** → I²C (SDA/SCL, +3.3V, GND)  
- **Buttons** → GPIOs with Pull-Up/Pull-Down resistors  
- **Sensor** → GPIO (digital output for Piezo or analog for microphone, possibly via ADC)  

---

## 2. Software

### Programming Language
Python is recommended (works on Linux, supports GPIO/I²C, many libraries for SSD1306 and sensors).

### Libraries
```bash
sudo apt update
sudo apt install python3-pip
pip3 install RPi.GPIO
pip3 install adafruit-circuitpython-ssd1306
pip3 install adafruit-circuitpython-busdevice
```

### Basic Logic

#### Initialization 

  - Initialize display
  - Initialize GPIOs for buttons and sensor

#### Main Loop

  - Check sensor for shot detection
  - If triggered → increment shot count and update display
  - Check buttons (e.g., Reset, Start/Stop, Mode)

#### Display Update
  - OLED shows shot count, possibly mode or timer

Python Example (Basic)
```bash
python
Code kopieren
import time
import board
import digitalio
from adafruit_ssd1306 import SSD1306_I2C
import RPi.GPIO as GPIO

# GPIO Setup
GPIO.setmode(GPIO.BCM)
sensor_pin = 17  # Piezo Sensor
reset_button = 27
GPIO.setup(sensor_pin, GPIO.IN)
GPIO.setup(reset_button, GPIO.IN, pull_up_down=GPIO.PUD_UP)

# Display Setup
i2c = board.I2C()
oled = SSD1306_I2C(128, 64, i2c)
oled.fill(0)
oled.show()

# Shot counter
shots = 0

def update_display():
    oled.fill(0)
    oled.text("Airsoft Stats", 0, 0)
    oled.text(f"Shots: {shots}", 0, 20)
    oled.show()

try:
    while True:
        if GPIO.input(sensor_pin) == GPIO.HIGH:
            shots += 1
            update_display()
            time.sleep(0.2)  # Debounce / delay

        if GPIO.input(reset_button) == GPIO.LOW:
            shots = 0
            update_display()
            time.sleep(0.2)

        time.sleep(0.01)

except KeyboardInterrupt:
    GPIO.cleanup()
```

## 3. Possible Enhancements
  - Sound detection instead of Piezo → uses microphone, detects gunshot noise
  - Multiple modes → e.g., timer, team kills, statistics
  - Save to file / SD card → persist shot counts
  - Bluetooth or Wi-Fi → live stats or remote monitoring
  - Next steps: Create a full wiring diagram and enhance software for debounce, multiple buttons, and mode switching.

##  4. SSD1306 to Raspberry Pi 5 Wiring (I²C)

SSD1306 → Raspberry Pi 5 Wiring Plan (I²C)
The Raspberry Pi 5 uses the same 40-pin header as previous models, including the I²C pins.
Your SSD1306 I²C display typically has 4 pins:

| SSD1306 Pin | Raspberry Pi 5 Pin | GPIO | Description         |
|-------------|--------------------|-------|---------------------|
| VCC         | Pin 1              | 3.3V  | Power for display   |
| GND         | Pin 6              | GND   | Ground              |
| SCL         | Pin 5              | GPIO 3 (SCL1) | I²C Clock |
| SDA         | Pin 3              | GPIO 2 (SDA1) | I²C Data  |

Raspberry Pi 5 I²C Notes

I²C is disabled by default. Enable it here:
Settings → Raspberry Pi Configuration → Interfaces → I²C → Enable
or via terminal:
```
sudo raspi-config
```
After enabling, reboot the Pi:

```
sudo reboot
```

Verify the Display is Detected

Once wired and I²C enabled:
```
sudo apt install -y i2c-tools
i2cdetect -y 1
```

You should see the SSD1306 appear as 0x3C or 0x3D in the grid.

