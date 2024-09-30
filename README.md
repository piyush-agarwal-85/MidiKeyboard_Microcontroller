# MidiKeyboard_Microcontroller
Making a Midi Keyboard with Microcontroller Arduino UNO and old Synthesiser
![IMG_20230925_233524](https://github.com/user-attachments/assets/8e7b1e11-bb21-4dbe-a6be-b6905e39d149)
![IMG_20230925_233532](https://github.com/user-attachments/assets/6fbc7572-3909-4542-96f4-e5f0cca7a334)
<h2>References</b></h2>
https://docs.arduino.cc/tutorials/communication/guide-to-shift-out/

https://youtu.be/qVPsnqUbu6M?feature=shared



```
// Pin Definitions
// Rows are connected to
const int row1 = 5;
const int row2 = 6;
const int row3 = 7;
const int row4 = 8;
const int row5 = 

// The 74HC595 uses a serial communication 
// link which has three pins
const int clock = 9;
const int latch = 10;
const int data = 11;


uint8_t keyToMidiMap[44];

boolean keyPressed[44];

int noteVelocity = 127;


// use prepared bit vectors instead of shifting bit left everytime
int bits[] = { B00000001, B00000010, B00000100, B00001000, B00010000, B00100000, B01000000, B10000000 };


// 74HC595 shift to next column
void scanColumn(int value) {
	digitalWrite(latch, LOW); //Pulls the chips latch low
	shiftOut(data, clock, MSBFIRST, value); //Shifts out the 8 bits to the shift register
	digitalWrite(latch, HIGH); //Pulls the latch high displaying the data
}

void setup() {
	
	// Map scan matrix buttons/keys to actual Midi note number. Lowest num 41 corresponds to F MIDI note.
	keyToMidiMap[0] = 48;
	keyToMidiMap[1] = 41;
	keyToMidiMap[2] = 43;
	keyToMidiMap[3] = 42;
	keyToMidiMap[4] = 44;
	keyToMidiMap[5] = 45;
	keyToMidiMap[6] = 46;
	keyToMidiMap[7] = 47;

	keyToMidiMap[8] = 56;
	keyToMidiMap[1 + 8] = 49;
	keyToMidiMap[2 + 8] = 51;
	keyToMidiMap[3 + 8] = 50;
	keyToMidiMap[4 + 8] = 52;
	keyToMidiMap[5 + 8] = 53;
	keyToMidiMap[6 + 8] = 54;
	keyToMidiMap[7 + 8] = 55;

	keyToMidiMap[16] = 64;
	keyToMidiMap[1 + 16] = 57;
	keyToMidiMap[2 + 16] = 59;
	keyToMidiMap[3 + 16] = 58;
	keyToMidiMap[4 + 16] = 60;
	keyToMidiMap[5 + 16] = 61;
	keyToMidiMap[6 + 16] = 62;
	keyToMidiMap[7 + 16] = 63;

	keyToMidiMap[24] = 72;
	keyToMidiMap[1 + 24] = 65;
	keyToMidiMap[2 + 24] = 67;
	keyToMidiMap[3 + 24] = 66;
	keyToMidiMap[4 + 24] = 68;
	keyToMidiMap[5 + 24] = 69;
	keyToMidiMap[6 + 24] = 70;
	keyToMidiMap[7 + 24] = 71;

  	keyToMidiMap[32] = 80;
	keyToMidiMap[1 + 32] = 73;
	keyToMidiMap[2 + 32] = 75;
	keyToMidiMap[3 + 32] = 74;
	keyToMidiMap[4 + 32] = 76;
	keyToMidiMap[5 + 32] = 77;
	keyToMidiMap[6 + 32] = 78;
	keyToMidiMap[7 + 32] = 79;

	// setup pins output/input mode
	pinMode(data, OUTPUT);
	pinMode(clock, OUTPUT);
	pinMode(latch, OUTPUT);

	pinMode(row1, INPUT);
	pinMode(row2, INPUT);
	pinMode(row3, INPUT);
	pinMode(row4, INPUT);
  pinMode(row5, INPUT);  

    Serial.begin(38400);

	delay(1000);

}

void loop() {

	for (int col = 0; col < 8; col++) {
		
		// shift scan matrix to following column
		scanColumn(bits[col]);

		// check if any keys were pressed - rows will have HIGH output in this case corresponding
		int groupValue1 = digitalRead(row1);
		int groupValue2 = digitalRead(row2);
		int groupValue3 = digitalRead(row3);
		int groupValue4 = digitalRead(row4);
    int groupValue5 = digitalRead(row5);

		// process if any combination of keys pressed
		if (groupValue1 != 0 || groupValue2 != 0 || groupValue3 != 0
				|| groupValue4 != 0 || groupValue5 != 0) {

			if (groupValue1 != 0 && !keyPressed[col]) {
				keyPressed[col] = true;
				noteOn(0x91, keyToMidiMap[col], noteVelocity);
			}

			if (groupValue2 != 0 && !keyPressed[col + 8]) {
				keyPressed[col + 8] = true;
				noteOn(0x91, keyToMidiMap[col + 8], noteVelocity);
			}

			if (groupValue3 != 0 && !keyPressed[col + 16]) {
				keyPressed[col + 16] = true;
				noteOn(0x91, keyToMidiMap[col + 16], noteVelocity);
			}

			if (groupValue4 != 0 && !keyPressed[col + 24]) {
				keyPressed[col + 24] = true;
				noteOn(0x91, keyToMidiMap[col + 24], noteVelocity);
			}

      if (groupValue5 != 0 && !keyPressed[col + 32]) {
				keyPressed[col + 32] = true;
				noteOn(0x91, keyToMidiMap[col + 32], noteVelocity);
			}

		}

		//  process if any combination of keys released
		if (groupValue1 == 0 && keyPressed[col]) {
			keyPressed[col] = false;
			noteOn(0x91, keyToMidiMap[col], 0);
		}

		if (groupValue2 == 0 && keyPressed[col + 8]) {
			keyPressed[col + 8] = false;
			noteOn(0x91, keyToMidiMap[col + 8], 0);
		}

		if (groupValue3 == 0 && keyPressed[col + 16]) {
			keyPressed[col + 16] = false;
			noteOn(0x91, keyToMidiMap[col + 16], 0);
		}

		if (groupValue4 == 0 && keyPressed[col + 24]) {
			keyPressed[col + 24] = false;
			noteOn(0x91, keyToMidiMap[col + 24], 0);
		}

    if (groupValue5 == 0 && keyPressed[col + 32]) {
			keyPressed[col + 32] = false;
			noteOn(0x91, keyToMidiMap[col + 32], 0);
		}

	}

}


void noteOn(int cmd, int pitch, int velocity) {
  	Serial.write(cmd);
	Serial.write(pitch);
	Serial.write(velocity);
}
```
