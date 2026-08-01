const int SETPOINT = 341;         // Recalculated using measured Vref=5.06V
const int OCR1A_MIN = 0;
const int OCR1A_MAX = 511;
const int MAX_STEP = 5;

float Kp = 0.05;
float Ki = 0.01;
float Kd = 0.01;

float integral = 0;
float lastError = 0;
unsigned long lastTime = 0;

int readFilteredADC() {
  // Average 8 samples to reduce switching noise on the reading
  long sum = 0;
  for (int i = 0; i < 8; i++) {
    sum += analogRead(A0);
  }
  return sum / 8;
}

void setup() {
  Serial.begin(9600);

  DDRB |= (1 << 1);        // Pin 9 as output

  TCCR1A = (1 << COM1A1) | (1 << WGM11);
  TCCR1B = (1 << WGM13) | (1 << WGM12) | (1 << CS10);

  ICR1 = 511;
  OCR1A = 213;

  lastTime = millis();
}

void loop() {
  unsigned long now = millis();
  float dt = (now - lastTime) / 1000.0;

  if (dt >= 0.01) {
    int reading = readFilteredADC();
    float error = SETPOINT - reading;

    integral += error * dt;
    integral = constrain(integral, -200, 200);

    float derivative = (error - lastError) / dt;

    float output = Kp * error + Ki * integral + Kd * derivative;

    int desiredDuty = OCR1A + (int)output;
    desiredDuty = constrain(desiredDuty, OCR1A_MIN, OCR1A_MAX);

    int change = desiredDuty - OCR1A;
    change = constrain(change, -MAX_STEP, MAX_STEP);
    OCR1A = OCR1A + change;

    Serial.print("Reading: "); Serial.print(reading);
    Serial.print("  Error: "); Serial.print(error);
    Serial.print("  OCR1A: "); Serial.println(OCR1A);

    lastError = error;
    lastTime = now;
  }
}
