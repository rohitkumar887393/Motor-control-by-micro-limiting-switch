# Motor-control-by-micro-limiting-switch
If switch  pressed then motor starts rotating ask for direction to rotate again


#include <Arduino.h>

// Define pins
#define MOTOR_PWM PA8           // PWM pin for motor speed control
#define MOTOR_DIR PA1           // Direction control pin
#define SWITCH_PIN PA0          // Micro switch input pin (NC)

// Variables
volatile bool switchPressed = false;  // State of the switch
bool motorRunning = false;            // Tracks if the motor is running
char motorDirection = 'F';            // Motor direction ('F' for forward, 'R' for reverse)
int pwm = 100;                        // Motor speed (adjustable)

// Interrupt Service Routine (ISR) for the micro switch
void switchISR() {
  switchPressed = digitalRead(SWITCH_PIN); // Update the state of the switch
  if (switchPressed) {
    motorRunning = false; // Stop the motor when the switch is pressed
  }
}

void setup() {
  // Set up motor control pins
  pinMode(MOTOR_PWM, OUTPUT);
  pinMode(MOTOR_DIR, OUTPUT);

  // Set up the micro switch pin with an internal pull-up resistor
  pinMode(SWITCH_PIN, INPUT_PULLUP);

  // Attach an interrupt to the micro switch
  attachInterrupt(digitalPinToInterrupt(SWITCH_PIN), switchISR, CHANGE);

  // Initialize motor
  analogWrite(MOTOR_PWM, 0); // Motor initially stopped

  // Start serial communication for direction control
  Serial.begin(9600);
  Serial.println("Enter motor direction: 'F' for forward, 'R' for reverse");
}

void loop() {
  // Check for serial input
  if (Serial.available() > 0) {
    char input = Serial.read();
    input = toupper(input); // Convert to uppercase for consistency
    if (input == 'F' || input == 'R') {
      motorDirection = input; // Update motor direction
      motorRunning = true;    // Allow motor to run again
      Serial.print("Motor direction set to: ");
      Serial.println(motorDirection == 'F' ? "Forward" : "Reverse");
    } else {
      Serial.println("Invalid input! Enter 'F' or 'R'.");
    }
  }

  // Motor control logic
  if (motorRunning) {
    // Rotate the motor in the specified direction
    if (motorDirection == 'F') {
      digitalWrite(MOTOR_DIR, HIGH); // Forward
    } else if (motorDirection == 'R') {
      digitalWrite(MOTOR_DIR, LOW);  // Reverse
    }
    analogWrite(MOTOR_PWM, pwm); // Set motor speed
  } else {
    // Stop the motor
    analogWrite(MOTOR_PWM, 0);
  }
}
