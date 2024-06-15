> [!IMPORTANT]
> This macro is in development and performance is not guarenteed. Please provide feedback on any issues or make PRs for changes.


# Beacon Thermal Expansion Compensation
This is a macro set that will calibrate and apply a g_code Z offset to compensate for thermal expansion of the hotend from standard probe temperatures (~150-180) to print temperatures (~220-350C). The macro calibrates an expansion coefficent and calculates a specific z-offset based on the print temperature.

# What the macro is doing

The macro will home all axis and perform a QGL/Tilt using the printer default settings. After that it performs 10 poke operations to settle the printer mechanics. The macro will then heat to 150C, wait for the temperature to stabilize for 60s and perform a probe operation. This data is saved and then the hotend is heated to 250C for another stabilisation and probe operation. The macro then runs these two probe operations again to ensure the results are repeatable. The hotend is moved back to a central position and the nozzle expansion data saved to the variables file. Depending on how fast the hotend heats and cools down this could take 5-10min to complete.

A seperate macro called in print start will pull the nozzle expansion coefficent and extruder print temperature to calculate and apply the appropriate z_offset automatically at print time.

The expansion offset calibration should only need to be calibrated one time for a given mechanical setup. The variable "beacon_contact_expansion_multiplier" could be tweaked to fine tune the offsets. Currently no strong guidance exists on this.


# Pre-Reqs
- Beacon sensor updated to latest firmware version
- Contact mode setup and enabled on the beacon. See [https://docs.beacon3d.com/contact/]
- [save_variables] configured. An example given below:
```
[save_variables]
filename: ~/printer_data/config/variables.txt
```
- Ooze mitigation if 250C nozzle temp will lead to filament ooze. ?Suggest to perform this without filament fitted?

# Steps
- Copy macro block into your config
- Review/adjust the default variables in BEACONTEMP (reccomended to use defaults right now)
- Adjust print_start and print_end macros as below
- Restart klipper
- Run BEACON_CALIBRATE_NOZZLE_TEMP_OFFSET from the command line
- Review the variables file to ensure there is an entry similar to
  ```
  [Variables]
  nozzle_expansion_coefficient = 0.04749999999994525
  ```
- Run a print!!!

# Print Start
Apply this to the print start macro to calculate and apply a GCODE offset Z_adjust to the print. Note that this replaces "SET_GCODE_OFFSET" or "SET_GCODE_OFFSET" gcode lines as reccomended in the Beacon Contact Docs.

```
# set nozzle thermal expansion offset
{% if printer.configfile.settings.beacon is defined %}
    _BEACON_SET_NOZZLE_TEMP_OFFSET 
{% endif %}
```

# Print End
This removes the applied offset at the end of the print.

```
# reset nozzle thermal expansion offset
{% if printer.configfile.settings.beacon is defined %}
    _BEACON_REMOVE_NOZZLE_TEMP_OFFSET
    _BEACON_SET_NOZZLE_TEMP_OFFSET RESET=True
{% endif %}
```


# Credits

Credit to RatOS team for developing this macro. You can see their work in Beacon.cfg: https://github.com/Rat-OS/RatOS-configuration/blob/v2.1.x/z-probe/beacon.cfg

