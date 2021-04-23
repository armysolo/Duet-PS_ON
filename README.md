# Duet-PS_ON
This is purely an optional mod for users that have a RPI as a SBC powered from the Duet.  

After upgrading to a Duet 3 and Raspberry Pi I powered the RPI from the Duet and kept the original wiring setup.  
  Hold the power button until the board powers up and sends M80 to PS_ON and ground the SSR.  

With a Raspberry Pi 4B I had to hold the power button for about 20 seconds for the Raspberry Pi to boot up before I could let go of the button.  
For this mod to work you'll to use a separate RPI power supply and remove the 5V -> SBC jumper.  
The RPI will always be on unless you manually shut it down.  
The LCD will be on and your printer will be in "Standby" mode.  

# BOM  
[5v relay board](https://www.amazon.com/dp/B00LW15A4W/ref=cm_sw_em_r_mt_dp_N1P35HN3BCPKBMJH969X)   
[5V PSU](https://www.amazon.com/dp/B005T6UJBU/ref=cm_sw_em_r_mt_dp_H0TYRGBDHGWBVFF8ZMYS)  
[Momentary Push Button Switched Wired NO/C](https://www.aliexpress.com/item/4000094832237.html?spm=a2g0s.9042311.0.0.2be14c4daTSuak)  
Official RPI power supply  

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
Save trigger2.g.  

You'll need to add 3 lines to your config.g file.  
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

`M582 T2`
1. M582 = Check External Trigger
2. T2 = Trigger number assigned in M581


