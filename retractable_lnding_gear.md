# Arduino Motor Direction Control via FS-iA6B Receiver and L298N

This project allows you to control the *direction of a DC motor* using a *3-position switch* on an *FS-iA6B-compatible transmitter (like RadioMaster TX12), with an **Arduino Uno, **L298N motor driver, and **PWM signal from the receiver*.

## 🎯 Features
- 3-way switch controls motor direction:
  - *Switch Up: Motor spins **anticlockwise (ACW)*
  - *Switch Middle: Motor is **stopped*
  - *Switch Down: Motor spins **clockwise (CW)*

---

## 🧰 Hardware Required
- Arduino Uno
- FS-iA6B Receiver
- L298N Motor Driver Module
- 1x DC Motor
- RadioMaster TX12 or compatible transmitter
- Power source (e.g., 12V battery for motor driver)
- Jumper wires

---

## 🔌 Connections

### 🟢 FS-iA6B Receiver to Arduino:
| Receiver Channel | Arduino Pin | Description              |
|------------------|-------------|--------------------------|
| CH5 (Switch)     | D2          | Reads 3-position switch  |
| GND              | GND         | Common ground            |
| VCC (5V)         | 5V          | Power for receiver       |

> ⚠ *Do not connect the FS-iA6B "B/VCC"* to Arduino unless your receiver supports 5V operation. Most FS-iA6Bs work fine with 5V from Arduino.

---

### 🔴 L298N to Arduino:
| L298N Pin | Arduino Pin | Description                   |
|----------|-------------|-------------------------------|
| IN1      | D9          | Controls motor direction (CW) |
| IN2      | D10         | Controls motor direction (ACW) |
| ENA      | Jumper ON   | Full speed (no PWM speed ctrl)|
| 5V       | N/C         | Leave unconnected (Arduino is already powered separately) |
| GND      | GND         | Common ground with Arduino     |
| 12V      | External power (battery or adapter) |

> ✅ Keep the ENA jumper ON to run motor at full speed.
> ❌ Do not power Arduino from L298N 5V unless you're not using USB.

---

### ⚙ Motor:
- Connect the motor terminals to *OUT1* and *OUT2* of the L298N.

---

## 📟 Arduino Code

```cpp
const int RC_INPUT_PIN = 2;      // Input from CH5 (switch)
const int FORWARD_PWM_PIN = 9;   // IN1 on L298N
const int REVERSE_PWM_PIN = 10;  // IN2 on L298N

const int MOTOR_SPEED = 200;  // Fixed speed (not used if ENA jumper is ON)

const unsigned int FORWARD_THRESHOLD = 1700;  // >1700µs = CW
const unsigned int REVERSE_THRESHOLD = 1300;  // <1300µs = ACW

void setup() {
  Serial.begin(9600);
  pinMode(RC_INPUT_PIN, INPUT);
  pinMode(FORWARD_PWM_PIN, OUTPUT);
  pinMode(REVERSE_PWM_PIN, OUTPUT);

  analogWrite(FORWARD_PWM_PIN, 0);
  analogWrite(REVERSE_PWM_PIN, 0);
}

void loop() {
  unsigned long pulseWidth = pulseIn(RC_INPUT_PIN, HIGH, 25000);

  if (pulseWidth == 0) {
    Serial.println("No pulse detected. Motor stopped.");
    analogWrite(FORWARD_PWM_PIN, 0);
    analogWrite(REVERSE_PWM_PIN, 0);
  } else {
    Serial.print("Pulse Width: ");
    Serial.println(pulseWidth);

    if (pulseWidth > FORWARD_THRESHOLD) {
      Serial.println("Command: Clockwise");
      analogWrite(FORWARD_PWM_PIN, MOTOR_SPEED);
      analogWrite(REVERSE_PWM_PIN, 0);
    }
    else if (pulseWidth < REVERSE_THRESHOLD) {
      Serial.println("Command: Anti-clockwise");
      analogWrite(FORWARD_PWM_PIN, 0);
      analogWrite(REVERSE_PWM_PIN, MOTOR_SPEED);
    }
    else {
      Serial.println("Command: Stop");
      analogWrite(FORWARD_PWM_PIN, 0);
      analogWrite(REVERSE_PWM_PIN, 0);
    }
  }

  delay(50);
}
