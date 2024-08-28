# How-to-Use-External-Real-Time-Clock-RTC-with-Wemos-Esp8266

    // hd44780 library see https://github.com/duinoWitchery/hd44780
    // thehd44780 library is available through the IDE library manager
    #include <Wire.h>
    #include <hd44780.h>                       // main hd44780 header
    #include <hd44780ioClass/hd44780_I2Cexp.h> // i2c expander i/o class header
    #include <ThreeWire.h>  
    #include <RtcDS1302.h>

    ThreeWire myWire(D4,D5,D3); // IO, SCLK, CE
    RtcDS1302<ThreeWire> Rtc(myWire);
    hd44780_I2Cexp lcd; // declare lcd object: auto locate & auto config expander chip

    // LCD geometry
    const int LCD_COLS = 16;
    const int LCD_ROWS = 2;

    void setup()
    {
     Serial.begin(57600);

     lcd.begin(LCD_COLS, LCD_ROWS);
     lcd.clear();
     lcd.backlight();
     Serial.print("compiled: ");
     Serial.print(__DATE__);
     Serial.println(__TIME__);

    Rtc.Begin();

    RtcDateTime compiled = RtcDateTime(__DATE__, __TIME__);
    printDateTime(compiled);
    Serial.println();

    if (!Rtc.IsDateTimeValid()) 
    {
        // Common Causes:
        //    1) first time you ran and the device wasn't running yet
        //    2) the battery on the device is low or even missing

        Serial.println("RTC lost confidence in the DateTime!");
        Rtc.SetDateTime(compiled);
    }

    if (Rtc.GetIsWriteProtected())
    {
        Serial.println("RTC was write protected, enabling writing now");
        Rtc.SetIsWriteProtected(false);
    }

    if (!Rtc.GetIsRunning())
    {
        Serial.println("RTC was not actively running, starting now");
        Rtc.SetIsRunning(true);
    }

    RtcDateTime now = Rtc.GetDateTime();
    if (now < compiled) 
    {
        Serial.println("RTC is older than compile time!  (Updating DateTime)");
        Rtc.SetDateTime(compiled);
    }
    else if (now > compiled) 
    {
        Serial.println("RTC is newer than compile time. (this is expected)");
    }
    else if (now == compiled) 
    {
        Serial.println("RTC is the same as compile time! (not expected but all is fine)");
    }
    
     lcd.print("Hello World");
     lcd.setCursor(0, 1);
     lcd.print("Clock ");

   
    }

     void loop()
    {
    
    // updateLCD();

     RtcDateTime now = Rtc.GetDateTime();

    printDateTime(now);
    Serial.println();

    if (!now.IsValid())
    {
        // Common Causes:
        //    1) the battery on the device is low or even missing and the power line was disconnected
        Serial.println("RTC lost confidence in the DateTime!");
    }

      delay(10000); // ten seconds
    
     }

      #define countof(a) (sizeof(a) / sizeof(a[0]))

      void printDateTime(const RtcDateTime& dt)
       {
        lcd.clear();
            //24 Hours
            
            // char datestring[20];
            
            //   snprintf_P(datestring, 
            
            //  countof(datestring),
            
            //    PSTR("%02u/%02u/%04u %02u:%02u:%02u"),
            
           //    dt.Month(),
           
           //    dt.Day(),
           
           //   dt.Year(),
           
          //    dt.Hour();
          
          //   dt.Minute(),
          
          //   dt.Second() );
          
          // Serial.print(datestring);
          
          //  lcd.print(datestring);
     
           // 12 AM PM 
           const uint8_t h = dt.Hour();
           const uint8_t hr_12 = h%12;
           const uint8_t hrw=h%12;
           String AMPM = (h < 12) ? " AM" : " PM" ;
           if(hr_12 < 10){  
                  hrw == (h > 12) ? h - 12 : ((h == 0) ? 12 : h), DEC;
                         }
         else
                        {
                   hrw == ((h > 12) ? h - 12 : ((h == 0) ? 12 : h), DEC);
                         }
    
          char datestrings[10];
          snprintf_P(datestrings, 
          countof(datestrings),
          PSTR(" %02u:%02u:%02u  "),
            hrw ,
            dt.Minute(),
            dt.Second() );
            Serial.print(datestrings + AMPM );
           lcd.print(datestrings + AMPM);
       
       
           lcd.setCursor(1, 1);
           char datestring[11];
           snprintf_P(datestring, 
          countof(datestring),
          PSTR("%02u/%02u/%04u "),
            
            dt.Month(),
            dt.Day(),
            dt.Year()
           );
           Serial.print(datestring);
           lcd.print(datestring);
}
