# tuni-m12
This is a central repository for some software modules developed for M12 machine at Tampere University [Innovative Hydraulics and Automation unit (IHA)](https://research.tuni.fi/iha/)

This repo also serves as the updatable storage for M12 documentation. It will contain most of the machines components datasheets and schematics on how they are connected.

# What is M12?
M12 is a heavily customized wheel loader at Tampere University. It is a research platform for mobile hydraulics and autonomous machine research, equipped with multiple fluid power and exteroceptive sensors.


![Image of M12](https://github.com/ahonena/tuni-m12/blob/master/doc/photos/IMG_20200902_133649.jpg)

You can see more of the [university research equipment here](https://research.tuni.fi/iha/research/autonomous-off-road-machines/).

# M12 Computer and communication hardware
## Real-time PC
M12 has one iEi tank-720 installed with preempt_rt patched Linux as the operating system. The tank-720 also has 2 PEAK PCAN-miniPCIe Dual Channel can bus cards, and one motherboard native CAN bus port.
It is connected to one CAN-bus network and one Ethernet local area network. The M12 CAN bus network is connected to two ports, one PEAK can card channel and the motherboard native CAN channel. This is done to make it easier on the software level to transmit and receive "simultaneously".

The PC also has USB cable connector outside and VGA port for direct access.
## Low level control
For low-level hydraulic machine control, there are two Bosch Rexroth BODAS-controllers installed: RC36 and RC28. Their detailed function will not be explained here. They are connected to the 24V battery on M12, and they need to be switched on in order to be able to move the machine. 
## Communication
The machine also has rugged WiFi access point installed and an onboard ethernet switch with one free CAT-cable connected to it for direct access to the LAN.

# M12 localization related sensors
M12 has many sensors installed, and only the ones related to localization and kinematics are listed here.

## Trimble BD982 GNSS receiver with dual antennas
The GNSS receiver has two antennas, which means that it can give out orientation information even if it is not moving. It has been configured to send out UDP-based GSOF messages, GSOF is a Trimble specific protocol. It also uses differential GPS signal for accuracy.

You can find more information on the BD982 [here](https://www.trimble.com/precision-gnss/BD982-Board.aspx?tab=support).

You can find more information on GSOF [here](https://www.trimble.com/OEM_ReceiverHelp/v4.85/en/Default.html#GSOFmessages_Overview.html).

## ADIS16385 Inertial Measurement Unit
This IMU is attached to the M12 CAN bus and has been configured to send out measurement information periodically. 

More detailed information on the IMU CAN messages can be found from [this spreadsheet document](https://github.com/ahonena/tuni-m12/blob/master/doc/CANmessages.xlsx), COB-ID (gyro) 0x1B2 (hexadecimal) / 434 (decimal).
COB-ID (acceleration) 0x2B2 (hexadecimal) / 690 (decimal).

## Axiomatic AXRES resolver
This resolver measures the angular position and the angular velocity of the central axis. It is connected to the M12 CAN bus and has been configured to send out measurement information periodically. 

More detailed information on the resolver CAN messages can be found from [this spreadsheet document](https://github.com/ahonena/tuni-m12/blob/master/doc/CANmessages.xlsx), COB-ID 0x19F (hexadecimal) / 415 (decimal).

## Bosch Rexroth A6VM speed sensor, attached to hydraulic motors
This speed sensor measures the hydrostatic drive motor speed. The odometry is estimated based on combining the information from the resolver and this motor speed. It is connected to the BODAS RC36 control module, which in turn sends out this sensor information periodically through CAN bus.

More detailed information on the HSD speed CAN message can be found from [this spreadsheet document](https://github.com/ahonena/tuni-m12/blob/master/doc/CANmessages.xlsx), COB-ID 0x485 (hexadecimal) / 1157 (decimal).

## Micro-Epsilon WDS-750-P60-CR-P Wire sensor + Gefran cylindrical displacement potentiometer: tilt and/or lift
These sensors are attached to the wheel loaders tilt and lift, the main work hydraulic tools. They are connected to the BODAS RC36 control module, which in turn sends out these sensors information periodically through CAN bus.

More detailed information on the tilt and lift position CAN message can be found from [this spreadsheet document](https://github.com/ahonena/tuni-m12/blob/master/doc/CANmessages.xlsx),
COB-ID 0x186 (hexadecimal) / 390 (decimal).


# M12 major software parts
## DDS interfacing of sensors
There are 2 executables for accessing sensor information. One is for CAN bus based sensors and one is for the GNSS.

[CAN to RTI Connext DDS sensor interface git repository](https://github.com/ahonena/m12-can2dds-publishers)

[GNSS RTI Connext DDS interface git repository](https://github.com/ahonena/m12-gnss-publisher)

The localization module is implemented as a Simulink model, which is derived from earlier versions made for Simulink Real-Time. This model is used to generate code with [this Simulink code generation target for preempt_rt Linux](https://github.com/aa4cc/ert_linux). The generated code is then compiled and statically linked with RTI Connext C libraries. The resulting executable can then be copied into the tank-720 onboard PC.

[M12 Simulink model for generating code for real-time localization module (private repository for now)](https://github.com/ahonena/m12-localization-simulink)


## DDS teleoperation

The teleoperation consists of two parts, one that takes in joystick commands and sends it to DDS, and the second that receives the commands via DDS and sends out the commands to CAN bus.

[The git repository for the part that writes the received DDS commands to CAN bus](https://github.com/ahonena/m12-dds2can-subscribers)

## Jetson TX2 hardware accelerated video compression DDS publisher

TBD
