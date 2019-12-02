// PINS
const int HAPTIC_PIN = A3;
const int DOWNBEAT_LED_PIN = 11;
const int LED_PIN = 9;
const int BUTTON_PIN = 2;

unsigned long lastMillis;

bool initialized = true; // device just started

// Button State Management
int buttonPushCounter = 0;                  // counter for the number of button presses
int buttonState;                            // current state of the button
int lastButtonState = LOW;                  // previous state of the button
unsigned long lastButtonStateIntervals[4] = {0, 0, 0, 0};  // the time interval between each button press

/* Metronome State Management
   States:
   0 - off / deactivated
   1 - on / activated
*/
int metronomeState = 0;
int metronomeBeat = LOW;
unsigned long metronomeBeatTempo; // the average time between each beat
unsigned long previousMillis = 0;
int metronomeBeatCounter = 0;

void setup() {
  Serial.begin(9600);

  pinMode(DOWNBEAT_LED_PIN, OUTPUT);
  pinMode(LED_PIN, OUTPUT);
  pinMode(HAPTIC_PIN, OUTPUT);
  pinMode(BUTTON_PIN, INPUT);

  lastMillis = millis();
}

void loop() {
  unsigned long currentMillis = millis();
  int elapsed = millis() - lastMillis;

  buttonState = digitalRead(BUTTON_PIN);

  // compare the buttonState to its previous state
  if (buttonState != lastButtonState) {
    // once button is pushed, deviced is no longer in initial state
    if (initialized == true) {
      initialized = false;
    }
    if (buttonState == HIGH) {
      // if the current state is HIGH then the button went from off to on:
      
      // reset data and disable metronome if this is the first of 4 pushes
      if (buttonPushCounter == 0) {
        resetIntervals();
        metronomeBeatTempo = 0;
        metronomeState = false;
        metronomeBeat = LOW;
        
        digitalWrite(LED_PIN, metronomeBeat);
        digitalWrite(DOWNBEAT_LED_PIN, metronomeBeat);
        digitalWrite(HAPTIC_PIN, metronomeBeat);
      }
      
      // track button pushes
      buttonPushCounter++;

      // record the time that elapsed since the previous push
      lastButtonStateIntervals[buttonPushCounter - 1] = millis();

      Serial.println("Button: Pushed");
      Serial.print("Number of button pushes: ");
      Serial.println(buttonPushCounter);
      Serial.print("Time of press: ");
      Serial.println(millis());
      Serial.print("Button Intervals: [");
      for (byte i = 0; i < 4; i++) {
        Serial.print(lastButtonStateIntervals[i]);
        Serial.print(", ");
      }
      Serial.println("]");
    } else {
      // if the current state is LOW then the button went from on to off:
      Serial.println("Button: Released");
    }

    // Delay a little bit to avoid bouncing
    // TODO: replace with a millis check
    delay(25);
  }
  // save the current state as the last state, for next time through the loop
  lastButtonState = buttonState;

  if (initialized) {
    // set initial state behaviour if button has not been pushed since being powered on
    digitalWrite(LED_PIN, HIGH);
    digitalWrite(DOWNBEAT_LED_PIN, HIGH);
  } else {
    // turns on the LED every four button pushes by checking the modulo of the
    // button push counter. the modulo function gives you the remainder of the
    // division of two numbers:
    if (buttonPushCounter == 4) {
      // calculate and average intervals
        unsigned long intervalSum = 0;
        for (byte i = 1; i < 4; i++) {
          intervalSum += lastButtonStateIntervals[i] - lastButtonStateIntervals[i - 1];
        }
        // calculate the average tempo
        metronomeBeatTempo = intervalSum / 3;
        // initialize metronome
        metronomeState = true;
        
        Serial.print("Interval Sum: ");
        Serial.println(intervalSum);
        Serial.print("Beat Tempo: ");
        Serial.println(metronomeBeatTempo);
        
        // Reset counter
        buttonPushCounter = 0; // reset button counter

    } else {
      // digitalWrite(LED_PIN, LOW); 
    }
    
    if (metronomeState == true) {
      
      if (currentMillis - previousMillis >= (metronomeBeatTempo / 2)) {
        // keep track of beat counts to denote the downbeat
        metronomeBeatCounter++;
          
        // save the last time you blinked the LED
        previousMillis = currentMillis;
    
        // if the LED is off turn it on and vice-versa:
        metronomeBeat = metronomeBeat == LOW ? HIGH : LOW;
    
        // set the LED with the ledState of the variable:
        if (metronomeBeatCounter == 1) {
          digitalWrite(DOWNBEAT_LED_PIN, metronomeBeat);
        } else {
          digitalWrite(LED_PIN, metronomeBeat);
        }
        digitalWrite(HAPTIC_PIN, metronomeBeat);
        
        Serial.println(metronomeBeatCounter);
          if (metronomeBeatCounter == 8) {
            metronomeBeatCounter = 0;
          }
  
        
      }
    }
  }

  lastMillis = millis();
}

void resetIntervals() {
  for (byte i = 0; i < 4; i++) {
    lastButtonStateIntervals[i] = 0;
  }
}
