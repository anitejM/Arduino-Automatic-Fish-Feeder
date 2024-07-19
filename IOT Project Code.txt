#include <DS3231.h>
#include <LiquidCrystal.h>
#include <EEPROM.h>
#include <Servo.h>             
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
LiquidCrystal_I2C lcd(0x27, 16, 2);

DS3231  rtc(SDA, SCL);

Servo servo_test;        
int countToZero = 0;
int angle = 0;
int sw2 = 3;     
int sw3 = 2;   
int buzz = 10;
int flag = 0;
int val = 0;       
int val1 = 0;       
int val2 = 0;      
int count = 0;
int count1 = 0;
int count2 = 0;
int count3 = 0;
int count4 = 0;
int tempcount = 0;
String submin = "";
int mins = 0;
int flag1 = 0;


void setup()
{
  Serial.begin(9600);
  lcd.init();
  lcd.init();
  lcd.backlight();
  rtc.begin(); 
  pinMode(sw2, INPUT_PULLUP);
  pinMode(sw3, INPUT_PULLUP);
  pinMode(buzz, OUTPUT);

  lcd.setCursor(0, 0);
  lcd.print("   WELCOME     ");
  delay(1000);
  lcd.setCursor(0, 0);
  lcd.print("    AUTOMATIC    ");
  lcd.setCursor(0, 1);
  lcd.print("  FISH FEEDING  ");
  delay(1000);
  lcd.clear();
  //rtc.setTime(19, 16, 0);     
  //rtc.setDate(11, 28, 2019);   
  servo_test.attach(6);     

}

void loop()
{
  lcd.setCursor(0, 0);
  lcd.print("Time:");
  lcd.print(rtc.getTimeStr());
  lcd.setCursor(14, 0);
  lcd.print(tempcount);
  lcd.setCursor(0, 1);
  lcd.print(count1);
  lcd.setCursor(4, 1);
  lcd.print(count2);
  lcd.setCursor(8, 1);
  lcd.print(count3);
  lcd.setCursor(12, 1);
  lcd.print(count4);
  submin = rtc.getTimeStr();
  submin = submin.substring(3, 5);
  mins = submin.toInt();
  flag = 0;
  if (mins == count1 || mins == count2 || mins == count3 || mins == count4)
  {
    if (flag1 == 0)
    {
      digitalWrite(buzz, HIGH);
      delay(500);
      digitalWrite(buzz, LOW);
    }
    servo_test.attach(6);     
    lcd.setCursor(14, 1);
    lcd.print("FD");
    for (angle = 0; angle < 90; angle += 1)  
    {
      servo_test.write(angle);                 
      delay(5);
    }

    for (angle = 90; angle >= 1; angle -= 1) 
    {
      servo_test.write(angle);              
      delay(5);
    }
    flag1 = 1;
    countToZero = mins == count1 ? 1 : (mins == count2 ? 2 : (mins == count3 ? 3 : (mins == count4 ? 4 : 0)));
    gsm(countToZero);
  }
  else
  {
    lcd.setCursor(14, 1);
    lcd.print("--");
    if (flag1 == 1)
    {
      digitalWrite(buzz, LOW);
      servo_test.detach();
      flag1 = 0;
      countToZero = mins == count1 ? 1 : (mins == count2 ? 2 : (mins == count3 ? 3 : (mins == count4 ? 4 : 0)));
      gsm(countToZero);
      delay(50);

    }

  }
  val = digitalRead(sw3);
  if (val == LOW)
  {
    lcd.clear();
    while (1)
    {
      if (flag == 1)
        break;
      lcd.setCursor(0, 0);
      lcd.print("Select Memory ");
      val = digitalRead(sw3);
      if (val == LOW)
      {
        if (count > 3)
        {
          count = 0;
        }
        count++;
        lcd.setCursor(0, 1);
        lcd.print(count);
        delay(200);

      }

      val1 = digitalRead(sw2);
      if (val1 == LOW)
      {
        lcd.clear();
        delay(200);
        while (1)
        {
          lcd.setCursor(0, 0);
          lcd.print("Select Mins ");
          val2 = digitalRead(sw3);
          if (val2 == LOW)
          {
            tempcount++;
            if (tempcount > 59)
            {
              tempcount = 0;
              lcd.setCursor(0, 1);
              lcd.print("  ");
            }
            lcd.setCursor(0, 1);
            lcd.print(tempcount);
            delay(200);
          }

          val1 = digitalRead(sw2);
          if (val1 == LOW)
          {
            delay(200);
            if (count == 1)
            {
              count1 = tempcount;
            }
            if (count == 2)
            {
              count2 = tempcount;
            }
            if (count == 3)
            {
              count3 = tempcount;
            }
            if (count == 4)
            {
              count4 = tempcount;
            }
            lcd.clear();
            delay(500);
            flag = 1;
            break;
          }
        }
        delay(200);
      }
    }
  }
}

void gsm(int countToZero)
{
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print(" SENDING    ");
  lcd.setCursor(0, 1);
  lcd.print("    IOT >>> ");
  Serial.print("*");
  Serial.print("Food Feeded");
  Serial.print("#");
  delay(500);

  // Zero out the specified count
  if (countToZero == 1) {
    count1 = 0;
  } else if (countToZero == 2) 
{
    count2 = 0;
  } else if (countToZero == 3)
 {
    count3 = 0;
  } else if (countToZero == 4) 
{
    count4 = 0;
  }

  lcd.clear();
}