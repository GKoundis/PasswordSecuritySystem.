#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Keypad.h>
#include <SevSeg.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

const byte ROWS = 4; // four rows
const byte COLS = 4; // four columns
char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};
byte pin_rows[ROWS] = {9, 8, 7, 6}; // connect to the row pinouts of the keypad
byte pin_column[COLS] = {5, 4, 3, 2}; // connect to the column pinouts of the keypad

Keypad keypad = Keypad(makeKeymap(keys), pin_rows, pin_column, ROWS, COLS);
SevSeg sevseg;
LiquidCrystal_I2C lcd(0x27, 16, 2);

const int buzzerPin = 10;
const int greenLEDPin = 0;
const int redLEDPin = 11;

// OLED-related constants
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1 // Reset pin # (or -1 if sharing Arduino reset pin)

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

int password[] = {2, 0, 0, 5};
int enteredDigits[4];
int digitCount = 0;

void setup()
{
  sevseg.begin(COMMON_CATHODE, 4, pin_column, pin_rows, false, false, false, false);
  sevseg.setBrightness(90);

  pinMode(buzzerPin, OUTPUT);
  pinMode(greenLEDPin, OUTPUT);
  pinMode(redLEDPin, OUTPUT);

  digitalWrite(greenLEDPin, LOW);
  digitalWrite(redLEDPin, LOW);

  lcd.begin(16, 2);
  lcd.print("Enter text:");

  // Initialize the OLED display
  if (!display.begin(0x3C, OLED_RESET))
  {
    Serial.println(F("SSD1306 allocation failed"));
    for (;;)
      ;
  }
  display.display(); // Show initial display buffer contents on the screen
  delay(2000);       // Pause for 2 seconds
  display.clearDisplay(); // Clear the buffer
}

void loop()
{
  char key = keypad.getKey();

  if (key)
  {
    if (key == '#')
    {
      // Check the entered password
      if (checkPassword())
      {
        digitalWrite(greenLEDPin, HIGH);
        digitalWrite(redLEDPin, LOW);
        playSuccessSound();
      }
      else
      {
        digitalWrite(greenLEDPin, LOW);
        digitalWrite(redLEDPin, HIGH);
        playErrorSound();
      }

      // Clear entered digits
      clearEnteredDigits();

      // Clear the OLED display
      display.clearDisplay();
    }
    else
    {
      // Store the entered digit and update the display
      if (digitCount < 4)
      {
        enteredDigits[digitCount] = key - '0';
        digitCount++;
        updateDisplay();
      }

      // Print the pressed key on the LCD
      lcd.setCursor(digitCount - 1, 1);
      lcd.print(key);
    }
  }
}

bool checkPassword()
{
  for (int i = 0; i < 4; i++)
  {
    if (enteredDigits[i] != password[i])
    {
      return false;
    }
  }
  return true;
}

void clearEnteredDigits()
{
  for (int i = 0; i < 4; i++)
  {
    enteredDigits[i] = 0;
  }
  digitCount = 0;
  updateDisplay();
}

void updateDisplay()
{
  sevseg.setNumber(enteredDigits, 4);
  sevseg.refreshDisplay();

  // Update OLED display with entered digits
  display.clearDisplay();
  display.setTextSize(2);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 0);

  for (int i = 0; i < digitCount; i++)
  {
    display.print(enteredDigits[i]);
  }

  display.display();
}

void playSuccessSound()
{
  tone(buzzerPin, 1000, 500); // Play a success sound for 500 milliseconds
  delay(1000);                // Wait for 1 second
  noTone(buzzerPin);          // Stop the buzzer
}

void playErrorSound()
{
  tone(buzzerPin, 200, 1000); // Play an error sound for 1 second
  delay(1000);                // Wait for 1 second
  noTone(buzzerPin);          // Stop the buzzer
}
