SparkFun APDS9960 RGB and Gesture Sensor Arduino Library -- Modified for NonBlocking Use 
=========================================================

Made a few changes to get this to work with esp8266.

The main two are:
* Removing wire.begin() from SparkFun_APDS9960.cpp and moving it into the examples so that the pins it uses can be specified in your sketch

* Changed the LED_BOOST_300 to LED_BOOST_100 in SparkFun_APDS9960.cpp as I couldn't get the gesture sensor to work without changing this

See Sparkfun's original library here https://github.com/sparkfun/SparkFun_APDS-9960_Sensor_Arduino_Library for usage.

SQ
--
Modified/Created the following to work with ESP8266-12 to support instructable

1.
int16_t SparkFun_APDS9960::readGesture()

Made this nonblocking. If there is a lot of sensor activity it causes the ESP8266-12 to crash/reset with a W/Dog failure due to continual looping in APDS9960 code. 
Introduced a loop counter (default of 10 interations).


2. 
bool SparkFun_APDS9960::init(int16_t loop_count_max)

Added 'loop_count_max' to allow user to determine how many loops the user would like to set before quitting readGesture()


3.
Created FIFO buffer overflow handler.

bool SparkFun_APDS9960::clearGFIFO()


SQ - Update 17/03/18
--------------------
So here's the thing...

I did a lot of googling looking for libraries for the APDS9660. For the most part I came across two. The SparkFun and AdaFruit ones. There are other
variants, but by and large they are only config tweaks of the above, just like my earlier attempt.
In both cases they contain blocking code in the readGesture call, a while(1){}; loop. Which will block while there is reflected (or incident) IR reception
at the sensor(s) or until it detects a valid gesture. Incident IR is also important if you configure the sensor to be sensitive, the presence of an IR source 
(at quite some distance) can also cause false triggering. Also important is how you mount the sensor in an enclosure, if the wall thickness is too great there 
will be local reflections which will swamp the detector and cause false INT events.
Now the problem with this blocking call is, if like me you want to carry out multiple activities on your target microcontroller platform, using the Arduino IDE and 
code toolchain this blocking call will screw up everything else you are trying to achieve (by this I mean it's not an interrupt driven RTOS). 
In my case I wanted to create an MQTT over WiFi capable IoT desktop device which reads local temp, humidity, ambient light levels, barometric pressure, keeps track of 
time, display all these parameters on an LCD, log to a uSD card in real time, read button inputs, write to output LEDs and monitor gestures to control things in my 
IoT infrastructure and all of which to be controlled by an ESP8266-12.
From a processing capability perspective the ESP8266-12 has more than enough 'clout' to do the above, however try as I might, the second the sensor detected
received IR illumination it would rapidly interrupt and hang in this blocking loop which would eventually cause the ESP8266 to reset, no matter how many 'yields' 
I liberally 'sprinkled' around the code. The same is true if something remained close to the sensor for any protracted period, such as an errant moggie!
I then spent a lot of time trying to decipher the Avago (now Broadcom) datasheet, which at first glance seems very comprehensive, though after many attempts to 
understand how the gesture sensor could be adapted to work in a non-blocking mode and having 'stuffed' in a 'shed load' of debug code. I gave up. I simply couldn't
get to the bottom of it in the time I had available.
After an exchange with SparkFun's Shawn Hymel via YouTube it dawned on me that both SparkFun and Adafruit had most likely just ripped some example source from Avago 
(Shawn saying as much).

My comment is at the bottom of the comments section;

https://www.youtube.com/watch?v=J61_PKyWjxU&t=196s

So, I contacted Broadcom who put me onto a local distributor, who promptly ignored my request for information. I guess I'm just not worth their time, as is
often the case in these circumstances.

Not put off by this I decided I would relocate all the code and library for reading the gesture sensor to a second processor and have this processor communicate
with the ESP8266 via interrupts. ie. Read the sensor and process gestures in the background and communicate the gestures via interrupts foreground.

After trialling I2C, SPI and bespoke parallel as a means to communicate to the ESP8266 I settled on the latter.

ie. ATMega328P (with Arduino bootloader) connected to the APDS9660 via I2C communicating with the ESP8266 via a bespoke parallel interface protocol using a PCF8574 as 
parallel to serial conversion.

I have created the following GitHub repository for the result.

https://github.com/SteveQuinn1/APDS9960_NonBlocking

In creating this NonBlocking repository I updated my earlier APDS9660 check in to reflect this change of use with the following deltas;

1. Removed init(int16_t loop_count_max), loop_count_max parameter and legacy code.
2. Pre-fixed constants with 'APDS9960_'
3. De-sensitised the initial configuration parameters Reduced GGAIN from 4X to 1X, Reduced GLDRIVE from 25mA to 12.5mA
4. Moved 'bool wireReadDataByte(uint8_t reg, uint8_t &val);' from Private to public. To allow for better debugging if needed.
