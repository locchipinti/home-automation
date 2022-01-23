/*
 * Example for how to use the AC Unit
 * 
 * If you encounter any issues:
 * - check the readme.md at https://github.com/sinricpro/esp8266-esp32-sdk/blob/master/README.md
 * - ensure all dependent libraries are installed
 *   - see https://github.com/sinricpro/esp8266-esp32-sdk/blob/master/README.md#arduinoide
 *   - see https://github.com/sinricpro/esp8266-esp32-sdk/blob/master/README.md#dependencies
 * - open serial monitor and check whats happening
 * - check full user documentation at https://sinricpro.github.io/esp8266-esp32-sdk
 * - visit https://github.com/sinricpro/esp8266-esp32-sdk/issues and check for existing issues or open a new one
 */

// Uncomment the following line to enable serial debug output
//#define ENABLE_DEBUG

#ifdef ENABLE_DEBUG
#define DEBUG_ESP_PORT Serial
#define NODEBUG_WEBSOCKETS
#define NDEBUG
#endif

#include <Arduino.h>
#ifdef ESP8266
#include <ESP8266WiFi.h>
#include <IRremoteESP8266.h>
#include <IRsend.h>
#include <ir_Coolix.h>
#endif
#ifdef ESP32
#include <WiFi.h>
#endif

#include "SinricPro.h"
#include "SinricProWindowAC.h"

const uint16_t kIrLed = 4; // ESP8266 GPIO pin to use. Recommended: 4 (D2).
IRCoolixAC ac(kIrLed);     // Set the GPIO used for sending messages.

#define WIFI_SSID "WIFI_SSID"
#define WIFI_PASS "WIFI_PASS"
#define APP_KEY "APP_KEY"           // Should look like "de0bxxxx-1x3x-4x3x-ax2x-5dabxxxxxxxx"
#define APP_SECRET "APP_SECRET_KEY" // Should look like "5f36xxxx-x3x7-4x3x-xexe-e86724a9xxxx-4c4axxxx-3x3x-x5xe-x9x3-333d65xxxxxx"
#define ACUNIT_ID "DEVICE_ID"       // Should look like "5dc1564130xxxxxxxxxxxxxx"
#define BAUD_RATE 9600              // Change baudrate to your need

float globalTemperature;
bool globalPowerState;
int globalFanSpeed;

bool onPowerState(const String &deviceId, bool &state)
{
    Serial.printf("Thermostat %s turned %s\r\n", deviceId.c_str(), state ? "on" : "off");
    globalPowerState = state;
    Serial.printf("Status: %s", state ? "on" : "off");
    if ("%s", state ? "on" : "off" == "on")
    {
        Serial.println("Turning on A/C");
        ac.on();
        ac.setSwing();
        ac.send();
    }
    else
    {
        ac.off();
        ac.send();
    }
    return true; // request handled properly
}

bool onTargetTemperature(const String &deviceId, float &temperature)
{
    Serial.printf("Thermostat %s set temperature to %f\r\n", deviceId.c_str(), temperature);
    globalTemperature = temperature;
    ac.setTemp((int)globalTemperature);
    ac.send();
    return true;
}

bool onAdjustTargetTemperature(const String &deviceId, float &temperatureDelta)
{
    globalTemperature += temperatureDelta; // calculate absolut temperature
    Serial.printf("Thermostat %s changed temperature about %f to %f", deviceId.c_str(), temperatureDelta, globalTemperature);
    temperatureDelta = globalTemperature; // return absolut temperature
    return true;
}

bool onThermostatMode(const String &deviceId, String &mode)
{
    Serial.printf("Thermostat %s set to mode %s\r\n", deviceId.c_str(), mode.c_str());
    if (mode == "COOL")
    {
        Serial.println("Setting A/C in COOL mode");
        ac.setMode(kCoolixCool);
        ac.send();
    }
    if (mode == "AUTO")
    {
        Serial.println("Setting A/C in AUTO mode");
        ac.setMode(kCoolixAuto);
        ac.send();
    }
    if (mode == "ECO")
    {
        Serial.println("Setting A/C in FAN mode");
        ac.setMode(kCoolixFan);
        ac.send();
    }
    if (mode == "HEAT")
    {
        Serial.println("Setting A/C in HEAT mode");
        ac.setMode(kCoolixHeat);
        ac.send();
    }
    if (mode == "OFF")
    {
        Serial.println("Setting A/C Off");
        ac.off();
        ac.send();
    }
    return true;
}

bool onRangeValue(const String &deviceId, int &rangeValue)
{
    Serial.printf("Fan speed set to %d\r\n", rangeValue);
    globalFanSpeed = rangeValue;
    return true;
}

bool onAdjustRangeValue(const String &deviceId, int &valueDelta)
{
    globalFanSpeed += valueDelta;
    Serial.printf("Fan speed changed about %d to %d\r\n", valueDelta, globalFanSpeed);
    valueDelta = globalFanSpeed;
    return true;
}

void setupWiFi()
{
    Serial.printf("\r\n[Wifi]: Connecting");
    WiFi.begin(WIFI_SSID, WIFI_PASS);

    while (WiFi.status() != WL_CONNECTED)
    {
        Serial.printf(".");
        delay(250);
    }
    IPAddress localIP = WiFi.localIP();
    Serial.printf("connected!\r\n[WiFi]: IP-Address is %d.%d.%d.%d\r\n", localIP[0], localIP[1], localIP[2], localIP[3]);
}

void setupSinricPro()
{
    SinricProWindowAC &myAcUnit = SinricPro[ACUNIT_ID];
    myAcUnit.onPowerState(onPowerState);
    myAcUnit.onTargetTemperature(onTargetTemperature);
    myAcUnit.onAdjustTargetTemperature(onAdjustTargetTemperature);
    myAcUnit.onThermostatMode(onThermostatMode);
    myAcUnit.onRangeValue(onRangeValue);
    myAcUnit.onAdjustRangeValue(onAdjustRangeValue);

    // setup SinricPro
    SinricPro.onConnected([]()
                          { Serial.printf("Connected to SinricPro\r\n"); });
    SinricPro.onDisconnected([]()
                             { Serial.printf("Disconnected from SinricPro\r\n"); });
    SinricPro.begin(APP_KEY, APP_SECRET);
}

void printState()
{
    // Display the settings.
    Serial.println("Samsung A/C remote is in the following state:");
    Serial.printf("  %s\n", ac.toString().c_str());
}

void setup()
{
    Serial.begin(BAUD_RATE);
    Serial.printf("\r\n\r\n");
    setupWiFi();
    setupSinricPro();
    ac.begin();
    Serial.println("Setting initial state for A/C.");
    ac.off();
    ac.setFan(kCoolixFanAuto);
    ac.setMode(kCoolixCool);
    ac.setTemp(26);
    printState();
}

void loop()
{
    SinricPro.handle();
}