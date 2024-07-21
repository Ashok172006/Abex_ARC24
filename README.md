# TEAM ANVESHAK
## Abex_ARC24 Documentation
The script relevant to this documentation can be found in the repository Anveshak-Files in the path "Abex_ARC24_updated/Abex_ARC24_updated.ino"
## TABLE OF CONTENTS
  1. Libraries used
  3. Subscriber - Publisher relationship
  4. Intuition on variables and functions
  5. Flow of control in the script


## Libraries Used

The `AccelStepper.h` library offers more advanced functionality for controlling stepper motors compared to the standard `Stepper.h` library provided by Arduino. It includes features such as acceleration control and other advanced parameters, which are not available in the `Stepper.h` library.

`Adafruit_Sensor.h`: Used to interface with Adafruit Sensors, like the DHT11 humidity and temperature sensor in our case.

`DHT.h` and `DHT_U.h`: Provides additional functionality on top of `Adafruit_Sensor.h`

`Wire.h`: enables I2C communication with the board

`Adafruit_BMP085_U.h`: Driver for the BMP085 barometric pressure sensor

`OneWire.h`: Enables serial communication with multiple devices using a single wire, to reduce the number of wires on the board.

`DallasTemperature.h`: Enables communication with multiple DS18B20 sensors over a single wire (with the OneWire protocol)

## Subscriber-Publisher Relationship

`sensor_data`: The script publishes all the received sensor data on this topic, for use by other nodes.

`motor_data_science`: Controls are sent through a GUI, and these are stored in an array `science_motors`. The values in this array are used to execute operations like dropping soil, running the pump, and performing spectrometry.

## Variables

### Control Variables

These variables manage the overall behaviour of the script.

`sciencemotors[]`: stores commands received from the GUI, and controls motor actions and sensor data collection. It is a 1 dimensional list of integers.

`encoderCount`: Tracks position of encoder for motor control

`stepsPerRevolution`, `gearRatio`, `stepsPerOutputRevolution`: Parameters for the motor's step count and gear ratio.

### Sensor Data Variables

`soilMoisture_a`, `soilMoisture_b`: stores readings from soil moisture sensors.

`dhtDelayMS`: Determines the delay for DHT sensor readings.

`sensor_data`: Holds the array of all sensor readings to be published.

### Supporting Variables

`oneWireBus`: Defines the pin for the OneWire bus used by DS18B20 sensors.

`init_list[]`: Used to initialize the sensor data array.

`sensor`: stores temperature and humidity readings from DHT sensor.

`sensors`: stores temperature readings from DS18B20 sensors.

`stepper`: Object for controlling the stepper motor.


## Functions

### Setup Function

- Initializes ROS communication using a ROS NodeHandle `nh`.

- Initialize and configure DHT sensor:

```python
dht.begin();
sensor_t sensor;
dht.temperature().getSensor(&sensor);
dhtDelayMS = sensor.min_delay / 1000;
```

- Initialize I2C communication for BMP180:

```python
Wire.begin(SDA_PIN, SCL_PIN);
```

- Initializes DS18B20 Sensor with OneWire protocol
  
```python
sensors.begin();
```

- Initialize Sensor Data Array with `sensor_data.data`
- configure GPIO pins for sensors, motors and encoders
- set max speed and acceleration for stepper motor

```python
stepper.setMaxSpeed(20); // Adjust as needed
stepper.setAcceleration(10); // Adjust as needed
```
- Attach Interrupt for Encoder:

This allows encoder data to be processed as soon as they are received, instead of waiting for the main loop completion.

```python
attachInterrupt(digitalPinToInterrupt(ENCA), encoderISR, CHANGE);
```




### Loop Function
In this Function we are using the `science_motors` array, which contains the commands received from a GUI. This allows a user to control motor movements and sensor readings.


#### Main Control Loop

1.`Motor Rotation`:
  * If sciencemotors[0] is non-zero, it calls rotateMotor(-90) to rotate the motor by -90 degrees and then resets sciencemotors[0] to 0.

2.`Pump Control`:
  * If sciencemotors[1] is non-zero, it calls runpumps(sciencemotors[1]) to control the pumps and then resets sciencemotors[1] to 0.

3.`Stepper Motor Movement`:
  * If sciencemotors[2] is 1, it calls movestepper(90) to move the stepper motor by 90 degrees.
  * If sciencemotors[2] is 2, it calls movestepper(-90) to move the stepper motor by -90 degrees.
It then resets sciencemotors[2] to 0.

4.`Sensor Data Collection and Publishing`:
  * If sciencemotors[3] is 1, it collects data from various sensors:
      * Temperature and humidity from DHT11.
      * Soil moisture from analog sensors.
      * Pressure and altitude from BMP180.
      * Temperature from DS18B20.
Publishes the collected data to the `sensor_data` topic.
Resets sciencemotors[3] to 0.

5.`Delay`:
  * The loop includes a delay of 50 milliseconds.

### MoveStepper
* Steps
  * The movestepper function controls a stepper motor, rotating it by a specified `angle`. This angle can be either positive or negative.
  * Now using the angle which we got, We calculate the number of steps required to rotate the stepper motor by the specified angle.
  * Assuming the stepper motor has 200 steps per revolution (1.8 degrees per step), the calculation is:
    ```python
    long steps = (200 / 360.0) * abs(angle);
* Get the new position
  * now we target to get the new position
  * by the following command, we change the stepper.currentPosition() to the new position:
    ```python
    stepper.moveTo(stepper.currentPosition() + steps);
* Move the motor
  * Now we ove the stepper motor by the following command
    ```python
    stepper.runToPosition();
    
### Runpumps
The runpumps function controls the activation of two different pumps connected to digital pins(one of them is connected to DR12 and other to DR21). The function takes an integer argument `a` which determines which pump to activate and for how long. Let pump1 be the pump connected to DR12 and pump2 be the pump connected to DR21

* a==1
  * Pump1 activates and runs for 8 seconds(delay is given in the function for the pump to opearate) and then it stops.
    
* a==2
  * Pump2 activates and runs for 8 seconds(delay is given in the function for the pump to opearate) and then it stops.
  
* a==3
  * Pump1 activates and runs for 2 seconds(delay is given in the function for the pump to opearate) and then it stops.
    
* a==4
  * Pump2 activates and runs for 2 seconds(delay is given in the function for the pump to opearate) and then it stops.

### Rotate motor
The rotateMotor function is designed to rotate a Johnson motor to a specified angle using an encoder for feedback. It sets the motor's direction and runs it until the desired angle is achieved. 
* Similar to the stepper motor function, here also we calculate Steps.
  ```python
  long steps = (stepsPerOutputRevolution / 360) * abs(angle);
  
In this, stepsPerOutputRevolution is a variable which is breifly described above.
We reset encodercount as 0. Encodercount here is just used for the operation in the while loop.
Now,
* angle > 0
  * The motor is set to rotate in one direction by setting John1 high and John2 low.
* angle < 0
  * The motor is set to rotate in the opposite direction by setting John1 low and John2 high.
* Rotate motor
  * The motor is allowed to run until the absolute value of the encoder count reaches the calculated steps:
    ```python
    while (abs(encoderCount) < steps) delay(10);
  * The motor continues to rotate during this loop, and the encoder interrupt service routine (ISR) updates the encoderCount to provide feedback
* Stop Motor
  * After the motor has rotated the desired number of steps, both John1 and John2 are set low to stop the motor.

### encoderISR
The encoderISR function is an Interrupt Service Routine (ISR) designed to handle encoder pulses and update the encoder count based on the direction of rotation. This function is used to track the position of a DC motor shaft accurately.
* Read Encoder States
  * The function reads the current states of the two encoder channels (A and B) using digitalRead:
    ```python
    int stateA = digitalRead(ENCA);
    int stateB = digitalRead(ENCB);
* Update Encoder Count
  * The function compares the states of channels A and B:
     ```python
     if (stateA == stateB) encoderCount++;
     else encoderCount--;
  * If the states of both channels are the same (stateA == stateB), the encoderCount is incremented, indicating forward rotation and if they are not equal, the encoderCount is decremented indicating reverse direction.






