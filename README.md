# Cartesian-Laser-Welding
Marlin Firmware for XY Laser Welder

These Configuration.h, Configuration.adv and pins_RAMPS.h files determine the setup of the XY robot.

For dual Y steppers:
Have to say there is one extruder 
  #define EXTRUDERS 1 in Configuration.h
Set dual Y steppers in Configuration_adv.h
  #define Y_DUAL_STEPPER_DRIVERS
  
Then make these changes in Pins_RAMPS.h
	○ E0 should be directed to unused pins
	○ Then, Marlin chooses next available extruder as 2nd Y axis pins, which you direct to what was previously E0 in order to use the motor driver connnector.
   In Pins_RAMPS.h:
   
		#define E0_STEP_PIN        23
		#define E0_DIR_PIN         25
		#define E0_ENABLE_PIN      27
		#ifndef E0_CS_PIN
		  #define E0_CS_PIN        29
		#endif
		
		#define E1_STEP_PIN        26
		#define E1_DIR_PIN         28
		#define E1_ENABLE_PIN      24
		#ifndef E1_CS_PIN
		  #define E1_CS_PIN        42
		#endif

D6 on servo header used for 100% duty cycle PWM laser trigger.
  M107 P1 S0 switches off / M106 P1 S255 switches on.
  
	#define FAN1_PIN 6 // 2nd fan output attached to laser TTL input in configuration.h

http://marlinfw.org/docs/configuration/laser_spindle.html 

Temp sensors TEMP_SENSOR_0 and TEMP_SENSOR_BED defined as 998 in configuration.h for dummy (no) sensor.

X and Y axis endstops uninverted, Z axis inverted because there are none and leaving them undefined caused an error.
Set this in Configuration.h.

DRV8825 drivers used for X, Y and Y2 motors.

Marlin Configuration for Head Speed, Step Rate
	(steps * microstepping) / (teeth * pitch) =  no. steps per mm 
	See calculator: http://www.prusaprinters.org/calculator/ 

  Default Axis Steps Per Unit (steps/mm) : X, Y, Z, E0 [, E1[, E2[, E3[, E4]]]]
  #define DEFAULT_AXIS_STEPS_PER_UNIT   { 200, 200, 200, 200 }

  With 16th steps and 400 steps per rev (0.9deg steps):
	(400*16)/(16*2)  200  steps per mm = pulses/mm
	pulses/mm * mm/s = pulses/s
	25 mm/s speed means 200*25=5,000 Hz pulse, half of the 10kHz hardware limit:	https://reprap.org/wiki/Step_rates
  Marlin/Repetier on ATmega 16 MHz (e.g. RAMPS) in single-stepping: <10,000 steps/second (10 kHz).

  Default Max Feed Rate (mm/s) : X, Y, Z, E0 [, E1[, E2[, E3[, E4]]]]
  #define DEFAULT_MAX_FEEDRATE          { 300, 300, 300, 300 }



Defining bed size (defined by endstop placement) in Configuration.h:
// The size of the print bed
#define X_BED_SIZE 296
#define Y_BED_SIZE 356

To place the origin at the centre of the bed and allow for motion to negative coords set travel limits in Configuration.h: 
// Travel limits (mm) after homing, corresponding to endstop positions.
#define X_MIN_POS -(X_BED_SIZE/2)
#define Y_MIN_POS -(Y_BED_SIZE/2)
#define Z_MIN_POS 0
#define X_MAX_POS (X_BED_SIZE/2)+1
#define Y_MAX_POS (Y_BED_SIZE/2)+1
#define Z_MAX_POS 1

And set:

	// The center of the bed is at (X=0, Y=0)
	#define BED_CENTER_AT_0_0


# Using DXF2GCode to Generate GCode
Download here: https://sourceforge.net/projects/dxf2gcode/

Edit the Configuration:
	Machine config:
	Set G1 feedrates as 1500 mm/min for X and Y (and Z)
	Retraction coordinate as 0.00 mm
	
Edit the Postprocessor Configuration: see postro_config.cfg
	Software config:
	Output file extension: .gcode
	Startup: 
	
		M17 (Enable motors)
		G28 X Y (Home X and Y axes before start)
		M107 P1 S0 (Turn off laser)
		G0 F5000 (Set rapid move feedrate)
End:
	
		M107 P1 S0 (Turn off laser)
     		M84 (Disable motors)
		M2 (Program end)

Edit the Gcode files to insert laser trigger before and after each shape, take out Z moves, edit feedrate, etc.
	
# Set up Pronterface
Download here:
Use 250000 baud rate to communicate with arduino.

Set up bed dimensions to match Marlin.

Set matching feedrates in case pronterface alters Marlin settings.

Useful GCode commands:

	G28 X Y for homing X and Y
	M84 (disable motors)
	M17 (Enable motors)
	M280 P0 Sxxx (for servo position adjustment, using 0-255 (0-100%) duty cycle)
