## Overview
<p> This hat is based on TMC5130A-TA Stepper driver. It supoorts up to 2A (2.5A peak) coil current and voltage range of 6.2V, 9V, 12V, 24V and VIN for drving motor <p>

  ## Functionality
  ### Connections
<p> -J6: Encoder conector, from pin 0 is: ENCN_DCO. ENCB_DCEN_CFG4, ENCA_DCIN_CFG5</p>
<p> -J7: Motor Connector, from pin 0 is: B1 B2 A2 A1</p>
<p> -RV1: Trimmer potentiomter for AIN_REF</p>

  ### Selecting voltage
  <p> There are two mechanism for selecting voltage </p>
  <p> - First: You can either connect buck converter or XT60 with stepper driver by J8. Note: XT60 VIN mode has 3 diodes (D5, D6, D7) in series to accomodate Chainbus max voltage range  </p>
  <p> - Second: When using buck converter can select 24V, 12V, 9V, 6.2V by toggling the SW1. Make sure to change the configuration only when disconnected from power, and toggle only one of the voltage ranges at once</p>
  <p> Note: you can change the selectable voltage levels by changing R14, R15 or R16</p>
    
  ### Programming
  <p> IC communicates by SPI. To connect to it you only need to select the board by chainbus indexing system. CS is connected by bus switch directly to stepper driver</p>
