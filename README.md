# bobbycar-foc-model

This repository implements Field Oriented Control (FOC) model for stock hoverboards. Compared to the commutation method, this new FOC control method offers superior performance featuring:
 - reduced noise and vibrations 	
 - smooth torque output and improved motor efficiency. Thus, lower energy consumption
 - field weakening to increase maximum speed range
 
 ---
 ## FOC model
 
 This new model offers 4 control modes:
  - **VOLTAGE MODE**: in this mode the controller applies a constant Voltage to the motors
  - **SPEED MODE**: in this mode a closed-loop controller realizes the input target speed by rejecting any of the disturbance (resistive load) applied to the motor
  - **TORQUE MODE**: in this mode the target torque set by the user is realized. This mode enables motor "freewheeling" when the torque target is "0".
  - **OPEN MODE**: Just reduces the input to 0. Since it is in software it will be exact 0. It will charge the battery, it is basically a fast recharge phase.
  
In all the modes, the controller features maximum motor speed and maximum motor current protection. This brings great advantages to fulfil the needs of many robotic applications while maintaining safe operation.
  - for critical error (blocked motor, Hall sensor not connected), enable = 0. So, the MOSFETs get interrupted immediately, to avoid HW damage.
  - for soft error (Serial timeout, ADC out of range), I request OPEN mode. The rate of opening is controlled by dV_openRate see here. Decrease this parameter if you want slower opening.

### Model architecture

The model architecture includes:
- **Estimations**: estimates the rotor position, angle and motor speed based on Hall sensors signal
- **Diagnostics**: implements error detection such as unconnected Hall sensor, motor blocked, MOSFET defective
- **Control Manager**: manages the transitions between control modes (Voltage, Speed, Torque)
- **FOC Algorithm**: implements the FOC strategy
- **Control Type Manager**: Manages the transition between Commutation, Sinusoidal, and FOC control type

![Firmware architecture](/docs/pictures/FW_architecture.png)

The FOC algorithm architecture is illustrated in the figure below:

![FOC algorithm](/docs/pictures/FOC_algorithm.png)

In this firmware 3 control types are available:
- Commutation
- SIN (Sinusoidal)
- FOC (Field Oriented Control)
![Schematic representation of the available control methods](/01_Matlab/02_Figures/control_methods.png)


### Field Weakening / Phase Advance

 - The Field Weakening is a linear interpolation from 0 to FIELD_WEAK_MAX or PHASE_ADV_MAX (depeding if FOC or SIN is selected, respectively)
 - The Field Weakening starts engaging at FIELD_WEAK_LO and reaches the maximum value at FIELD_WEAK_HI
 - The figure below shows different possible calibrations for Field Weakening / Phase Advance
 ![Field Weakening](/docs/pictures/FieldWeakening.png) 
 - If you re-calibrate the Field Weakening please take all the safety measures! The motors can spin very fast!


### Parameters 
 - All the calibratable motor parameters can be found in the 'BLDC_controller_data.c'. I provided you with an already calibrated controller, but if you feel like fine tuning it feel free to do so 
 - The parameters are represented in Fixed-point data type for a more efficient code execution
 - For calibrating the fixed-point parameters use the [Fixed-Point Viewer](https://github.com/EmanuelFeru/FixedPointViewer) tool
 - The parameters data Fixed-point types are given in the following table:

![Parameters table](/docs/pictures/paramTable.png)


### Diagnostics
Each motor is constantly monitored for errors. These errors are:
- **Error 001**: Hall sensor not connected
- **Error 002**: Hall sensor short circuit
- **Error 004**: Motor NOT able to spin (Possible causes: motor phase disconnected, MOSFET defective, operational Amplifier defective, motor blocked)

The error codes above are reported for each motor in the variables **rtY_Left.z_errCode** and **rtY_Right.z_errCode** for Left motor (long wired motor) and Right motor (short wired motor), respectively. In case of error, the motor power is reduced to 0, while an audible (fast beep) can be heard to notify the user.


### General Notes
   - The C code for the controller was auto-code generated using [Matlab/Simulink](https://nl.mathworks.com/solutions/embedded-code-generation.html) from a model which I developed from scratch specifically for hoverboard control. For more details regarding the working principle of the controller please consult the [Matlab/Simulink model](/01_Matlab).
 - A [webview](/01_Matlab/BLDC_controller_ert_rtw/html/webview) was created, so Matlab/Simulink installation is not needed, unless you want to regenerate the code. The webview is an html page that can be opened with browsers like: Microsoft Internet Explorer or Microsoft Edge.


### Demo videos

[►Video: Commutation vs Advanced control (constant speed)](https://drive.google.com/open?id=1vC_kEkp2LE2lAaMCJcmK4z2m3jrPUoBD)

[►Video: Commutation vs Advanced control (variable speed)](https://drive.google.com/open?id=1rrQ4k5VLhhAWXQzDSCar_SmEdsbM-hq2)

[►Video: Reliable Serial Communication demo](https://drive.google.com/open?id=1mUM-p7SE6gmyTH7zhDHy5DUyczXvmy5d)

[►Video: HOVERCAR demo](https://drive.google.com/open?id=18IvRJVdQSsjTg1I0Wedlg19e0FuDjfdS)


---

## Acknowledgements

Last but not least, I would like to acknowledge and thank the following people:
- Original firmware: [@NiklasFauth](https://github.com/NiklasFauth)
- Github: [@TomTinkering](https://github.com/TomTinkering), [@ced2c](https://github.com/ced2c), [@btsimonh](https://github.com/btsimonh), [@lalalandrus](https://github.com/lalalandrus), [@p-h-a-i-l](https://github.com/p-h-a-i-l) , [@AntumArk](https://github.com/AntumArk), [@juodumas](https://github.com/juodumas)
- Github: all the people that contributed via Pull Requests
- ST Employee: [cedric H](https://community.st.com/s/question/0D50X0000B28qTDSQY/custom-foc-control-current-measurement-dma-timer-interrupt-needs-review)


---

