## Overview ##
This is updated version of original version of [Extended Database Library Project](http://www.arduino.cc/playground/Code/ExtendedDatabaseLibrary) which has several improvements:

 * This version creates a byte flag at the beginning of database header. This allows to check if Database is already created and readable.
 * Added *EDB_ERROR* status to **EDB_Status**
 * Now *open()* function returns *EDB_OK* if database is already created and readable, and returns *EDB_ERROR* if database is not created or readable
 * Added new example with External SD-card usage and AT24C512 I2C EEPROM with two Databases
 * Some common bugfixes

[Extended Database Library Project Home at the Arduino Playground](http://www.arduino.cc/playground/Code/ExtendedDatabaseLibrary)

Examples included:

  * [Arduino EEPROM Library](http://www.arduino.cc/en/Reference/EEPROM) providing 512-4096 bytes of storage
  * [AT24C512 I2C EEPROM Library](https://github.com/husio-org/AT24C512C)
  * [AT24C1024 I2C EEPROM Library](http://arduino.cc/playground/Code/I2CEEPROM24C1024) providing 128-512 kilobytes of storage

## Installing ##

Unzip the download into your Arduino-00xx/hardware/libraries directory. If the Arduino IDE is already running then exit and restart the Arduino IDE.

## Quickstart ##

How to use in a nutshell:

  * include EDB.h
  * define a structure for your records
  * include an I/O interface such as EEPROM.h
  * declare an instance of EDB
  * pick an address in EEPROM for the table to start

## Arduino Internal EEPROM Example ##
```Arduino
/*
   EDB_Simple.pde
   Extended Database Library + Internal Arduino EEPROM Demo Sketch

   The Extended Database library project page is here:
   http://www.arduino.cc/playground/Code/ExtendedDatabaseLibrary

 */

#include "Arduino.h"
#include <EDB.h>

// Use the Internal Arduino EEPROM as storage
#include <EEPROM.h>

// Uncomment the line appropriate for your platform
#define TABLE_SIZE 512 // Arduino 168 or greater

// The number of demo records that should be created.  This should be less
// than (TABLE_SIZE - sizeof(EDB_Header)) / sizeof(LogEvent).  If it is higher,
// operations will return EDB_OUT_OF_RANGE for all records outside the usable range.
#define RECORDS_TO_CREATE 10

// Arbitrary record definition for this table.
// This should be modified to reflect your record needs.
struct LogEvent {
    int id;
    int temperature;
}
logEvent;

// The read and write handlers for using the EEPROM Library
void writer(unsigned long address, byte data)
{
    EEPROM.write(address, data);
}

byte reader(unsigned long address)
{
    return EEPROM.read(address);
}

// Create an EDB object with the appropriate write and read handlers
EDB db(&writer, &reader);

void setup()
{
    Serial.begin(9600);
    Serial.println("Extended Database Library + Arduino Internal EEPROM Demo");
    Serial.println();

    EDB_Status result = db.open(0);
    if (result == EDB_ERROR) {
        db.create(0, TABLE_SIZE, sizeof(logEvent));
    }

    Serial.print("Record Count: "); Serial.println(db.count());

    Serial.println("Creating Records...");
    int recno;
    for (recno = 1; recno <= RECORDS_TO_CREATE; recno++)
    {
        logEvent.id = recno;
        logEvent.temperature = recno * 2;
        db.appendRec(EDB_REC logEvent);
    }

    Serial.print("Record Count: "); Serial.println(db.count());
    for (recno = 1; recno < RECORDS_TO_CREATE; recno++)
    {
        db.readRec(recno, EDB_REC logEvent);
        Serial.print("ID: "); Serial.println(logEvent.id);
        Serial.print("Temp: "); Serial.println(logEvent.temperature);
    }
}

void loop()
{
}
```


*This is a re-implementation of the database library originally written by Madhusudana das found here:*

http://www.arduino.cc/playground/Code/DatabaseLibrary
