# G-Code Customization (including Filament Loading and Unloading)
Happy Hare provides a few defined "callbacks" that, if they exist, will be called at specific times.  They are designed for you to be able to extend the base functionality and to implement additional operations.  For example, if you want to control your printers LED's based on the action Happy Hare is performing you would modify `_MMU_ACTION_CHANGED`.  All of the default handlers and examples are defined in `mmu_software.cfg` and serve as a starting point for modification.

<br>

## ![#f03c15](/doc/f03c15.png) ![#c5f015](/doc/c5f015.png) ![#1589F0](/doc/1589F0.png) _MMU_ACTION_CHANGED
Most of the time Happy Hare will be in the `Idle` state but it starts to perform a new action this macro is called.  The action string is passed as a `ACTION` parameter to the macro but can also be read with the printer variable `printer.mmu.action`

Possible action strings are:

```yml
    Idle           No action being performed
    Loading        Filament loading
    Unloading      Filamdng unloading
    Loading Ext    Loading filament into the extruder (usually occurs after Loading)
    Exiting Ext    Unloading filament from the extruder (usually after Foriming Tip and before Unloading)
    Forming Tip    When running standalone tip forming (cannot detect when slicer does it)
    Heating        When heating the nozzle
    Checking       Checking gates for filament (MMU_CHECK_GATES)
    Homing         Homing the selector
    Selecting      When the selector is moving to select a new filament
    Unknown        Should not occur
```

Here is the reference/example macro packaged in `mmu_software.cfg`:

```yml
###########################################################################
# This occurs when the MMU action status changes. The `ACTION` parameters will contain
# the current action string (also available in `printer.mmu.action` printer variable).
# Also the previous action is available in `OLD_ACTION`. See Happy Hare README for
# full list of action strings.
#
# This could be a place to set LED status or similar. The logic here
# demonstrates the three major action states
#
[gcode_macro _MMU_ACTION_CHANGED]
description: Called when an action has changed.
gcode:
    {% set ACTION = params.ACTION|string %}
    {% set OLD_ACTION = params.OLD_ACTION|string %}

    {% if ACTION|string == "Loading" %}
        # Add your logic here, e.g. loading LED
        # _STATUS_LOADING
    {% endif %}

    {% if ACTION|string == "Unloading" %}
        # Add your logic here, e.g. unloading LED
        # _STATUS_UNLOADING
    {% endif %}

    {% if ACTION|string == "Idle" %}
        # Add your logic here, e.g. LED off
        # _STATUS_STANDBY
    {% endif %}
```

<br>

## ![#f03c15](/doc/f03c15.png) ![#c5f015](/doc/c5f015.png) ![#1589F0](/doc/1589F0.png) _MMU_PRE_UNLOAD & _MMU_POST_LOAD
These two macros which are empty by default allow for a convenient place to add logic just before the unload of old filament and after the load of new filament on a toolchange. This allows for logic to move the toolhead to purge area to avoid ooze as well as to perform a tip cleaning procedure prior to continuing the print. Note that it is excepected that `_MMU_POST_LOAD` will return the toolhead to the position saved in `_MMU_PRE_UNLOAD`, however, Happy Hare has optional built in control of the z-hop height (set with `z_hop_height_toolchange` parameter) and also will ensure than the toolhead returns to the correct postion in all situations (like user corrected errors during the toolchange).

```yml
###########################################################################
# Callback macros for modifying Happy Hare behavour
# This occurs prior to unloading filament on a toolchange
#
# Typically you would move toolhead to a position where oozing is not a problem
# Note that the z_hop is automatically controlled by Happy Hare and is
# specified with the 'z_hop_height_toolchange' parameter
#
[gcode_macro _MMU_PRE_UNLOAD]
description: Optional pre unload routine for filament change
gcode:

###########################################################################
# Callback macros for modifying Happy Hare behavour
# This occurs after loading new filament on a toolchange
#
# Typically you would clean nozzle if equiped and return to previous position
# Note that restoration to original toolhead position is ensured by Happy Hare.
#
[gcode_macro _MMU_POST_LOAD]
description: Optional post load routine for filament change
gcode:
```

<br>

## ![#f03c15](/doc/f03c15.png) ![#c5f015](/doc/c5f015.png) ![#1589F0](/doc/1589F0.png) _MMU_ENDLESS_SPOOL_PRE_UNLOAD & _MMU_ENDLESS_SPOOL_POST_LOAD
If EndlessSpool is enabled, Happy Hare will unload the remains of the filament from the exhausted spool and load the new spool. These macros are called at the beginning and end of that sequence.  `_MMU_ENDLESS_SPOOL_PRE_UNLOAD` is called because Happy Hare initiates the tip forming and typically would move the toolhead to a suitable "park" position it doesn't ooze onto your print.  This is commonly exactly the same as your `PAUSE` macro and so that is what the default handler calls.<br>

`MMU_ENDLESS_SPOOL_POST_LOAD` is called after the MMU has loaded the new filament from the next spool in rotation to the nozzle the same way as a normal filament swap (the previously configured Pressure Advance will be restored). Typically this would be a place to purge additional filament if necessary (it really shouldn't be) and clean nozzle if your printer is suitably equipped. I.e. similar to a typical `RESUME` macro.<br>

Here are the default macros:

```yml
###########################################################################
# Callback macros for modifying Happy Hare behavour
# Note that EndlessSpool is an unsupervised filament change
# This occurs prior to MMU forming tip and ejecting the remains of the old filament
#
# Typically you would move toolhead to your park position so oozing is not a problem
# Note that the z_hop is automatically controlled by Happy Hare and is
# specified with the 'z_hop_height_toolchange' parameter
#
# This is probably similar to what you do in your PAUSE macro and you could simply call that here...
# (this call works with reference PAUSE macro supplied in client_macros.cfg)
#
[gcode_macro _MMU_ENDLESS_SPOOL_PRE_UNLOAD]
description: Pre unload routine for EndlessSpool changes
gcode:
    PAUSE

###########################################################################
# Callback macros for modifying Happy Hare behavour
# Note that EndlessSpool is an unsupervised filament change
# This occurs after MMU has loaded the new filament from the next spool in rotation
# MMU will have loaded the new filament to the nozzle the same way as a normal filament
# swap. Previously configured Pressure Advance will be retained.
# 
# This would be a place to purge additional filament if necessary (it really shouldn't be)
# and clean nozzle if your printer is suitably equipped.
# Note that restoration to original toolhead position is ensured by Happy Hare.
#
# This is probably similar to what you do in your RESUME macro and you could simply call that here...
# (this call works with reference RESUME macro supplied in client_macros.cfg)
#
[gcode_macro _MMU_ENDLESS_SPOOL_POST_LOAD]
description: Optional post load routine for EndlessSpool changes
gcode:
    RESUME
```

<br>

## ![#f03c15](/doc/f03c15.png) ![#c5f015](/doc/c5f015.png) ![#1589F0](/doc/1589F0.png) _MMU_FORM_TIP_STANDALONE
This is probably the most important aspect of getting a reliable MMU after basic calibration is complete. There is plenty written about tip forming and lots of advice in the forums.  What is important to understand here is that this macro mimicks the tip forming logic from SuperSlicer (almost identical to PrusaSlicer). Read SuperSlicer documentation for hints. That said, here are a few things you should know:
<ul>
<li>This macro will always be used when not printing, but you can elect to use it instead of your slicers logic by:</li>
 <ul>
  <li>Turning OFF all tip forming logic in your slicer</li>
  <li>Setting the `variable_standalone: 1` in the `T0` macro</li>
 </ul>
<li>When tuning if is useful to pull the bowden from your extruder, load filament with the `MMU_LOAD EXTRUDER_ONLY=1` command, then call `MMU_FORM_TIP` command (and not the macro directly) or better still `MMU_EJECT EXTRUDER_ONLY=1`</li>
 <ul>
  <li>The benefit of calling as desribed is the additional TMC current control and pressure advance restoration will occur so it exactly mimicks what will occur when called automatically later</li>
  <li>If calling `MMU_FORM_TIP` you will want to set `variable_final_eject: 1` so that the filament is fully ejected for inspection (MMU_EJECT will automatically do this and therefore is recommended)
  <li>Calling with `MMU_EJECT EXTRUDER_ONLY=1` will also report on the final parking position of the filament</li>
 </ul>
<li>Before you start tweaking, make sure the settings accurately represent the geometry of your extruder. The defaults are for my Voron Clockwork 2 extruder with Voron Revo hotend with 0.4mm tip</li>
<li>Lastely there is a setting called `parking_distance` which, if set, will determine the final resting place measured from the nozzle. This should be a postive number!</li>
</ul>

Here are the default values for tip forming.  These are the exact values I used for non PLA filaments (PLA seems to like skinny dip):

```yml
# Unloading and Ramming values - Initial moves to form and shape tip
variable_unloading_speed_start: 80     # Fast here to seperate the filament from meltzone (Very intitial retract SS uses distance of E-15)
variable_unloading_speed: 20           # Too fast forms excessively long tip or hair. Slow is better here UNLOADING_SPEED_START/COOLING_MOVES seems a good start
variable_ramming_volume: 0             # in mm3 SS default values = 2, 5, 9, 13, 18, 23, 27. Only Used to Simulate SS Ramming during standalone
variable_ss_ramming: 0                 # Set to 0 for standalone ramming (RAMMING_VOLUME) or tuning, 1 to let the slicer do it (i.e. turn off standalone)

# Cooling Move Values - To cool the tip formed and separate from strings
variable_cooling_tube_position: 35     # Dragon ST: 35, Dragon HF: 30, Mosquito: 30, Revo: 35, Phaetus Rapido HF: 43;  Measured from Top of Heater Block to Top of Heatsink
variable_cooling_tube_length: 10       # Dragon ST: 15, Dragon HF: 10, Mosquito: 20, Revo: 10, Phaetus Rapido HF: 22; Measured from Nozzle to Top of Heater Block
variable_initial_cooling_speed: 10     # Slow to solidify tip and cool string if formed.
variable_final_cooling_speed: 50       # High speed break the string formed. Too fast = tip deformation during eject. Too Slow = long string/no seperation
variable_toolchange_temp: 0            # Used if you want to lower temp during toolchanges default 0
variable_cooling_moves: 4              # 2-4 is a good start

# SkinnyDip values - To burn off VERY FINE hairs only (This is NOT for long tip reshaping)
variable_use_skinnydip: 1              # Tune this LAST, this is for removal of VERY FINE hairs only (Different than a long tip)
variable_skinnydip_distance: 30        # Start just under Cooling_tube_position and increase - Will depend on how much Ramming Volume is used
variable_dip_insertion_speed: 30       # Medium-Slow - Just long enough to melt the fine hairs. Too slow will pull up molten filament
variable_dip_extraction_speed: 70      # Around 2x Insertion speed, Prevents forming new hairs
variable_melt_zone_pause: 0            # in milliseconds - default 0
variable_cooling_zone_pause: 0         # in milliseconds - default 0 - If you need to adjust here its possible Dip Insertion too slow
variable_use_fast_skinnydip: 0         # Skip the toolhead temp change during skinnydip move - default 0

# Park filament ready to eject
variable_parking_distance: 35          # Final filament parking position after final cooling move, 0 will leave filament where it naturally ends up

# Final Eject - for standalone tuning only
variable_final_eject: 0                # default 0, enable during standalone tuning process to eject the filament

# The park position of the filament is relative to the nozzle and represents where the end of the filament is
# after tip forming. The park position is important and used by Happy Hare to finish unloading the extruder.
# A negative value means that Happy hare will automatically calculate the park position based on the aggregate
# movement of the extruder.  However, in some tip forming setups (e.g. filament cutter), the tip forming procedure
# must decide its own park position and report this back to Happy Hare by setting the output variable to a
# positive value.
variable_output_park_pos: -1           # The park position (output variable passed back to Happy Hare)
```

> [!NOTE]  
> The `output_park_pos` variable is used to pass the resultant position of the filament back to Happy Hare.  Only change this if you are doing something clever like filament cutting otherwise the default behavior of Happy Hare measuring where it ends up (i.e. -1 value)
> You can set the return value with:
> `SET_GCODE_VARIABLE MACRO=_MMU_FORM_TIP_STANDALONE VARIABLE=output_park_pos VALUE=`

<br>

## ![#f03c15](/doc/f03c15.png) ![#c5f015](/doc/c5f015.png) ![#1589F0](/doc/1589F0.png) _MMU_LOAD_SEQUENCE & _MMU_UNLOAD_SEQUENCE

> [!WARNING]  
> This is new EXPERIMENTAL functionality and as such is subject to change (with only a mild apology :-)

By default these macros are not called, however, if `gcode_load_sequence` or `gcode_unload_sequence` are enabled they will be.  The two default macros in `mmu_software.cfg` (copied here) will/should provide exactly the same logic as the internal logic using a set of provided "modular" loading/unloading functions. They are a good starting point.<br>

`mmu_software.cfg` contains futher examples for alternative MMU setups, but before experimenting it is essential to understand the state machine for filament position.  These states are as follows and the loading/unloading sequence must be capable of completing the load/unload sequence for any starting state.<br>

```mermaid
graph TD;
    UNLOADED --> START_BOWDEN
    START_BOWDEN --> IN_BOWDEN
    IN_BOWDEN --> END_BOWDEN
    END_BOWDEN --> HOMED_EXTRUDER
    END_BOWDEN --> EXTRUDER_ENTRY
    HOMED_EXTRUDER --> EXTRUDER_ENTRY
    EXTRUDER_ENTRY --> HOMED_TS
    EXTRUDER_ENTRY --> IN_EXTRUDER
    HOMED_TS --> IN_EXTRUDER
    IN_EXTRUDER --> LOADED
```

In additon to these states the macros are passed some additional information and hints about the context.  An important one is `FILAMENT_POS` which represents the position of the filament in mm either from "point 0" in the gate (load direction) or from the nozzle (unload direction).  Here are the default macros with additional information:<br>


```yml
###########################################################################
# ADVANCED: User modifable loading and unloading sequences
#
# By default Happy Hare will call internal logic to handle loading and unloading
# sequences. To enable the calling of user defined sequences you must add the
# following to your mmu_parameters.cfg
#
# gcode_load_sequence: 1	# Gcode loading sequence 1=enabled, 0=internal logic (default)
# gcode_unload_sequence: 1	# Gcode unloading sequence, 1=enabled, 0=internal logic (default)
#
# This reference example load sequence mimicks the internal ones exactly. It uses the
# high level "modular" movements that are all controlled by parameters defined in
# mmu_parameters.cfg and automatically keep the internal filament position state up-to-date.
# Switching to these macros should not change behavor and can serve as a starting point for
# your customizations
#
# State Machine:
# If you experiment beyond the basic example shown here you will need to understand
# the possible states for filament position.  This is the same state that is exposed
# as the `printer.mmu.filament_pos` printer variable. This internal state must be
# kept up-to-date and will need to be set directly as you progress through your
# custom move sequence.  At this time the state machine is non-extensible.
#
#        FILAMENT_POS_UNKNOWN = -1
#  L  ^  FILAMENT_POS_UNLOADED = 0
#  O  |  FILAMENT_POS_START_BOWDEN = 1
#  A  |  FILAMENT_POS_IN_BOWDEN = 2
#  D  U  FILAMENT_POS_END_BOWDEN = 3
#  |  N  FILAMENT_POS_HOMED_EXTRUDER = 4
#  |  L  FILAMENT_POS_EXTRUDER_ENTRY = 5
#  |  O  FILAMENT_POS_HOMED_TS = 6
#  |  A  FILAMENT_POS_IN_EXTRUDER = 7    # AKA Filament is past the Toolhead Sensor
#  v  D  FILAMENT_POS_LOADED = 8         # AKA Filament is homed to the nozzle
#
# Final notes:
# 1) You need to respect the context being passed into the macro such as the
#    desired 'length' to move because this can be called for test loading
# 2) The unload macro can be called with the filament in any position (states)
#    You are required to handle any starting point. The default reference
#    serves as a good guide
#
[gcode_macro _MMU_LOAD_SEQUENCE]
description: Called when MMU is asked to load filament
gcode:
    {% set filament_pos = params.FILAMENT_POS|float %}
    {% set length = params.LENGTH|float %}
    {% set full = params.FULL|int %}
    {% set home_extruder = params.HOME_EXTRUDER|int %}
    {% set skip_extruder = params.SKIP_EXTRUDER|int %}
    {% set extruder_only = params.EXTRUDER_ONLY|int %}

    {% if extruder_only %}
        _MMU_STEP_LOAD_TOOLHEAD EXTRUDER_ONLY=1

    {% elif filament_pos >= 5 %}
        {action_raise_error("Can't load - already in extruder!")}

    {% else %}
        {% if filament_pos <= 0 %}                      # FILAMENT_POS_UNLOADED
            _MMU_STEP_LOAD_ENCODER
        {% endif %}

        {% if filament_pos < 3 %}                       # FILAMENT_POS_END_BOWDEN
            _MMU_STEP_LOAD_BOWDEN LENGTH={length}
        {% endif %}

        {% if filament_pos < 4 and home_extruder %}     # FILAMENT_POS_HOMED_EXTRUDER
            _MMU_STEP_HOME_EXTRUDER
        {% endif %}

        {% if not skip_extruder %}                      # FILAMENT_POS_PAST_EXTRUDER
            _MMU_STEP_LOAD_TOOLHEAD
        {% endif %}

    {% endif %}

[gcode_macro _MMU_UNLOAD_SEQUENCE]
description: Called when MMU is asked to unload filament
gcode:
    {% set filament_pos = params.FILAMENT_POS|float %}
    {% set length = params.LENGTH|float %}
    {% set extruder_only = params.EXTRUDER_ONLY|int %}
    {% set park_pos = params.PARK_POS|float %}

    {% if extruder_only %}
        {% if filament_pos >= 5 %}                      # FILAMENT_POS_PAST_EXTRUDER
            _MMU_STEP_UNLOAD_TOOLHEAD EXTRUDER_ONLY=1 PARK_POS={park_pos}
        {% else %}
            {action_raise_error("Can't unload extruder - already unloaded!")}
        {% endif %}

    {% elif filament_pos == 0 %}
        {action_raise_error("Can't unload - already unloaded!")}

    {% else %}
        {% if filament_pos >= 5 %}                      # FILAMENT_POS_PAST_EXTRUDER
            # Exit extruder, fast unload of bowden, then slow unload encoder
            _MMU_STEP_UNLOAD_TOOLHEAD PARK_POS={park_pos}
        {% endif %}

        {% if filament_pos >= 3 %}                      # FILAMENT_POS_END_BOWDEN
            # Fast unload of bowden, then slow unload encoder
            _MMU_STEP_UNLOAD_BOWDEN FULL=1
            _MMU_STEP_UNLOAD_ENCODER

        {% elif filament_pos >= 1 %}                    # FILAMENT_POS_START_BOWDEN
            # Have to do slow unload because we don't know exactly where in the bowden we are
            _MMU_STEP_UNLOAD_ENCODER FULL=1
        {% endif %}

    {% endif %}
```

> [!NOTE]  
> Additional examples can be found at the end of `mmu_software.cfg`

<br>

## ![#f03c15](/doc/f03c15.png) ![#c5f015](/doc/c5f015.png) ![#1589F0](/doc/1589F0.png) Available Macro Reference

The following are internal macros that can be called from within the `_MMU_LOAD_SEQUENCE` and `MMU_UNLAOD_SEQUENCE` callbacks:

  | Macro | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Description&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Parameters |
  | ----- | ----------- | ---------- |
  | `_MMU_STEP_LOAD_ENCODER` | User composable loading step: Move filament from gate to start of bowden using encoder | |
  | `_MMU_STEP_LOAD_BOWDEN` | User composable loading step: Smart loading of bowden | `LENGTH=..` |
  | `_MMU_STEP_HOME_EXTRUDER` | User composable loading step: Extruder collision detection | |
  | `_MMU_STEP_LOAD_TOOLHEAD` | User composable loading step: Toolhead loading | `EXTRUDER_ONLY=[0\|1]` |
  | `_MMU_STEP_UNLOAD_TOOLHEAD` | User composable unloading step: Toolhead unloading | `EXTRUDER_ONLY=[0\|1]` `PARK_POS=..` |
  | `_MMU_STEP_UNLOAD_BOWDEN` | User composable unloading step: Smart unloading of bowden | `FULL=[0\|1]` `LENGTH=..` |
  | `_MMU_STEP_UNLOAD_ENCODER` | User composable unloading step: Move filament from start of bowden and park in the gate using encoder | `FULL=[0\|1]` |
  | `_MMU_STEP_SET_FILAMENT` | User composable loading step: Set filament position state | `STATE=[0..8]` `SILENT=[0\|1]` |
  | `_MMU_STEP_MOVE` | User composable loading step: Generic move | `MOVE=..[100]` Length of gear move in mm <br>`SPEED=..` (defaults to speed defined to type of motor/homing combination) Stepper move speed <br>`ACCEL=..` (defaults to min accel defined on steppers employed in move) Motor acceleration <br>`MOTOR=[gear\|extruder\|gear+extruder\|extruder+gear]` (default: gear) The motor or motor combination to employ. gear+extruder commands the gear stepper and links extruder to movement, extruder+gear commands the extruder stepper and links gear to movement |
  | `_MMU_STEP_HOMING_MOVE` | User composable loading step: Generic homing move | `MOVE=..[100]` Length of gear move in mm <br>`SPEED=..` (defaults to speed defined to type of motor/homing combination) Stepper move speed <br>`ACCEL=..` Motor accelaration (defaults to min accel defined on steppers employed in homing move) <br>`MOTOR=[gear\|extruder\|gear+extruder\|extruder+gear]` (default: gear) The motor or motor combination to employ. gear+extruder commands the gear stepper and links extruder to movement, extruder+gear commands the extruder stepper and links gear to movement. This is important for homing because the endstop must be on the commanded stepper <br>`ENDSTOP=..` Symbolic name of endstop to home to as defined in mmu_hardware.cfg. Must be defined on the primary stepper <br>`STOP_ON_ENDSTOP=[1\|-1]` (default 1) The direction of homing move. 1 is in the normal direction with endstop firing, -1 is in the reverse direction waiting for endstop to release. Note that virtual (touch) endstops can only be homed in a forward direction |

