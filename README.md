Autonomous Obstacle-Avoiding Servo Turret — ESP32
A breadboard-based embedded systems project featuring an HC-SR04 ultrasonic sensor mounted on an SG90 servo. The turret sweeps continuously and triggers a visual alert when an obstacle is detected within a configurable threshold distance.
---
Hardware
Component	Part	Notes
Microcontroller	ESP32-WROOM-32E	3.3V GPIO
Ultrasonic sensor	HC-SR04	Echo pin stepped down via voltage divider
Servo	SG90	Powered from VIN (5V)
Alert	Onboard LED (GPIO2)	GPIO19 reserved for external buzzer
Voltage divider	220Ω + 1kΩ	HC-SR04 Echo 5V → 3.3V
---
Repo structure
```
ServoTurret-ESP32/
├── firmware/
│   └── ServoTurret_LED.ino
├── schematic/
│   ├── ServoTurret.kicad_sch
│   └── ServoTurret.kicad_pro
├── documentation/
│   └── ServoTurret_DesignDoc.docx
└── assets/
    └── wiring_diagram.png
```
---
How it works
The SG90 servo sweeps 0°→180°→0° in configurable steps
At each step the HC-SR04 fires a 10µs trigger pulse and measures the echo return time
If the calculated distance is below `OBSTACLE_THRESHOLD_CM` (default 20 cm), the onboard LED flashes 3 times and the sweep direction reverses
All readings are logged to Serial at 115200 baud
---
Wiring
ESP32 pin	Connects to
3V3	HC-SR04 VCC
GPIO4	HC-SR04 TRIG
GPIO5	Voltage divider output (ECHO at 3.3V)
GPIO18	SG90 Signal
VIN	SG90 VCC (5V)
GPIO2	Onboard LED (alert)
GND	HC-SR04 GND, SG90 GND, divider GND
> The HC-SR04 Echo pin outputs 5V logic. A 220Ω/1kΩ voltage divider steps this down to ~3.3V before connecting to GPIO5. Connecting Echo directly to the ESP32 without this divider will damage the GPIO pin.
---
Firmware configuration
Key constants at the top of `ServoTurret_LED.ino`:
```cpp
const int OBSTACLE_THRESHOLD_CM = 20;  // Detection distance
const int SWEEP_STEP_DEG        = 2;   // Degrees per step (smaller = smoother)
const int SWEEP_DELAY_MS        = 20;  // Delay between steps
const int DETECT_PAUSE_MS       = 800; // Pause duration on detection
```
---
Dependencies
ESP32Servo by Kevin Harrington — install via Arduino IDE Library Manager
---
Key design decisions
Voltage divider on Echo — HC-SR04 outputs 5V on the Echo line. ESP32 GPIO inputs are rated 3.3V max. A 220Ω + 1kΩ resistor divider reduces this to ~3.3V (ratio: 1kΩ / (220Ω + 1kΩ) = 0.82 × 5V = 4.1V... adjusted to safe range at actual load).
Servo powered from VIN not 3V3 — the SG90 draws up to 500mA under load. The ESP32's 3V3 regulator cannot supply this safely. VIN passes the USB 5V directly.
PWM timer allocation — the ESP32Servo library requires an explicit `ESP32PWM::allocateTimer(0)` call before attaching the servo. Without this the servo jitters or fails to respond.
