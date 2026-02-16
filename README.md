# RDAC


// XIAO RP2040 - 50 Hz PWM Example



const int pwmInputPin_1 = D1;
volatile uint32_t riseTime_1 = 0;
volatile uint32_t pulseWidth_1 = 0;

const int pwmInputPin_2 = D2;
volatile uint32_t riseTime_2 = 0;
volatile uint32_t pulseWidth_2 = 0;

const int rightPin = D3;   // Choose any PWM-capable pin (e.g., D3)
const int leftPin = D4;   // Choose any PWM-capable pin (e.g., D4)
const int pwmFreq = 100; // 100 Hz is the minimum
const int pwmResolution = 16; // 16-bit resolution

uint16_t duty = round(6553.5*1); // 50% of 65535 (16-bit)
uint16_t dutyRight = 6554;
uint16_t dutyLeft = 6554;

int steering = 0;
int bracking = 0;

int right;
int left;

bool brackingValid = 0;
bool steeringValid = 0;


void setup() {

  Serial.begin(115200);
  delay(50);

  pinMode(pwmInputPin_1, INPUT);
  attachInterrupt(digitalPinToInterrupt(pwmInputPin_1), pwmISR_1, CHANGE);
  pinMode(pwmInputPin_2, INPUT);
  attachInterrupt(digitalPinToInterrupt(pwmInputPin_2), pwmISR_2, CHANGE);


  analogWriteFreq(pwmFreq);        // Set PWM frequency
  analogWriteResolution(pwmResolution); // Set resolution
  
  
}

void loop() {
    



    
    

    if(pulseWidth_1 != 0){
      if(pulseWidth_1 < 1000){pulseWidth_1 = 1000;}
      if(pulseWidth_1 > 2000){pulseWidth_1 = 2000;}
      bracking = pulseWidth_1 - 1000;
      brackingValid = 1;
      }
    if(pulseWidth_2 != 0){
      if(pulseWidth_2 < 1000){pulseWidth_2 = 1000;}
      if(pulseWidth_2 > 2000){pulseWidth_2 = 2000;}
      steering = pulseWidth_2 - 1500;
      steeringValid = 1;
      }

    bool mix = (1000 + (bracking + abs(steering))) < 2003;

    if (mix){
      if(steering > 0){
        right = 1000 + bracking + steering;
        left = 1000 + bracking;
      }else{
        right = 1000 + bracking;
        left = 1000 + bracking - steering;
      }
      
    }else{
      if(steering > 0){
        right = 1000 + bracking;
        left = 1000 + bracking - steering;
      }else{
        right = 1000 + bracking + steering;
        left = 1000 + bracking;
      }
    }

    dutyRight = round(6.5535 * right);
    dutyLeft = round(6.5535 * left);

    //if(brackingValid || steeringValid){
    if(1){
      analogWrite(rightPin, dutyRight);
      analogWrite(leftPin, dutyLeft);
    }
/*
    static uint32_t lastPrint = 0;
    if (millis() - lastPrint > 500) {
      lastPrint = millis();

      noInterrupts();
      uint32_t width = pulseWidth_1;
      interrupts();

      Serial.print("R: ");
      Serial.println(mix);
      Serial.print("L: ");
      Serial.println(left);
    }*/
    



}


  void pwmISR_1() {
    if (digitalRead(pwmInputPin_1) == HIGH) {
      riseTime_1 = micros();
    } else {
      uint32_t pulseWidth = micros() - riseTime_1;
      if (pulseWidth > 990 && pulseWidth < 2010 ){
        pulseWidth_1 = pulseWidth;
      }else{pulseWidth_1 = 0;}
    }
  }

    void pwmISR_2() {
    if (digitalRead(pwmInputPin_2) == HIGH) {
      riseTime_2 = micros();
    } else {
      uint32_t pulseWidth = micros() - riseTime_2;
      if (pulseWidth > 990 && pulseWidth < 2010 ){
        pulseWidth_2 = pulseWidth;
      }else{pulseWidth_2 = 0;}
    }
  }
