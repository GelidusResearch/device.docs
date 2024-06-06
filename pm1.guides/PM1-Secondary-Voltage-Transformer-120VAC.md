## Installing a Secondary 6VAC Reference Voltage Transformer

Any 120VAC to 6VAC mains connected AC transformer signal can be used as a reference for highly accurate current and phase metering on the PM1.

To use a secondary voltage source JP2 on the PCB must be cut open and the transformer output connects to the 6v-AC input. Polarity only matters if you need to use the phase angle data. If you observe CT1 and CT2 phases are in sync or very close then your 6V-AC inputs need to be swapped.

<div style="display: flex; justify-content: space-between 0;">
    <img src="../images/pm1/6vac.connector.jp2.jpg" alt="Select" style="width: 200px;"/>
    <img src="../images/pm1/6vac.connector.top.jpg" alt="Select" style="width: 200px;"/>
</div>
<br><br>
The PM1RVT120 or PM1RVT120KIT will have a calibration value near 4671
<br><br>
Edit your YAML to match it as follows.

```yaml
  phase_a_current_cal: '15270'
  phase_c_current_cal: '15270'
  phase_a_voltage_cal: '4470'
  phase_c_voltage_cal: '4671'
  ```


## Connecting the PM1 and a secondary transfomer

The PM1 should be supplied from L1 which will be the odd numbered breakers. L1 will be referenced from the PM1's on board separate 6VAC transformer winding. If you have a secondary 6VAC reference voltage tranformer then it should be connected to L2 which will be even numbered.

<div style="display: flex; justify-content: space-between 0;">
    <img src="../images/pm1/200A.split.phase.panel.jpg" alt="Select" style="width: 500px;"/>
</div>
<br><br>

Once physically connected you can do a measured calibration check using the procedure defined at https://esphome.io/components/sensor/atm90e32 The procedure would adjust for minor component variances. The precalibrated values in the YAML should be within +-1% of actual.