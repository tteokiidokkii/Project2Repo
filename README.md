# Project2Repo

const int servoPin  = 9;   // servo signal
const int switchPin = 2;   // copper switch input

// Servo movement control
int servoPos = 0;                       // current servo angle
const int minAngle = 0;
const int maxAngle = 180;
const unsigned long stepInterval = 15;  // ms between steps (speed control)

unsigned long lastStepTime = 0;

// States: what the servo is doing
enum ServoState {
  IDLE,
  MOVING_UP,
  MOVING_DOWN
};

ServoState currentState = IDLE;

// For detecting when the switch is newly pressed
int lastSwitchState = HIGH;  // because of INPUT_PULLUP

void setup() {
  myServo.attach(servoPin);
  myServo.write(servoPos);

  pinMode(switchPin, INPUT_PULLUP);  // uses internal pull-up
}

void loop() {
  unsigned long now = millis();

  // --- READ COPPER SWITCH (non-blocking) ---
  int switchState = digitalRead(switchPin);

  // Detect a "press" when it goes from HIGH -> LOW
  bool switchJustPressed = (lastSwitchState == HIGH && switchState == LOW);
  lastSwitchState = switchState;

  // If the switch is just pressed and we are idle, start a sweep
  if (switchJustPressed && currentState == IDLE) {
    currentState = MOVING_UP;
  }

  // --- SERVO STATE MACHINE (non-blocking) ---
  if (currentState == MOVING_UP) {
    if (now - lastStepTime >= stepInterval) {
      lastStepTime = now;
      if (servoPos < maxAngle) {
        servoPos++;
        myServo.write(servoPos);
      } else {
        // Reached top, go back down
        currentState = MOVING_DOWN;
      }
    }
  } else if (currentState == MOVING_DOWN) {
    if (now - lastStepTime >= stepInterval) {
      lastStepTime = now;
      if (servoPos > minAngle) {
        servoPos--;
        myServo.write(servoPos);
      } else {
        // Back to start, go idle
        currentState = IDLE;
      }
    }
  }

