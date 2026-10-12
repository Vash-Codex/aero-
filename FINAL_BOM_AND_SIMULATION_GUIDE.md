# Final BOM & Simulation Guide
## Ultra-Cheap 1S RC Airplane with Arduino Uno + Gamepad Control

---

## 1. FINAL BILL OF MATERIALS (BOM)

### 1.1 Electronics (Core)

| Item | Qty | Part Number / Example | Est. Cost | Where to Buy | Notes |
|------|-----|----------------------|-----------|--------------|-------|
| **ESP32 DevKit** | 1 | ESP32-WROOM-32 | $10 | Amazon / AliExpress | You have this |
| **Arduino Uno** | 1 | ATmega328P Uno Clone | $5-8 | Amazon / AliExpress | You have this |
| **NRF24L01 with antenna** | 2 | NRF24L01+ PA+LNA | $3-5 each | AliExpress | Need 2: one for Uno, one for ESP32 |
| **1S LiPo Battery** | 1 | 3.7V 400-500mAh | $12 | Banggood / Amazon | Turnigy, Tattu, or cheap brand |
| **1S Brushless Motor** | 1 | 1104 KV 2300-3500 | $5-8 | AliExpress | Tarot or generic |
| **Brushless ESC** | 1 | 5A BLHeli_S | $6-10 | AliExpress | Tiny, <3g |
| **Micro Servo** | 1 | 3.3V rated (GY90S) | $4-5 | AliExpress | **Must be 3.3V-rated** |
| **3.3V LDO Regulator** | 1 | AMS1117-3.3 or similar | $0.50-1 | AliExpress | Powers ESP32 from 1S LiPo |
| **Capacitors** | - | 100 µF + 100 nF | $1-2 | Local electronics store | Power filtering |
| **Resistors** | - | 1/4W assortment | $1 | Local or AliExpress | General purpose |
| **USB Cable (A to B)** | 1 | Standard Arduino cable | $2-3 | Amazon | For Uno programming + serial |
| **USB Cable (Micro)** | 1 | Micro-USB cable | $2-3 | Amazon | For ESP32 programming |
| **3-pin servo connectors** | 2-3 | JST-GH or generic | $1 | AliExpress | For servo plugs |
| **2-pin battery connectors** | 2 | XT30 or JST-PH | $1-2 | AliExpress | Battery + ESC |
| **Jumper wires / breadboard** | - | 22 AWG solid core | $3-5 | Amazon | For bench testing |
| **Breadboard** | 1 | 400-hole solderless | $2-3 | Amazon | For prototyping |
| | | | **Total: ~$65-90** | | |

### 1.2 Mechanical / Frame

| Item | Qty | Est. Cost | Where | Notes |
|------|-----|-----------|-------|-------|
| **Foam board** | 1 sheet | $3-5 | Local hobby store | For DIY frame |
| **Propeller (5-6")** | 1 | $3-5 | AliExpress | Folding or fixed blade |
| **Tape / hot glue / adhesive** | - | $2-3 | Local | For assembly |
| **Balsa wood (optional)** | - | $2-5 | Hobby store | Structural reinforcement |
| | | **~$10-18** | | |

### 1.3 Tools & Testing

| Item | Qty | Est. Cost | Have? |
|------|-----|-----------|-------|
| **Soldering iron + solder** | 1 | $10-20 | Ideally yes |
| **Multimeter** | 1 | $8-15 | Ideally yes |
| **USB LiPo Charger** | 1 | $8-12 | Need to buy |
| **Oscilloscope (optional)** | 1 | $50+ | For PWM verification |
| | | **~$26-47** | |

### 1.4 Software (All Free)

| Software | Purpose | Download |
|----------|---------|----------|
| **Arduino IDE** | Program Arduino Uno & ESP32 | arduino.cc |
| **Python 3** | Run gamepad reader script | python.org |
| **Pygame** | Python gamepad library | `pip install pygame` |
| **NRF24 Arduino Library** | RF24 control on Arduino | Arduino Library Manager |
| **NRF24 ESP32 Library** | RF24 control on ESP32 | Arduino Library Manager |
| **Proteus / Tinkercad** | Circuit simulation (optional) | proteus.com / tinkercad.com |

---

## 2. COMPLETE WIRING DIAGRAM

### 2.1 Arduino Uno (Transmitter) Wiring

```
ARDUINO UNO TRANSMITTER SIDE
═════════════════════════════════════════════════════════

┌─────────────────────────┐
│   Arduino Uno           │
├─────────────────────────┤
│ GND ────────────────┐   │
│ 3.3V ───────────┐   │   │
│ D7 ─────────┐   │   │   │
│ D8 ──────┐  │   │   │   │
│ D11 ───┐ │  │   │   │   │
│ D12 ──┐│ │  │   │   │   │
│ D13 ─┐││ │  │   │   │   │
│      │││ │  │   │   │   │
│ TX ──────────→ USB Serial to Laptop
│ RX ←─────────── USB Serial from Laptop
│      │││ │  │   │   │   │
└──────┼┼┼─┼──┼───┼───┼───┘
       │││ │  │   │   │
       │││ │  │   │   └──→ 3.3V (Power)
       │││ │  │   └──────→ GND (Power)
       │││ │  └──────────→ CE (Chip Enable)
       │││ └─────────────→ CSN (Chip Select)
       │││
       ││└────────────────→ MOSI (SPI)
       │└─────────────────→ MISO (SPI)
       └──────────────────→ SCK (SPI Clock)
                │
                ▼
        ┌───────────────────┐
        │  NRF24L01 Module  │
        │ (with PA+LNA)     │
        │ Transmitter       │
        └───────────────────┘
```

**Pin-by-pin connections:**

| Arduino Uno Pin | Signal | NRF24L01 Pin | Signal |
|-----------------|--------|--------------|--------|
| GND | Ground | GND | Ground |
| 3.3V | Power | VCC | Power (3.3V) |
| D7 | CE | CE | Chip Enable |
| D8 | CSN | CSN | Chip Select |
| D11 | MOSI | MOSI | Data In |
| D12 | MISO | MISO | Data Out |
| D13 | SCK | SCK | Clock |

**USB Connection (to Laptop):**
- Arduino USB port → Windows Laptop (via standard A-to-B cable)
- This provides both power and serial communication

---

### 2.2 ESP32 (Receiver) Wiring

```
ESP32 RECEIVER SIDE
═════════════════════════════════════════════════════════

┌─────────────────────────────────────┐
│   ESP32 DevKit                      │
├─────────────────────────────────────┤
│ GND ──────────────────┐             │
│ GPIO 3.3V ────────┐   │             │
│ GPIO 4 ──────┐    │   │             │
│ GPIO 5 ────┐ │    │   │             │
│ GPIO 18 ──┐│ │    │   │             │
│ GPIO 19 ─┐││ │    │   │             │
│ GPIO 23 ┐│││ │    │   │             │
│          ││││ │    │   │             │
│ GPIO 26 ──→ ESC Signal (PWM)        │
│ GPIO 25 ──→ Servo Signal (PWM)      │
│                                     │
└──┬───────┬┬┬┬─┬────┬───┬───────────┘
   │       ││││ │    │   │
   │       ││││ │    │   └──→ 3.3V (Power)
   │       ││││ │    └──────→ GND (Power)
   │       ││││ └──────────→ CE
   │       │││└───────────→ CSN
   │       ││└────────────→ MOSI
   │       │└─────────────→ MISO
   │       └──────────────→ SCK
   │
   └──→ USB Micro (Power + Programming)
           from 1S LiPo via 3.3V regulator
                │
                ▼
        ┌───────────────────┐
        │  NRF24L01 Module  │
        │ (with PA+LNA)     │
        │ Receiver          │
        └───────────────────┘
```

**Pin-by-pin connections:**

| ESP32 Pin | Signal | NRF24L01 Pin | Signal |
|-----------|--------|--------------|--------|
| GND | Ground | GND | Ground |
| 3.3V | Power | VCC | Power (3.3V) |
| GPIO 4 | CE | CE | Chip Enable |
| GPIO 5 | CSN | CSN | Chip Select |
| GPIO 23 | MOSI | MOSI | Data In |
| GPIO 19 | MISO | MISO | Data Out |
| GPIO 18 | SCK | SCK | Clock |
| GPIO 26 | PWM | ESC Signal In | Throttle command |
| GPIO 25 | PWM | Servo Signal In | Elevator command |

---

### 2.3 Power Distribution on Airplane

```
AIRPLANE POWER ROUTING
═════════════════════════════════════════════════════════

┌─────────────────────────────┐
│   1S LiPo Battery           │
│  (3.7V, 400-500mAh)         │
└──────┬──────────────────────┘
       │
       ├─→ Battery (+) terminal
       │
    ┌──┴──────────────────────────────────┐
    │                                      │
    ▼                                      ▼
┌──────────────┐                  ┌──────────────┐
│   ESC        │                  │   3.3V LDO   │
│   Input (+)  │                  │  Regulator   │
└──────┬───────┘                  └──────┬───────┘
       │                                 │
       │                            ┌────▼────┐
       │                            │ 3.3V    │
       │                            │  Rail   │
       │                            └────┬────┘
       │                                 │
       │                    ┌────────────┼────────────┐
       │                    │            │            │
       ▼                    ▼            ▼            ▼
   ┌────────┐         ┌─────────┐  ┌────────┐  ┌──────────┐
   │ Motor  │         │ ESP32   │  │ Servo  │  │NRF24L01  │
   │        │         │ 3.3V in │  │ Power  │  │ VCC      │
   └────────┘         └─────────┘  └────────┘  └──────────┘

BATTERY NEGATIVE (GND) - STAR POINT
│
├─→ ESC GND
├─→ ESP32 GND (all pins)
├─→ Servo GND
├─→ NRF24L01 GND
└─→ Common ground plane

DECOUPLING / FILTERING
├─→ 100 µF capacitor: across LDO output (3.3V to GND)
├─→ 100 nF capacitor: across ESP32 3.3V pin
└─→ 100 µF capacitor: across ESC power input (optional, for motor noise)
```

---

## 3. SIMULATION & TESTING BEFORE BUILDING

### 3.1 Circuit Simulation (Proteus / Tinkercad)

**What you can simulate:**
- Power distribution without actual hardware
- Voltage levels at each node
- Current draw calculations
- Ground plane integrity
- Decoupling capacitor placement

**Tools:**
1. **Tinkercad Circuits** (Free, web-based, easiest for beginners)
   - https://www.tinkercad.com/circuits
   - Drag-and-drop Arduino, NRF24, ESP32 symbols
   - Simulate power rails
   - No actual firmware execution, but good for wiring verification

2. **Proteus Design Suite** (Professional, $200+, overkill for this)
   - Full circuit simulation with real microcontroller models
   - Can run actual Arduino/ESP32 code in simulation
   - Overkill unless you're doing this professionally

3. **KiCad** (Free, open-source, steeper learning curve)
   - Full PCB design and electrical rule checking
   - Good for verifying connections
   - Simulation requires additional plugin (ngspice)

**Recommendation for you:** Use **Tinkercad Circuits** for free wiring verification.

---

### 3.2 Software Simulation (Before Hardware)

#### Step 1: Test Gamepad Reading (Python on Laptop)

**Simulate:** Gamepad input without hardware

```python
# test_gamepad.py
import pygame
import sys

pygame.init()
pygame.joystick.init()

# Detect gamepad
joystick_count = pygame.joystick.get_count()
if joystick_count == 0:
    print("No gamepad detected! Plug in your gamepad.")
    sys.exit(1)

joystick = pygame.joystick.Joystick(0)
joystick.init()

print(f"Gamepad detected: {joystick.get_name()}")
print(f"Axes: {joystick.get_numaxes()}")
print(f"Buttons: {joystick.get_numbuttons()}")
print("\nPress gamepad sticks and buttons (Ctrl+C to exit)...\n")

clock = pygame.time.Clock()

try:
    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                sys.exit(0)
            elif event.type == pygame.JOYAXISMOTION:
                # Read analog sticks
                if event.axis == 0:  # Left stick X
                    print(f"Left Stick X: {event.value:.2f}")
                elif event.axis == 1:  # Left stick Y
                    print(f"Left Stick Y: {event.value:.2f}")
                elif event.axis == 2:  # Right stick X
                    print(f"Right Stick X: {event.value:.2f}")
                elif event.axis == 3:  # Right stick Y
                    print(f"Right Stick Y: {event.value:.2f}")
            elif event.type == pygame.JOYBUTTONDOWN:
                print(f"Button pressed: {event.button}")
        
        clock.tick(20)  # 20 Hz update rate

except KeyboardInterrupt:
    print("\nExiting...")
    pygame.quit()
```

**Run this first** to confirm your gamepad is recognized by Python.

---

#### Step 2: Simulate Serial Communication (Arduino ↔ Laptop)

**Simulate:** Send throttle/elevator values from Python to Arduino Uno without actual RF module

**Arduino code (simulate_receiver.ino):**
```cpp
void setup() {
  Serial.begin(9600);
  Serial.println("Arduino ready for serial commands");
}

void loop() {
  if (Serial.available()) {
    String command = Serial.readStringUntil('\n');
    // Parse "T:1200,E:1500"
    int throttle = 1000, elevator = 1500;
    
    if (command.startsWith("T:")) {
      int commaPos = command.indexOf(',');
      throttle = command.substring(2, commaPos).toInt();
      elevator = command.substring(commaPos + 3).toInt();
      
      Serial.print("Throttle: ");
      Serial.print(throttle);
      Serial.print(" | Elevator: ");
      Serial.println(elevator);
    }
  }
}
```

**Python script (simulate_transmitter.py):**
```python
import serial
import pygame
import time

pygame.init()
pygame.joystick.init()

if pygame.joystick.get_count() == 0:
    print("No gamepad!")
    exit(1)

joystick = pygame.joystick.Joystick(0)
joystick.init()

# Open serial port to Arduino
try:
    ser = serial.Serial('COM3', 9600, timeout=1)  # Change COM3 to your Arduino port
    print("Connected to Arduino on COM3")
except:
    print("Failed to open serial port. Check device manager for COM port.")
    exit(1)

time.sleep(2)  # Wait for Arduino to reset

clock = pygame.time.Clock()

try:
    while True:
        for event in pygame.event.get():
            pass
        
        # Read gamepad
        throttle = int(1000 + joystick.get_axis(1) * 500)  # Left stick Y
        elevator = int(1500 + joystick.get_axis(0) * 500)  # Left stick X
        
        # Send to Arduino
        command = f"T:{throttle},E:{elevator}\n"
        ser.write(command.encode())
        
        print(f"Sent: {command.strip()}")
        
        clock.tick(20)  # 20 Hz

except KeyboardInterrupt:
    ser.close()
    print("Serial closed")
```

**What this tests:**
- ✅ Gamepad input is correctly mapped to throttle/elevator
- ✅ Serial communication works (Arduino receives commands)
- ✅ No RF module needed yet
- ✅ Verify latency (should be <50ms per update)

---

#### Step 3: Simulate NRF24L01 Packet Format

**Simulate:** Create dummy packets without actual radio hardware

**Arduino simulation (nrf24_test.ino):**
```cpp
struct ControlData {
  uint16_t throttle;
  uint16_t elevator;
};

void setup() {
  Serial.begin(9600);
}

void loop() {
  if (Serial.available() >= 4) {
    // Read 4 bytes (2 uint16_t values)
    byte data[4];
    Serial.readBytes(data, 4);
    
    // Reconstruct struct
    ControlData control;
    control.throttle = (data[0] << 8) | data[1];
    control.elevator = (data[2] << 8) | data[3];
    
    Serial.print("Throttle: ");
    Serial.print(control.throttle);
    Serial.print(" Elevator: ");
    Serial.println(control.elevator);
  }
}
```

**Python sender (binary format):**
```python
import serial

ser = serial.Serial('COM3', 9600)

throttle = 1500  # uint16_t
elevator = 1500  # uint16_t

# Pack as 4 bytes (big-endian)
data = bytes([
    (throttle >> 8) & 0xFF,
    throttle & 0xFF,
    (elevator >> 8) & 0xFF,
    elevator & 0xFF
])

ser.write(data)
print(f"Sent 4-byte packet: {data.hex()}")
```

---

### 3.3 Hardware Simulation (Breadboard Test Before Soldering)

**Phase 1: Test Power Rails Only**

```
┌──────────────┐
│ USB 5V from  │
│ Laptop       │
└──────┬───────┘
       │
       ▼
   [Breadboard]
   │
   ├─ Power row (+5V)
   ├─ GND row (-)
   │
   ├─→ Arduino Uno (powered via USB)
   ├─→ NRF24L01 (via breadboard 3.3V)
   └─→ Multimeter probes at key nodes
```

**Checklist:**
- [ ] 3.3V line shows 3.3V (with no load)
- [ ] 5V line shows 5V
- [ ] No shorts between rails
- [ ] GND is continuous

**Phase 2: Test Arduino + NRF24 (Breadboard)**

```
Breadboard layout:
┌─────────────────────────────────────┐
│ Arduino Uno (USB powered)           │
│ D7, D8, D11, D12, D13 → Breadboard  │
│ 3.3V pin → Breadboard               │
│ GND → Breadboard                    │
│                                     │
│ NRF24L01 (4 pins on breadboard)     │
│ VCC ← 3.3V rail                     │
│ GND ← GND rail                      │
│ CE ← D7                             │
│ CSN ← D8                            │
│ MOSI ← D11                          │
│ MISO ← D12                          │
│ SCK ← D13                           │
└─────────────────────────────────────┘
```

**Test code (Arduino):**
```cpp
#include <SPI.h>
#include <RF24.h>

RF24 radio(7, 8);  // CE, CS

void setup() {
  Serial.begin(9600);
  if (radio.begin()) {
    Serial.println("NRF24 initialized OK");
  } else {
    Serial.println("NRF24 FAILED");
  }
}

void loop() {}
```

**Upload and check Serial Monitor:**
- If you see "NRF24 initialized OK" → your SPI is correct!
- If "FAILED" → check wiring

---

### 3.4 Complete Bench Test (Before Flying)

**Setup:**
- Arduino Uno + NRF24 on breadboard (transmitter)
- ESP32 + NRF24 on breadboard (receiver, no battery yet)
- Windows laptop with gamepad + Python script

**Test sequence:**

| Step | What | Expected Result | Pass/Fail |
|------|------|-----------------|-----------|
| 1 | Plug gamepad into laptop | Appears in device manager | |
| 2 | Run Python gamepad test script | Detects gamepad input | |
| 3 | Arduino connected via USB | Serial Monitor opens | |
| 4 | NRF24 on Arduino powers on | LED on NRF24 blinks | |
| 5 | NRF24 on ESP32 powers on | LED on NRF24 blinks | |
| 6 | Run Python transmitter script | Arduino receives serial data | |
| 7 | Arduino sends via NRF24 | Oscilloscope shows SPI activity | |
| 8 | ESP32 receives RF packet | LED on ESP32 blinks | |
| 9 | Gamepad stick → throttle PWM | GPIO 26 shows PWM (oscilloscope) | |
| 10 | Gamepad stick → servo PWM | GPIO 25 shows PWM (oscilloscope) | |

---

## 4. COMPLETE TEST ROADMAP (No Hardware Yet)

### Week 1: Software Simulation

```
Day 1: Install and test gamepad reader
  └─ Run test_gamepad.py
  └─ Confirm pygame detects your Antesports controller
  └─ Map Left Stick Y to throttle (0-1023 range)
  └─ Map Right Stick X to elevator (0-1023 range)

Day 2-3: Test serial communication
  └─ Upload Arduino sketch to Uno
  └─ Run Python serial sender
  └─ Verify Arduino receives and parses throttle/elevator
  └─ Monitor latency with timing prints

Day 4-5: Test NRF24 initialization
  └─ Assemble NRF24 on breadboard
  └─ Upload NRF24 test sketch
  └─ Verify "NRF24 initialized OK" message
  └─ Check NRF24 antenna position

Day 6-7: Full integration test (no aircraft)
  └─ Gamepad → Laptop → Arduino → NRF24
  └─ Verify end-to-end latency <100ms
  └─ Test failsafe: kill transmitter, check receiver goes neutral
```

### Week 2: Hardware Assembly & Bench Testing

```
Day 8: Solder power components
  └─ 3.3V LDO regulator
  └─ Capacitors (100µF, 100nF)
  └─ Battery connector to ESC

Day 9: Assemble airframe
  └─ Foam board structure
  └─ Motor + prop mount
  └─ Servo mount
  └─ Battery holder

Day 10: Full system bench test
  └─ **DO NOT ATTACH PROPELLER YET**
  └─ Power on, check no burning smell
  └─ Send throttle commands, verify motor spins smoothly
  └─ Send elevator commands, verify servo moves

Day 11-14: Flight preparation
  └─ Attach propeller (balanced)
  └─ Test Wi-Fi/RF range outdoors
  └─ Program failsafe
  └─ Final safety check
```

---

## 5. TOOLS NEEDED FOR SIMULATION & TESTING

### Software (All Free)

```
Essential:
├─ Arduino IDE v2.x (arduino.cc) - for uploading code
├─ Python 3.9+ (python.org) - for gamepad script
├─ Pygame (pip install pygame) - gamepad input
├─ pyserial (pip install pyserial) - serial communication
├─ NRF24 Arduino library (via Library Manager)
└─ Notepad++ or VS Code - code editing

Optional (for visualization):
├─ Tinkercad Circuits (tinkercad.com) - circuit diagram
├─ Fritzing (fritzing.org) - breadboard layout diagrams
└─ SerialPlotter (built into Arduino IDE) - real-time data graphing
```

### Hardware (For Bench Testing)

```
Essential:
├─ Breadboard (400-hole, $2)
├─ Jumper wires (pack of 40, $2)
├─ USB cables (A-B for Arduino, micro for ESP32)
├─ Multimeter (for voltage checking, $8)
└─ Oscilloscope (optional but highly recommended for PWM verification, $50+)

Nice to have:
├─ Logic analyzer (16-channel, $20-30, for SPI debugging)
├─ Alligator clip test leads (for breadboard probing)
└─ Barrel jack adapters (for clean power distribution)
```

---

## 6. RECOMMENDED SIMULATION SEQUENCE

### **Do This in Order:**

1. **✅ Install software (30 min)**
   - Arduino IDE
   - Python + Pygame + pyserial
   - Download NRF24 libraries

2. **✅ Test gamepad detection (30 min)**
   - Run `test_gamepad.py`
   - Confirm your Antesports controller works
   - Map axes to throttle/elevator

3. **✅ Test serial communication (1 hour)**
   - Upload basic serial receiver sketch to Arduino Uno
   - Run Python sender script
   - Watch Arduino print received throttle/elevator values

4. **✅ Test NRF24 initialization (1 hour)**
   - Wire NRF24 on breadboard
   - Upload RF24 test sketch
   - Verify initialization message

5. **✅ Full transmitter/receiver test (2 hours)**
   - Both Arduino and ESP32 with NRF24
   - Gamepad → Laptop → Arduino → RF → ESP32
   - Measure end-to-end latency

6. **✅ Motor + servo control simulation (1 hour)**
   - Upload ESP32 PWM code
   - Send throttle commands and verify GPIO 26/25 output PWM
   - Use oscilloscope to check pulse widths

7. **✅ Bench test with power (2 hours)**
   - Connect 1S LiPo to ESC + motor (no prop)
   - Send throttle commands from gamepad
   - Verify motor spins smoothly
   - Test servo deflection under power

8. **✅ Pre-flight checklist (1 hour)**
   - Attach propeller
   - Test indoors in controlled space
   - Verify failsafe (loss of signal = motor stops)

---

## 7. EXPECTED TIMELINES

| Phase | Task | Time | Hardware Needed |
|-------|------|------|-----------------|
| Simulation | Software install + gamepad test | 1 hour | Laptop, gamepad |
| Simulation | Serial communication test | 1.5 hours | Laptop, Arduino Uno, USB cable |
| Simulation | NRF24 initialization | 1 hour | Arduino, NRF24, breadboard, wires |
| Simulation | Full integration test | 2 hours | All components, oscilloscope (optional) |
| Hardware | Component soldering | 2 hours | Soldering iron, solder, components |
| Hardware | Airframe assembly | 3-4 hours | Frame materials, hot glue, motor mount |
| Hardware | Bench test (no prop) | 1.5 hours | All electronics, multimeter |
| Hardware | Pre-flight (with prop) | 1 hour | Balanced propeller, failsafe testing |
| **Total** | | **~14-16 hours** | |

---

## 8. SIMULATION CHECKLIST (Print This Out)

```
GAMEPAD & SERIAL COMMUNICATION
☐ Pygame detects gamepad
☐ Left stick Y mapped to throttle (1000-2000 µs)
☐ Right stick X mapped to elevator (1000-2000 µs)
☐ Serial connection to Arduino at 9600 baud
☐ Arduino receives throttle/elevator values
☐ Latency verified <50ms per command

NRF24L01 MODULES
☐ NRF24 on Arduino detects via SPI
☐ NRF24 on ESP32 detects via SPI
☐ Both modules power on without errors
☐ RF24 library compiles without warnings

WIRELESS TRANSMISSION
☐ Arduino transmits RF24 packets
☐ ESP32 receives RF24 packets
☐ Packet contents match sent values (throttle, elevator)
☐ Failsafe works: 500ms timeout = neutral commands

PWM GENERATION
☐ ESP32 GPIO 26 outputs 50 Hz PWM (ESC)
☐ ESP32 GPIO 25 outputs 50 Hz PWM (Servo)
☐ PWM pulse width range: 1000-2000 µs
☐ PWM frequency stays at 50 Hz under load

POWER DISTRIBUTION
☐ 3.3V line stable at 3.3V (no load)
☐ 5V line stable at 5V (no load)
☐ No shorts between power rails
☐ Capacitors in place (100µF bulk, 100nF ceramic)

BENCH TEST (WITH HARDWARE)
☐ Motor spins smoothly on bench (no propeller)
☐ Throttle commands control motor speed
☐ Servo deflects full range (±45° or similar)
☐ No burning smell or excessive heat
☐ Current draw reasonable (<2A idle)
☐ Battery voltage stable during motor spin
```

---

## 9. TROUBLESHOOTING SIMULATION ISSUES

### Issue: Gamepad not detected by Pygame

**Causes:**
- Gamepad not paired / connected
- Pygame not installed correctly
- USB driver missing

**Fix:**
```bash
# Reinstall pygame
pip uninstall pygame
pip install pygame --upgrade

# Verify installation
python -c "import pygame; print(pygame.ver)"
```

---

### Issue: Arduino not receiving serial data

**Causes:**
- Wrong COM port in Python script
- Baud rate mismatch (should be 9600)
- USB driver issue

**Fix:**
```
1. Open Arduino IDE → Tools → Port
2. Note the COM port (e.g., COM3)
3. Update Python script: ser = serial.Serial('COM3', 9600)
4. Try different baud rates (115200 if 9600 fails)
```

---

### Issue: NRF24 reports initialization failed

**Causes:**
- SPI pins wired incorrectly
- NRF24 powered by wrong voltage (should be 3.3V)
- Antenna not fully inserted
- Defective module

**Fix:**
```
1. Check SPI wiring: D13=SCK, D12=MISO, D11=MOSI
2. Verify 3.3V voltage at NRF24 VCC pin
3. Fully insert antenna into SMA connector
4. Try a different NRF24 module if available
```

---

### Issue: Latency >200ms

**Causes:**
- Python script running at <10 Hz
- Serial baud rate too low
- Too much delay() in Arduino loop

**Fix:**
```cpp
// Arduino: Use millis() instead of delay()
if (millis() - lastUpdate > 50) {  // 20 Hz
    // Process command
    lastUpdate = millis();
}
```

---

## 10. FINAL SUMMARY

### You Will Have Tested Before Building:

✅ Gamepad input reading  
✅ Serial communication (Laptop ↔ Arduino)  
✅ RF24 packet transmission (Arduino ↔ ESP32)  
✅ PWM signal generation (correct frequency + pulse width)  
✅ Motor control response  
✅ Servo control response  
✅ Failsafe timeout behavior  
✅ Power distribution stability  
✅ Full system latency (<100ms)  

### Total Simulation Time: ~8-10 hours (spread over 1-2 weeks)

### Confidence Level After Simulation: **95%+**

---

**Ready to proceed with simulation, or do you have questions about any part?**

