# TEAM ANVESHAK
## Abex_ARC24 Documentation
The script relevant to this documentation can be found in the repository Anveshak-Files in the path "Abex_ARC24_updated/Abex_ARC24_updated.ino"
## TABLE OF CONTENTS
  1. Libraries used
  2. ROS topics used
  3. Subscriber - Publisher relationship
  4. Intuition on variables and functions
  5. Flow of control in the script
     
There are many topics used in this code, mainly accel_stepper.h and some other topics for sensors have been used in the code like Adafruit_Sensor.h, Adafruit_BMP085_U.h, DHT.h, DallasTemperature.h.

## Libraries Used

The `AccelStepper.h` library offers more advanced functionality for controlling stepper motors compared to the standard `Stepper.h` library provided by Arduino. It includes features such as acceleration control and other advanced parameters, which are not available in the `Stepper.h` library.

`Adafruit_Sensor.h`: Used to interface with Adafruit Sensors, like the DHT11 moisture sensor in our case.

`DHT.h` and `DHT_U.h`: Provides additional functionality on top of `Adafruit_Sensor.h`

`Wire.h`: enables I2C communication with the board

`Adafruit_BMP085_U.h`: Driver for the BMP085 barometric pressure sensor

`OneWire.h`: Enables serial communication with multiple devices using a single wire, to reduce the number of wires on the board.

`DallasTemperature.h`: Enables communication with multiple DS18B20 sensors over a single wire (with the OneWire protocol)

## Subscriber-Publisher Relationship

`sensor_data`: The script publishes all the received sensor data on this topic, for use by other nodes.

`motor_data_science`: Controls are sent through a GUI, and these are stored in an array `science_motors`. The values in this array are used to execute operations like dropping soil, running the pump, and performing spectrometry.



