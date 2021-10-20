# Duet-PS_ON
This is purely an optional mod for Duet 2 Wifi or Duet 3 with or without a RPI. In theory this could work with other boards that use the PS_ON functionality but I have no means to test.  

After upgrading to a Duet 3 and Raspberry Pi, I powered the RPI from the Duet and kept the original wiring setup.  
  Hold the power button(normally 1-2 seconds) until the board powers up and sends M80 to PS_ON and ground the SSR.  

With a Raspberry Pi 4B I had to hold the power button for about 20 seconds for the Raspberry Pi to boot up before I could let go of the button.  
For this mod to work you'll have to use a separate RPI power supply and remove the 5V -> SBC jumper.  
The RPI will always be on unless you manually shut it down.  
The printer will be in a low power state.  
The LCD will be on and your printer will be in "Standby" mode.  
You can power on the Duet by either pressing the button or using the PSU control in DWC.  

# BOM  
[5v relay board](https://www.amazon.com/dp/B00LW15A4W/ref=cm_sw_em_r_mt_dp_N1P35HN3BCPKBMJH969X)   
[5V PSU](https://www.amazon.com/dp/B005T6UJBU/ref=cm_sw_em_r_mt_dp_H0TYRGBDHGWBVFF8ZMYS)  
[Momentary Push Button Switched Wired NO/C](https://www.aliexpress.com/item/4000094832237.html?spm=a2g0s.9042311.0.0.2be14c4daTSuak)  
Official RPI power supply-Not needed if you're not using a RPI  

# Wiring  
The wiring is pretty straight forward.  

* 5V PSU  
Load -> AC Load  
Neutral -> AC Neutral  
Ground -> AC Ground  
+V -> Relay 5V  
+V -> Relay Signal  
-V -> Duet EXT 5V GND  

* 24V PSU  
Load -> AC Load  
Neutral -> C terminal on Relay  
Ground -> AC Ground  
+V -> Duet POWER IN VIN  
-V -> Duet POWER IN GND  

* Relay  
GND -> Duet EXT 5V PSON  
5V -> 5V +V  
Signal -> 5V +V  
C -> 24V Neutral  
NO -> AC Neutral  

* Duet  
** The IO pins you use don't have to be the same as the ones in this tutorial. Just make sure you change the configs and wiring accordingly.    
EXT 5V +5V -> 5V PSU +V  
EXT 5V PSON -> Relay GND  
EXT 5V GND -> 5V PSU -V  
POWER IN GND -> 24V PSU -V    
POWER IN VIN -> 24V PSU +V  
Io_8.GND -> Momentary Push Button C  
Io_8.in -> Momentary Push Button NO  

# Configs  
To make this work we'll configure External Triggers.  
-Note; you cannot use trigger 0 or 1. 2 is the 1st free trigger you can assign  
https://duet3d.dozuki.com/Wiki/Gcode#Section_M581_RepRapFirmware_3_01RC2_and_later  

In the System directory(same place where config.g is stored) create a trigger file called "trigger2.g".  
In the trigger file you only need to add  
`M80           ; Power on `  
Save trigger2.g file.  

You'll need to add 2 lines to your config.g file.  
`M950 J1 C"io8.in"`  
1. M950 = Create GPIO pin  
2. J1 = Input pin number  
3. C"io8.in" = Input pin io8.in  

`M581 T2 P1 S1 R0`  
1. M581 Configure External Trigger  
2. T2 = Trigger2  
3. P1 = Input pin number/J# in M950  
4. S1 = Inactive-to-active (momentary button wired with NO and C)  
5. R0 = Trigger condition at anytime  


Taking it a step farther you can also use the power button to turn the printer on and off by using conditional G code. Quick press will turn the power on, long press will turn the power off. Paste the following into your trigger file:    
```
; Script to control power on button
; in config.g:
; M950 J1 C"!0.io4.in"			; Create GPIO pin for On button wired NO-must be inverted with !
; M581 T2 P1 S1 R0		        ; T2-Run Trigger 2; P1-J1; S1-When button pressed; R0-trigger any time
; Trigger file created for on/off button
M80								; Power on
G4 P200							; wait 2ms
M98 P"config.g"					; run config.g to re-recognize toolboard
G4 P200							; wait 2ms
if sensors.gpIn[1].value = 1 	; check if button still pressed
	M291 P"Power off" S1 T2 	;
	G4 S1						; give time to let go of the button to prevent accidental power on
    M81 						; turn off power  
