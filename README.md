#include <Wire.h>
#include "RTClib.h"
RTC_DS1307 rtc;

#include <EEPROM.h>

#include <LiquidCrystal.h>
LiquidCrystal lcd(7, 6, 5, 4, 3, 2);

byte gama[8] = {
  B11111,
  B10000,
  B10000,
  B10000,
  B10000,
  B10000,
  B10000,
};

byte delta[8] = {
  B00100,
  B00000,
  B01010,
  B00000,
  B10001,
  B00000,
  B11111,
};

byte uhta[8] = {
  B00100,
  B10001,
  B10001,
  B10101,
  B10001,
  B10001,
  B00100,
};

byte lamda[8] = {
  B00100,
  B00000,
  B01010,
  B00000,
  B10001,
  B00000,
  B10001,
};

byte ji[8] = {
  B11111,
  B00000,
  B00000,
  B01110,
  B00000,
  B00000,
  B11111,  
};

byte pi[8] = {
  B11111,
  B10001,
  B10001,
  B10001,
  B10001,
  B10001,
  B10001,  
};

byte sigma[8] = {
  B11111,
  B10000,
  B01000,
  B00100,
  B01000,
  B10000,
  B11111,  
};

byte vmega[8] = {
  B00100,
  B10001,
  B10001,
  B10001,
  B00100,
  B00000,
  B11111, 
};

const int inputPin = A0;   

uint16_t inputValue = 0;   

#include <SoftwareSerial.h>
SoftwareSerial Thermal(12, 13);
int heatTime = 80;
int heatInterval = 255;
char printDensity = 15;
char printBreakTime = 15;
//char barCode[]={ '2','0','1','5','0','1','1','6','2','0','3','2'};
int zero=0;
int arapo=1000;
int ope=1;
float sum1=0;
float sum2=0;
float sum3=0;
float sum4=0;
float sum5=0;
float sum;
int counter1=1;
int counter2=1;
int counter3=1;
int counter4=1;
int counter5=1;

void setup() 
{
Serial.begin(57600);
#ifdef AVR
  Wire.begin();
#else
  Wire1.begin(); // Shield I2C pins connect to alt I2C bus on Arduino Due
#endif
  rtc.begin();

  if (! rtc.isrunning()) {
    Serial.println("RTC is NOT running!");
    // following line sets the RTC to the date & time this sketch was compiled
    rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
    // This line sets the RTC with an explicit date & time, for example to set
    // January 21, 2014 at 3am you would call:
    rtc.adjust(DateTime(2015, 1, 2, 21, 27, 40));
  } 
/*EEPROM_Write(&sum1, 1); // arxikopoihsh
EEPROM_Write(&sum2, 2); // arxikopoihsh 
EEPROM_Write(&sum3, 3); // arxikopoihsh 
EEPROM_Write(&sum4, 4); // arxikopoihsh 
EEPROM_Write(&sum5, 5); // arxikopoihsh 
*/
Thermal.begin(19200); // to write to our new printer
initPrinter();

  lcd.begin(16, 2);              
  lcd.createChar(1, delta);
  lcd.setCursor(0, 0); 
  lcd.write(byte(1));
  lcd.setCursor(1, 0);
  lcd.print("HMO");  
  lcd.createChar(6, sigma);
  lcd.setCursor(4, 0);           
  lcd.write(byte(6));
  lcd.createChar(3, lamda);
  lcd.setCursor(6, 0);           
  lcd.write(byte(3));
  lcd.setCursor(7, 0);           
  lcd.print("API"); 
  lcd.createChar(6, sigma);
  lcd.setCursor(10, 0);           
  lcd.write(byte(6));
  lcd.setCursor(11, 0);           
  lcd.print("AI");
  lcd.createChar(7, vmega);
  lcd.setCursor(13, 0);  
  lcd.write(byte(7));
  lcd.setCursor(14, 0);  
  lcd.print("N"); 
  lcd.createChar(5, pi);
  lcd.setCursor(0, 1); 
  lcd.write(byte(5));
  lcd.setCursor(1, 1); 
  lcd.print("APKOMETPO");
}

void initPrinter()
{
//Modify the print speed and heat
Thermal.write(27);
Thermal.write(55);
Thermal.write(7); //Default 64 dots = 8*('7'+1)
//Thermal.write(heatTime); //Default 80 or 800us
Thermal.write(30); //Default 80 or 800us
Thermal.write(heatInterval); //Default 2 or 20us
//Modify the print density and timeout
Thermal.write(18);
Thermal.write(35);
int printSetting = (printDensity<<4) | printBreakTime;
Thermal.write(printSetting); //Combination of printDensity and printBreakTime
//Serial.println();
//Serial.println("Printer ready");
}

void printBarcodeThick(char zz[])
{
 Thermal.write(29); 
 Thermal.write(72); 
 Thermal.write(2); 
 Thermal.write(29); 
 Thermal.write(119); 
 Thermal.write(3); 
 Thermal.write(29); 
 Thermal.write(104); 
 Thermal.write(100); 
 Thermal.write(29); 
 Thermal.write(107);
 Thermal.write(zero); 
 for (int z=0; z<12; z++)
 {
 Thermal.write(zz[z]);
 }
 Thermal.write(zero); 
 delay(0000); 
}

void EEPROM_Write(float *sum, int MemPos)
{
  byte ByteArray[4];
  memcpy(ByteArray, sum, 4);
  for(int x = 0; x < 4; x++)
  {
    EEPROM.write((MemPos * 4) + x, ByteArray[x]);
  }  
}

void EEPROM_Read(float *sum, int MemPos)
{
  byte ByteArray[4];
  for(int x = 0; x < 4; x++)
  {
    ByteArray[x] = EEPROM.read((MemPos * 4) + x);    
  }
  memcpy(sum, ByteArray, 4);
}

void loop() 
{
  inputValue = analogRead(inputPin);
  if(inputValue < 100 && inputValue >= 0) inputValue = 1;
  else if(inputValue < 250 && inputValue > 150) inputValue = 2;
  else if(inputValue < 470 && inputValue > 370) inputValue = 3;
  else if(inputValue < 670 && inputValue > 570) inputValue = 4;
  else if(inputValue < 870 && inputValue > 770) inputValue = 5;
  else if(inputValue <= 1023 && inputValue > 950) inputValue = 0;
  
  if(inputValue == 1)
  {
    DateTime now = rtc.now();
    Thermal.print("A");
    Thermal.write(0xD0);
    Thermal.print("O");
    Thermal.write(0xC4);
    Thermal.print("EI");
    Thermal.write(0xCE);
    Thermal.print("H  ");
    Thermal.write(0xD0);
    Thermal.print("APOXH");
    Thermal.write(0xD3);
    Thermal.print(" Y");
    Thermal.write(0xD0);
    Thermal.print("HPE");
    Thermal.write(0xD3);
    Thermal.print("I");
    Thermal.write(0xD9);
    Thermal.println("N");
    Thermal.write(0xC4);
    Thermal.print("HMO");
    Thermal.write(0xD3);
    Thermal.print(" ");
    Thermal.write(0xCB);
    Thermal.print("API");
    Thermal.write(0xD3);
    Thermal.print("AI");
    Thermal.write(0xD9);
    Thermal.println("N");
    Thermal.print("<");
    Thermal.write(0xD0);
    Thermal.println("APKOMETPO>");
    Thermal.print("TH");
    Thermal.write(0xCB);
    Thermal.println(":2410000000");
    Thermal.print("A");
    Thermal.write(0xD6);
    Thermal.println("M 000000000");
    Thermal.write(0xC4);
    Thermal.print("OY ");
    Thermal.write(0xCB);
    Thermal.print("API");
    Thermal.write(0xD3);
    Thermal.print("A");
    Thermal.write(0xD3);
    Thermal.println();
    Thermal.print("A/A.A");
    Thermal.write(0xD0);
    Thermal.print("O");
    Thermal.write(0xC4);      
    Thermal.print(".:");
    arapo ++;
    Thermal.println(arapo);      
    Thermal.print("HM/NIA:");      
    Thermal.print(now.year(), DEC);
    Thermal.print('/');
    Thermal.print(now.month(), DEC);
    Thermal.print('/');
    Thermal.print(now.day(), DEC);
    Thermal.print(' ');
    Thermal.print(now.hour(), DEC);
    Thermal.print(':');
    Thermal.print(now.minute(), DEC);
    Thermal.print(':');
    Thermal.print(now.second(), DEC);
    Thermal.println();
    Thermal.write(0xC4);      
    Thermal.print("IAPKEIA:O.5");
    Thermal.write(0xD9);
    Thermal.println("PA");
    Thermal.print("KO");
    Thermal.write(0xD3);
    Thermal.print("TO");
    Thermal.write(0xD3);
    Thermal.print(":O.5EYP");
    Thermal.write(0xD9);
    Thermal.println();
    Thermal.write(29);
    Thermal.write(33);
    Thermal.write(300);
    DateTime future (now.unixtime() + 30*60);
    Thermal.write(0xCB);
    Thermal.print("H");
    Thermal.write(0xCE);   
    Thermal.print("H:");
    Thermal.print(future.hour(), DEC);
    Thermal.print(':');
    Thermal.print(future.minute(), DEC);
    Thermal.println();
    Thermal.write(29);
    Thermal.write(33);
    Thermal.write(zero);
    Thermal.println();
    Thermal.write(10);
    Thermal.print("A");
    Thermal.write(0xE8);
    Thermal.print("/TA BA");
    Thermal.write(0xD3);
    Thermal.print("EI AYO ");
    Thermal.write(0xD0);
    Thermal.print("O");
    Thermal.write(0xCB);
    Thermal.println(".1083/03");
    Thermal.write(10);
    long aa = now.year() + now.month() + now.day();
    long bb = aa * aa + (now.day() * 1000000 + now.month() * 100000);
    int a0 = bb/10000000; 
    char b0=char(a0+48);  
    int a1 = (bb-a0*10000000)/1000000; 
    char b1=char(a1+48); 
    int a2 = (bb-a0*10000000-a1*1000000)/100000; 
    char b2=char(a2+48); 
    int a3 = (bb-a0*10000000-a1*1000000-a2*100000)/10000; 
    char b3=char(a3+48); 
    int a4 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L)/1000L; 
    char b4=char(a4+48); 
    int a5 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L-a4*1000L)/100L; 
    char b5=char(a5+48); 
    int a6 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L-a4*1000L-a5*100L)/10L; 
    char b6=char(a6+48); 
    int a7 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L-a4*1000L-a5*100L-a6*10L)/1L; 
    char b7=char(a7+48); 
    int a8 = future.hour()/10; 
    char b8=char(a8+48); 
    int a9 = (future.hour()-a8*10)/1; 
    char b9=char(a9+48); 
    int a10 = future.minute()/10; 
    char b10=char(a10+48); 
    int a11 = (future.minute()-a10*10)/1; 
    char b11=char(a11+48); 
    char barCode[12]={ b0,b1,b2,b3,b4,b5,b6,b7,b8,b9,b10,b11};
    printBarcodeThick(barCode);
    Thermal.print("Operator:");
    Thermal.println(ope);          
    if (counter1=1)
    {EEPROM_Read(&sum1, 1); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum2, 2); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum3, 3); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum4, 4); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum5, 5); // starts at 0 and goes up to max depending on processor.
    }
    counter1++;
    sum1 = sum1 + 0.5;
    EEPROM_Write(&sum1, 1); // memory position tested as '1'. Memory position
    sum=sum1+sum2+sum3+sum4+sum5;
    Thermal.print("Sum:");
    Thermal.print(sum);             
    Thermal.write(0x80);   
    Thermal.println();              
    Thermal.write(10);
    Thermal.write(10);
    Thermal.write(10);
    Thermal.write(10);    
    lcd.setCursor(0, 0);
    lcd.print(now.year(), DEC);
    lcd.print('/');
    lcd.print(now.month(), DEC);
    lcd.print('/');
    lcd.print(now.day(), DEC);
    lcd.print(' ');
    lcd.print(now.hour(), DEC);
    lcd.print(':');
    lcd.print(now.minute(), DEC);
    lcd.print(':');
    lcd.print(now.second(), DEC);
    lcd.println();
    delay(000);
    lcd.setCursor(0, 1);
    lcd.print("0.5");
    lcd.createChar(7, vmega);
    lcd.setCursor(3, 1);  
    lcd.write(byte(7));
    lcd.setCursor(4, 1);      
    lcd.print("PA=0.5EYP");
    lcd.createChar(7, vmega);
    lcd.setCursor(13, 1);  
    lcd.write(byte(7));
    delay(2000);
    lcd.clear();
  }
  else if(inputValue == 2)
  {
    lcd.setCursor(0, 0);
    DateTime now = rtc.now();
    Thermal.print("A");
    Thermal.write(0xD0);
    Thermal.print("O");
    Thermal.write(0xC4);
    Thermal.print("EI");
    Thermal.write(0xCE);
    Thermal.print("H  ");
    Thermal.write(0xD0);
    Thermal.print("APOXH");
    Thermal.write(0xD3);
    Thermal.print(" Y");
    Thermal.write(0xD0);
    Thermal.print("HPE");
    Thermal.write(0xD3);
    Thermal.print("I");
    Thermal.write(0xD9);
    Thermal.println("N");
    Thermal.write(0xC4);
    Thermal.print("HMO");
    Thermal.write(0xD3);
    Thermal.print(" ");
    Thermal.write(0xCB);
    Thermal.print("API");
    Thermal.write(0xD3);
    Thermal.print("AI");
    Thermal.write(0xD9);
    Thermal.println("N");
    Thermal.print("<");
    Thermal.write(0xD0);
    Thermal.println("APKOMETPO>");
    Thermal.print("TH");
    Thermal.write(0xCB);
    Thermal.println(":2410000000");
    Thermal.print("A");
    Thermal.write(0xD6);
    Thermal.println("M 000000000");
    Thermal.write(0xC4);
    Thermal.print("OY ");
    Thermal.write(0xCB);
    Thermal.print("API");
    Thermal.write(0xD3);
    Thermal.print("A");
    Thermal.write(0xD3);
    Thermal.println();
    Thermal.print("A/A.A");
    Thermal.write(0xD0);
    Thermal.print("O");
    Thermal.write(0xC4);      
    Thermal.print(".:");
    arapo ++;
    Thermal.println(arapo);      
    Thermal.print("HM/NIA:");      
    Thermal.print(now.year(), DEC);
    Thermal.print('/');
    Thermal.print(now.month(), DEC);
    Thermal.print('/');
    Thermal.print(now.day(), DEC);
    Thermal.print(' ');
    Thermal.print(now.hour(), DEC);
    Thermal.print(':');
    Thermal.print(now.minute(), DEC);
    Thermal.print(':');
    Thermal.print(now.second(), DEC);
    Thermal.println();
    Thermal.write(0xC4);      
    Thermal.print("IAPKEIA:1");
    Thermal.write(0xD9);
    Thermal.println("PA");
    Thermal.print("KO");
    Thermal.write(0xD3);
    Thermal.print("TO");
    Thermal.write(0xD3);
    Thermal.print(":1EYP");
    Thermal.write(0xD9);
    Thermal.println();
    Thermal.write(29);
    Thermal.write(33);
    Thermal.write(300);
    DateTime future (now.unixtime() + 60*60);
    Thermal.write(0xCB);
    Thermal.print("H");
    Thermal.write(0xCE);   
    Thermal.print("H:");
    Thermal.print(future.hour(), DEC);
    Thermal.print(':');
    Thermal.print(future.minute(), DEC);
    Thermal.println();
    Thermal.write(29);
    Thermal.write(33);
    Thermal.write(zero);
    Thermal.println();
    Thermal.write(10);
    Thermal.print("A");
    Thermal.write(0xE8);
    Thermal.print("/TA BA");
    Thermal.write(0xD3);
    Thermal.print("EI AYO ");
    Thermal.write(0xD0);
    Thermal.print("O");
    Thermal.write(0xCB);
    Thermal.println(".1083/03");
    Thermal.write(10);
    long aa = now.year() + now.month() + now.day();
    long bb = aa * aa + (now.day() * 1000000 + now.month() * 100000);
    int a0 = bb/10000000; 
    char b0=char(a0+48);  
    int a1 = (bb-a0*10000000)/1000000; 
    char b1=char(a1+48); 
    int a2 = (bb-a0*10000000-a1*1000000)/100000; 
    char b2=char(a2+48); 
    int a3 = (bb-a0*10000000-a1*1000000-a2*100000)/10000; 
    char b3=char(a3+48); 
    int a4 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L)/1000L; 
    char b4=char(a4+48); 
    int a5 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L-a4*1000L)/100L; 
    char b5=char(a5+48); 
    int a6 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L-a4*1000L-a5*100L)/10L; 
    char b6=char(a6+48); 
    int a7 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L-a4*1000L-a5*100L-a6*10L)/1L; 
    char b7=char(a7+48); 
    int a8 = future.hour()/10; 
    char b8=char(a8+48); 
    int a9 = (future.hour()-a8*10)/1; 
    char b9=char(a9+48); 
    int a10 = future.minute()/10; 
    char b10=char(a10+48); 
    int a11 = (future.minute()-a10*10)/1; 
    char b11=char(a11+48); 
    char barCode[12]={ b0,b1,b2,b3,b4,b5,b6,b7,b8,b9,b10,b11};
    printBarcodeThick(barCode);
    Thermal.print("Operator:");
    Thermal.println(ope);          
    if (counter2=1)
    {EEPROM_Read(&sum1, 1); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum2, 2); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum3, 3); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum4, 4); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum5, 5); // starts at 0 and goes up to max depending on processor.
    }
    counter2++;
    sum2 = sum2 + 1;
    EEPROM_Write(&sum2, 2); // memory position tested as '1'. Memory position
    sum=sum1+sum2+sum3+sum4+sum5;
    Thermal.print("Sum:");
    Thermal.print(sum);             
    Thermal.write(0x80);   
    Thermal.println();              
    Thermal.write(10);
    Thermal.write(10);
    Thermal.write(10);
    Thermal.write(10);    
    lcd.print(now.year(), DEC);
    lcd.print('/');
    lcd.print(now.month(), DEC);
    lcd.print('/');
    lcd.print(now.day(), DEC);
    lcd.print(' ');
    lcd.print(now.hour(), DEC);
    lcd.print(':');
    lcd.print(now.minute(), DEC);
    lcd.print(':');
    lcd.print(now.second(), DEC);
    lcd.println();
    delay(000);
    lcd.setCursor(0, 1);
    lcd.print("1");
    lcd.createChar(7, vmega);
    lcd.setCursor(1, 1);  
    lcd.write(byte(7));
    lcd.setCursor(2, 1);      
    lcd.print("PA=1EYP");
    lcd.createChar(7, vmega);
    lcd.setCursor(9, 1);  
    lcd.write(byte(7));
    delay(2000);
    lcd.clear();
  }
  else if(inputValue == 3)
  {
    lcd.setCursor(0, 0);
    DateTime now = rtc.now();
    Thermal.print("A");
    Thermal.write(0xD0);
    Thermal.print("O");
    Thermal.write(0xC4);
    Thermal.print("EI");
    Thermal.write(0xCE);
    Thermal.print("H  ");
    Thermal.write(0xD0);
    Thermal.print("APOXH");
    Thermal.write(0xD3);
    Thermal.print(" Y");
    Thermal.write(0xD0);
    Thermal.print("HPE");
    Thermal.write(0xD3);
    Thermal.print("I");
    Thermal.write(0xD9);
    Thermal.println("N");
    Thermal.write(0xC4);
    Thermal.print("HMO");
    Thermal.write(0xD3);
    Thermal.print(" ");
    Thermal.write(0xCB);
    Thermal.print("API");
    Thermal.write(0xD3);
    Thermal.print("AI");
    Thermal.write(0xD9);
    Thermal.println("N");
    Thermal.print("<");
    Thermal.write(0xD0);
    Thermal.println("APKOMETPO>");
    Thermal.print("TH");
    Thermal.write(0xCB);
    Thermal.println(":2410000000");
    Thermal.print("A");
    Thermal.write(0xD6);
    Thermal.println("M 000000000");
    Thermal.write(0xC4);
    Thermal.print("OY ");
    Thermal.write(0xCB);
    Thermal.print("API");
    Thermal.write(0xD3);
    Thermal.print("A");
    Thermal.write(0xD3);
    Thermal.println();
    Thermal.print("A/A.A");
    Thermal.write(0xD0);
    Thermal.print("O");
    Thermal.write(0xC4);      
    Thermal.print(".:");
    arapo ++;
    Thermal.println(arapo);      
    Thermal.print("HM/NIA:");      
    Thermal.print(now.year(), DEC);
    Thermal.print('/');
    Thermal.print(now.month(), DEC);
    Thermal.print('/');
    Thermal.print(now.day(), DEC);
    Thermal.print(' ');
    Thermal.print(now.hour(), DEC);
    Thermal.print(':');
    Thermal.print(now.minute(), DEC);
    Thermal.print(':');
    Thermal.print(now.second(), DEC);
    Thermal.println();
    Thermal.write(0xC4);      
    Thermal.print("IAPKEIA:1.5");
    Thermal.write(0xD9);
    Thermal.println("PA");
    Thermal.print("KO");
    Thermal.write(0xD3);
    Thermal.print("TO");
    Thermal.write(0xD3);
    Thermal.print(":1.5EYP");
    Thermal.write(0xD9);
    Thermal.println();
    Thermal.write(29);
    Thermal.write(33);
    Thermal.write(300);
    DateTime future (now.unixtime() + 90*60);
    Thermal.write(0xCB);
    Thermal.print("H");
    Thermal.write(0xCE);   
    Thermal.print("H:");
    Thermal.print(future.hour(), DEC);
    Thermal.print(':');
    Thermal.print(future.minute(), DEC);
    Thermal.println();
    Thermal.write(29);
    Thermal.write(33);
    Thermal.write(zero);
    Thermal.println();
    Thermal.write(10);
    Thermal.print("A");
    Thermal.write(0xE8);
    Thermal.print("/TA BA");
    Thermal.write(0xD3);
    Thermal.print("EI AYO ");
    Thermal.write(0xD0);
    Thermal.print("O");
    Thermal.write(0xCB);
    Thermal.println(".1083/03");
    Thermal.write(10);
    long aa = now.year() + now.month() + now.day();
    long bb = aa * aa + (now.day() * 1000000 + now.month() * 100000);
    int a0 = bb/10000000; 
    char b0=char(a0+48);  
    int a1 = (bb-a0*10000000)/1000000; 
    char b1=char(a1+48); 
    int a2 = (bb-a0*10000000-a1*1000000)/100000; 
    char b2=char(a2+48); 
    int a3 = (bb-a0*10000000-a1*1000000-a2*100000)/10000; 
    char b3=char(a3+48); 
    int a4 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L)/1000L; 
    char b4=char(a4+48); 
    int a5 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L-a4*1000L)/100L; 
    char b5=char(a5+48); 
    int a6 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L-a4*1000L-a5*100L)/10L; 
    char b6=char(a6+48); 
    int a7 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L-a4*1000L-a5*100L-a6*10L)/1L; 
    char b7=char(a7+48); 
    int a8 = future.hour()/10; 
    char b8=char(a8+48); 
    int a9 = (future.hour()-a8*10)/1; 
    char b9=char(a9+48); 
    int a10 = future.minute()/10; 
    char b10=char(a10+48); 
    int a11 = (future.minute()-a10*10)/1; 
    char b11=char(a11+48); 
    char barCode[12]={ b0,b1,b2,b3,b4,b5,b6,b7,b8,b9,b10,b11};
    printBarcodeThick(barCode);
    Thermal.print("Operator:");
    Thermal.println(ope);          
    if (counter3=1)
    {EEPROM_Read(&sum1, 1); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum2, 2); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum3, 3); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum4, 4); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum5, 5); // starts at 0 and goes up to max depending on processor.
    }
    counter3++;
    sum3 = sum3 + 1.5;
    EEPROM_Write(&sum3, 3); // memory position tested as '1'. Memory position
    sum=sum1+sum2+sum3+sum4+sum5;
    Thermal.print("Sum:");
    Thermal.print(sum);             
    Thermal.write(0x80);   
    Thermal.println();              
    Thermal.write(10);
    Thermal.write(10);
    Thermal.write(10);
    Thermal.write(10);    
    lcd.print(now.year(), DEC);
    lcd.print('/');
    lcd.print(now.month(), DEC);
    lcd.print('/');
    lcd.print(now.day(), DEC);
    lcd.print(' ');
    lcd.print(now.hour(), DEC);
    lcd.print(':');
    lcd.print(now.minute(), DEC);
    lcd.print(':');
    lcd.print(now.second(), DEC);
    lcd.println();
    delay(000);
    lcd.setCursor(0, 1);
    lcd.print("1.5");
    lcd.createChar(7, vmega);
    lcd.setCursor(3, 1);  
    lcd.write(byte(7));
    lcd.setCursor(4, 1);      
    lcd.print("PA=1.5EYP");
    lcd.createChar(7, vmega);
    lcd.setCursor(13, 1);  
    lcd.write(byte(7));
    delay(2000);
    lcd.clear();
  }
  else if(inputValue == 4)
  {
    lcd.setCursor(0, 0);
    DateTime now = rtc.now();
    Thermal.print("A");
    Thermal.write(0xD0);
    Thermal.print("O");
    Thermal.write(0xC4);
    Thermal.print("EI");
    Thermal.write(0xCE);
    Thermal.print("H  ");
    Thermal.write(0xD0);
    Thermal.print("APOXH");
    Thermal.write(0xD3);
    Thermal.print(" Y");
    Thermal.write(0xD0);
    Thermal.print("HPE");
    Thermal.write(0xD3);
    Thermal.print("I");
    Thermal.write(0xD9);
    Thermal.println("N");
    Thermal.write(0xC4);
    Thermal.print("HMO");
    Thermal.write(0xD3);
    Thermal.print(" ");
    Thermal.write(0xCB);
    Thermal.print("API");
    Thermal.write(0xD3);
    Thermal.print("AI");
    Thermal.write(0xD9);
    Thermal.println("N");
    Thermal.print("<");
    Thermal.write(0xD0);
    Thermal.println("APKOMETPO>");
    Thermal.print("TH");
    Thermal.write(0xCB);
    Thermal.println(":2410000000");
    Thermal.print("A");
    Thermal.write(0xD6);
    Thermal.println("M 000000000");
    Thermal.write(0xC4);
    Thermal.print("OY ");
    Thermal.write(0xCB);
    Thermal.print("API");
    Thermal.write(0xD3);
    Thermal.print("A");
    Thermal.write(0xD3);
    Thermal.println();
    Thermal.print("A/A.A");
    Thermal.write(0xD0);
    Thermal.print("O");
    Thermal.write(0xC4);      
    Thermal.print(".:");
    arapo ++;
    Thermal.println(arapo);      
    Thermal.print("HM/NIA:");      
    Thermal.print(now.year(), DEC);
    Thermal.print('/');
    Thermal.print(now.month(), DEC);
    Thermal.print('/');
    Thermal.print(now.day(), DEC);
    Thermal.print(' ');
    Thermal.print(now.hour(), DEC);
    Thermal.print(':');
    Thermal.print(now.minute(), DEC);
    Thermal.print(':');
    Thermal.print(now.second(), DEC);
    Thermal.println();
    Thermal.write(0xC4);      
    Thermal.print("IAPKEIA:2");
    Thermal.write(0xD9);
    Thermal.print("PE");
    Thermal.write(0xD3);
    Thermal.println();
    Thermal.print("KO");
    Thermal.write(0xD3);
    Thermal.print("TO");
    Thermal.write(0xD3);
    Thermal.print(":2EYP");
    Thermal.write(0xD9);
    Thermal.println();
    Thermal.write(29);
    Thermal.write(33);
    Thermal.write(300);
    DateTime future (now.unixtime() + 120*60);
    Thermal.write(0xCB);
    Thermal.print("H");
    Thermal.write(0xCE);   
    Thermal.print("H:");
    Thermal.print(future.hour(), DEC);
    Thermal.print(':');
    Thermal.print(future.minute(), DEC);
    Thermal.println();
    Thermal.write(29);
    Thermal.write(33);
    Thermal.write(zero);
    Thermal.println();
    Thermal.write(10);
    Thermal.print("A");
    Thermal.write(0xE8);
    Thermal.print("/TA BA");
    Thermal.write(0xD3);
    Thermal.print("EI AYO ");
    Thermal.write(0xD0);
    Thermal.print("O");
    Thermal.write(0xCB);
    Thermal.println(".1083/03");
    Thermal.write(10);
    long aa = now.year() + now.month() + now.day();
    long bb = aa * aa + (now.day() * 1000000 + now.month() * 100000);
    int a0 = bb/10000000; 
    char b0=char(a0+48);  
    int a1 = (bb-a0*10000000)/1000000; 
    char b1=char(a1+48); 
    int a2 = (bb-a0*10000000-a1*1000000)/100000; 
    char b2=char(a2+48); 
    int a3 = (bb-a0*10000000-a1*1000000-a2*100000)/10000; 
    char b3=char(a3+48); 
    int a4 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L)/1000L; 
    char b4=char(a4+48); 
    int a5 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L-a4*1000L)/100L; 
    char b5=char(a5+48); 
    int a6 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L-a4*1000L-a5*100L)/10L; 
    char b6=char(a6+48); 
    int a7 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L-a4*1000L-a5*100L-a6*10L)/1L; 
    char b7=char(a7+48); 
    int a8 = future.hour()/10; 
    char b8=char(a8+48); 
    int a9 = (future.hour()-a8*10)/1; 
    char b9=char(a9+48); 
    int a10 = future.minute()/10; 
    char b10=char(a10+48); 
    int a11 = (future.minute()-a10*10)/1; 
    char b11=char(a11+48); 
    char barCode[12]={ b0,b1,b2,b3,b4,b5,b6,b7,b8,b9,b10,b11};
    printBarcodeThick(barCode);
    Thermal.print("Operator:");
    Thermal.println(ope);          
    if (counter4=1)
    {EEPROM_Read(&sum1, 1); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum2, 2); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum3, 3); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum4, 4); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum5, 5); // starts at 0 and goes up to max depending on processor.
    }
    counter4++;
    sum4 = sum4 + 2;
    EEPROM_Write(&sum4, 4); // memory position tested as '1'. Memory position
    sum=sum1+sum2+sum3+sum4+sum5;
    Thermal.print("Sum:");
    Thermal.print(sum);             
    Thermal.write(0x80);   
    Thermal.println();              
    Thermal.write(10);
    Thermal.write(10);
    Thermal.write(10);
    Thermal.write(10);    
    lcd.print(now.year(), DEC);
    lcd.print('/');
    lcd.print(now.month(), DEC);
    lcd.print('/');
    lcd.print(now.day(), DEC);
    lcd.print(' ');
    lcd.print(now.hour(), DEC);
    lcd.print(':');
    lcd.print(now.minute(), DEC);
    lcd.print(':');
    lcd.print(now.second(), DEC);
    lcd.println();
    delay(000);
    lcd.setCursor(0, 1);
    lcd.print("2");
    lcd.createChar(7, vmega);
    lcd.setCursor(1, 1);  
    lcd.write(byte(7));
    lcd.setCursor(2, 1);      
    lcd.print("PE");
    lcd.createChar(6, sigma);
    lcd.setCursor(4, 1);           
    lcd.write(byte(6));    
    lcd.setCursor(5, 1);  
    lcd.print("=2EYP");
    lcd.createChar(7, vmega);
    lcd.setCursor(10, 1);  
    lcd.write(byte(7));
    delay(2000);
    lcd.clear();
  }
  else if(inputValue == 5)
  {
    lcd.setCursor(0, 0);
    DateTime now = rtc.now();
    Thermal.print("A");
    Thermal.write(0xD0);
    Thermal.print("O");
    Thermal.write(0xC4);
    Thermal.print("EI");
    Thermal.write(0xCE);
    Thermal.print("H  ");
    Thermal.write(0xD0);
    Thermal.print("APOXH");
    Thermal.write(0xD3);
    Thermal.print(" Y");
    Thermal.write(0xD0);
    Thermal.print("HPE");
    Thermal.write(0xD3);
    Thermal.print("I");
    Thermal.write(0xD9);
    Thermal.println("N");
    Thermal.write(0xC4);
    Thermal.print("HMO");
    Thermal.write(0xD3);
    Thermal.print(" ");
    Thermal.write(0xCB);
    Thermal.print("API");
    Thermal.write(0xD3);
    Thermal.print("AI");
    Thermal.write(0xD9);
    Thermal.println("N");
    Thermal.print("<");
    Thermal.write(0xD0);
    Thermal.println("APKOMETPO>");
    Thermal.print("TH");
    Thermal.write(0xCB);
    Thermal.println(":2410000000");
    Thermal.print("A");
    Thermal.write(0xD6);
    Thermal.println("M 000000000");
    Thermal.write(0xC4);
    Thermal.print("OY ");
    Thermal.write(0xCB);
    Thermal.print("API");
    Thermal.write(0xD3);
    Thermal.print("A");
    Thermal.write(0xD3);
    Thermal.println();
    Thermal.print("A/A.A");
    Thermal.write(0xD0);
    Thermal.print("O");
    Thermal.write(0xC4);      
    Thermal.print(".:");
    arapo ++;
    Thermal.println(arapo);      
    Thermal.print("HM/NIA:");      
    Thermal.print(now.year(), DEC);
    Thermal.print('/');
    Thermal.print(now.month(), DEC);
    Thermal.print('/');
    Thermal.print(now.day(), DEC);
    Thermal.print(' ');
    Thermal.print(now.hour(), DEC);
    Thermal.print(':');
    Thermal.print(now.minute(), DEC);
    Thermal.print(':');
    Thermal.print(now.second(), DEC);
    Thermal.println();
    Thermal.write(0xC4);      
    Thermal.print("IAPKEIA:2.5");
    Thermal.write(0xD9);
    Thermal.print("PE");
    Thermal.write(0xD3);
    Thermal.println();
    Thermal.print("KO");
    Thermal.write(0xD3);
    Thermal.print("TO");
    Thermal.write(0xD3);
    Thermal.print(":2.5EYP");
    Thermal.write(0xD9);
    Thermal.println();
    Thermal.write(29);
    Thermal.write(33);
    Thermal.write(300);
    DateTime future (now.unixtime() + 150*60);
    Thermal.write(0xCB);
    Thermal.print("H");
    Thermal.write(0xCE);   
    Thermal.print("H:");
    Thermal.print(future.hour(), DEC);
    Thermal.print(':');
    Thermal.print(future.minute(), DEC);
    Thermal.println();
    Thermal.write(29);
    Thermal.write(33);
    Thermal.write(zero);
    Thermal.println();
    Thermal.write(10);
    Thermal.print("A");
    Thermal.write(0xE8);
    Thermal.print("/TA BA");
    Thermal.write(0xD3);
    Thermal.print("EI AYO ");
    Thermal.write(0xD0);
    Thermal.print("O");
    Thermal.write(0xCB);
    Thermal.println(".1083/03");
    Thermal.write(10);
    long aa = now.year() + now.month() + now.day();
    long bb = aa * aa + (now.day() * 1000000 + now.month() * 100000);
    int a0 = bb/10000000; 
    char b0=char(a0+48);  
    int a1 = (bb-a0*10000000)/1000000; 
    char b1=char(a1+48); 
    int a2 = (bb-a0*10000000-a1*1000000)/100000; 
    char b2=char(a2+48); 
    int a3 = (bb-a0*10000000-a1*1000000-a2*100000)/10000; 
    char b3=char(a3+48); 
    int a4 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L)/1000L; 
    char b4=char(a4+48); 
    int a5 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L-a4*1000L)/100L; 
    char b5=char(a5+48); 
    int a6 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L-a4*1000L-a5*100L)/10L; 
    char b6=char(a6+48); 
    int a7 = (bb-a0*10000000L-a1*1000000L-a2*100000L-a3*10000L-a4*1000L-a5*100L-a6*10L)/1L; 
    char b7=char(a7+48); 
    int a8 = future.hour()/10; 
    char b8=char(a8+48); 
    int a9 = (future.hour()-a8*10)/1; 
    char b9=char(a9+48); 
    int a10 = future.minute()/10; 
    char b10=char(a10+48); 
    int a11 = (future.minute()-a10*10)/1; 
    char b11=char(a11+48); 
    char barCode[12]={ b0,b1,b2,b3,b4,b5,b6,b7,b8,b9,b10,b11};
    printBarcodeThick(barCode);
    Thermal.print("Operator:");
    Thermal.println(ope);          
    if (counter5=1)
    {EEPROM_Read(&sum1, 1); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum2, 2); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum3, 3); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum4, 4); // starts at 0 and goes up to max depending on processor.
    EEPROM_Read(&sum5, 5); // starts at 0 and goes up to max depending on processor.
    }
    counter5++;
    sum5 = sum5 + 2.5;
    EEPROM_Write(&sum5, 5); // memory position tested as '1'. Memory position
    sum=sum1+sum2+sum3+sum4+sum5;
    Thermal.print("Sum:");
    Thermal.print(sum);             
    Thermal.write(0x80);   
    Thermal.println();              
    Thermal.write(10);
    Thermal.write(10);
    Thermal.write(10);
    Thermal.write(10);    
    lcd.print(now.year(), DEC);
    lcd.print('/');
    lcd.print(now.month(), DEC);
    lcd.print('/');
    lcd.print(now.day(), DEC);
    lcd.print(' ');
    lcd.print(now.hour(), DEC);
    lcd.print(':');
    lcd.print(now.minute(), DEC);
    lcd.print(':');
    lcd.print(now.second(), DEC);
    lcd.println();
    delay(000);
    lcd.setCursor(0, 1);
    lcd.print("2.5");
    lcd.createChar(7, vmega);
    lcd.setCursor(3, 1);  
    lcd.write(byte(7));
    lcd.setCursor(4, 1);      
    lcd.print("PE");
    lcd.createChar(6, sigma);
    lcd.setCursor(6, 1);           
    lcd.write(byte(6));    
    lcd.setCursor(7, 1);  
    lcd.print("=2.5EYP");
    lcd.createChar(7, vmega);
    lcd.setCursor(14, 1);  
    lcd.write(byte(7));
    delay(2000);
    lcd.clear();
  }
  else
  {
    lcd.setCursor(0, 0);
    lcd.createChar(1, delta);
    lcd.setCursor(0, 0); 
    lcd.write(byte(1));
    lcd.setCursor(1, 0);
    lcd.print("HMO");  
    lcd.createChar(6, sigma);
    lcd.setCursor(4, 0);           
    lcd.write(byte(6));
    lcd.createChar(3, lamda);
    lcd.setCursor(6, 0);           
    lcd.write(byte(3));
    lcd.setCursor(7, 0);           
    lcd.print("API"); 
    lcd.createChar(6, sigma);
    lcd.setCursor(10, 0);           
    lcd.write(byte(6));
    lcd.setCursor(11, 0);           
    lcd.print("AI");
    lcd.createChar(7, vmega);
    lcd.setCursor(13, 0);  
    lcd.write(byte(7));
    lcd.setCursor(14, 0);  
    lcd.print("N"); 
    lcd.createChar(5, pi);
    lcd.setCursor(0, 1); 
    lcd.write(byte(5));
    lcd.setCursor(1, 1); 
    lcd.print("APKOMETPO");   
    }
