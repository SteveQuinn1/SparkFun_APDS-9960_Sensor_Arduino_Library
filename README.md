SparkFun APDS9960 RGB and Gesture Sensor Arduino Library -- Modified for esp8266 
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