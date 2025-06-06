# HBD1E
Kicad project for a configurable gate driver board to drive two MOSFETs with TO-247-4 package and
pinout [Drain, Source, KelvinSource, Gate].

Front side of board:

![Image front of PCB](media/pcb_image_front.png)

Back side of board:

![Image back of PCB](media/pcb_image_back.png)

# Features

- Supports 2 independent channels with 1 MOSFET per channel.
- No high-power connections on the board.  The user is free to connect the MOSFET power pins in any configuration they wish.
- Utilizes a gate driver chip in an SOIC-16 wide package, with a pin layout supported by multiple vendors.
- Each channel can be independently configured to be isolated from the gate driver control circuit.
- Each channel can be independently configured to use a linear regulator to precisely control the gate driver voltage.
- Most compatible gate driver chips support configurable dead time.

# Applications

- Half bridge, with input and output having a common ground: Configure the high side to be isolated, and the
low side non-isolated.
- T-Type inverter leg: Use 2 boards, with all gate power domains isolated, with the two middle MOSFETs configured
appropriately for common drain or common source.
- High power parallel MOSFETs: Disable dead time on the gate driver chip, and configure both MOSFET power
domains as non-isolated.

# Board Connections

Board input pins:  

![Board input pins](media/pcb_input.png)

- PWMH: PWM signal for the high-side MOSFET Q1, uses typical CMOS switching levels (off <= 0.8V, on >= 2.0V).
- PWML: PWM signal for the low-side MOSFET Q2, uses typical CMOS switching levels.
- DISABL: Pull this value low to enable the driver chip.  There is a pull up resistor to VCC on this circuit,
so the control component must pull this signal low to enable switching.  Uses typical CMOS switching levels.
- GND: Ground common to VCC and VGD.
- VCC: Power to the control side of the gate driver chip.  Typically 3.3V or 5V, maximum 20V.
- VGD: Power to the gate driver regions, optionally through isolation modules.

For the reference gate driver chip UCC21520, VCC and VGD may be the same for applications with lower
gate voltages.  Or you may choose to have them be separate, and use a higher voltage VGD to gain headroom
through the isolated modules, and configure the linear regulators to provide a precise post-isolation
gate drive voltage.

Note that there are no on-board connections for the high-current drain and source pins for either
MOSFET Q1 or Q2.  These must be connected separately, either through appropriately-sized wires or
a properly-designed laminated bus.

# Schematic

![Schematic drawing](media/schematic.png)

# PCB Layout Details

PCB components:

![PCB components](media/pcb_components.png)

Front copper layer:

![PCB front copper layer](media/pcb_copper_front.png)

Back copper layer:

![PCB back copper layer](media/pcb_copper_back.png)

# Configuring Channel Isolation

The front of the board has marked locations for two DC-DC isolated modules, U3 (high side, shown below) and U5 (low side).

![Isolated module marking](media/pcb_isolation.png)  

These modules take power from the VGD pin, and supply it to the regions of the PCB driving the gates for
the high side MOSFET Q1 and the low side MOSFET Q2.

To make either region isolated, solder a DC-DC isolated converter module in the marked location.
Our test boards used a Gaptec 3S7B-2424S3UP, but pin-compatible 7-SIP components are available from multiple manufacturers.

To make one of these power domains non-isolated, solder a wire connection from + to + and another from - to -.
Ordinary AWG 20 gauge hookup wire is adequate.

# Configuring Gate Drive Voltage Regulation

On the back side of the board, under the isolated modules U3 and U5, are locations for LM317 linear regulators
and associated components.

![Gate drive voltage regulation](media/pcb_regulation.png)  

- High Side: (under U3) U4, D3, D4, C10, C11, R14, R15, R16
- Low Side: (under U5) U2, D1, D2, C7, C9, R11, R12, R13

The following table shows which resistors on this board map to those in a typical LM317 datasheet.
This should allow you to select resistor values to configure your desired gate driver voltage.

| Datasheet | High Side | Low Side |
| --------- | --------- | -------- |
|    R1     |    R16    |   R11    |
|    R2     |  R14+R15  | R12+R13  |

Note that we used 2 resistors in series for the R2 resistor from the datasheet.
The power dissipation from the minimum regulation current may exceed the limit for a typical
1/10 watt resistor, so we use 2 resistors in series to spread the thermal load.

Here is a table showing combinations for some typical gate drive voltages, using common resistor values.
Of course, more precise voltages can be obtained using more specific resistor values and higher
tolerance 0.1% resistors.

| Target Voltage | R16/R11 | R14+R15/R12+R13 | Computed Voltage |
| -------------- | ------- | --------------- | ---------------- |
|       10       |   180   |     270/1000    |      10.069      |
|       12       |   180   |     560/1000    |      12.083      |
|       15       |   180   |    1000/1000    |      15.139      |
|       18       |   180   |    1200/1200    |      17.917      |
|       20       |   180   |    1200/1500    |      20.000      |

Note also that the LM317 is a very robust device that may not require all the configured components.
The adjust capacitors may be omitted if your testing shows that ripple and other voltage transients
do not cause any downstream issues when switching the MOSFETs.
Each of the adjust capacitors is paired with a diode:

- C9, D2
- C11, D4

If you omit the adjust capacitor, you can also omit the matching diode.  If you use the adjust capacitor,
you should also use the diode to prevent the capacitor from discharging through the regulator in a power off
scenario.

Also note that you should always populate D1 and D3, since there are large capacitors for the gate driver
chip that may discharge through the regulator in a power off scenario.  D1 and D3 will prevent this.

To configure either the high-side or low-side power domain for voltage regulation, populate the associated
components as listed above.

To configure either of the power domains without voltage regulation, omit all of the regulation components
listed above for that domain, and replace the associated diode (D1 or D3) with a 0 ohm shunt resistor.

# Configuring Dead Time

![Dead time configuration](media/pcb_dead_time.png)  

The components used to configure dead time are R1, R10 and C6.

Some, but not all of the pin-compatible gate driver chips support controlling dead time between Q1 and Q2
gate transitions, using a resistor connected to pin 6.  If the gate driver chip you use does not support dead
time generation, then do not populate R1, R10 and C6, and ignore the rest of this section.

If your gate driver chip supports dead time insertion, populate R10 and optionally C6 according to the
datasheet for your chip.  Do not populate R1.

If your chip supports dead time insertion, but you wish to suppress it, populate R1, but not R10 and C6.
This may be desireable if you use this board in a high-current parallel MOSFET application, where you want
both MOSFETs on at the same time.

If your control logic providing the PWM signals also implements dead time, one strategy is to program an
absolute minimum dead time via the hardware as described in this section, and program an optimized, longer
dead time in your control logic.  The gate driver chip uses the longer of the hardware configured dead
time or the software-controlled dead time in the PWM inputs.

# Configuring Input Signal Filtering

![Input signal filter configuration](media/pcb_input_filters.png)

Each of the 3 input signals from the controller to this gate driver board has an RC circuit to
filter transients that may cause issues for the gate driver IC input logic.  These signals and the
associated filter components are listed below.

| Signal Name | Filter Resistor | Value | Capacitor | Value | Pull Up Resistor | Value |
| ----------- | --------------- | ----- | --------- | ----- | ---------------- | ----- |
|     PWMH    |       R4        |  47Ω   |     C1    |  33pF |                  |       |
|     PWML    |       R5        |  47Ω   |     C4    |  33pF |                  |       |
|    DISABL   |       R6        |  47Ω   |     C5    |   1nF |        R7        |  10k  |

The size of all these components is 0805, making them less trouble to hand solder while changing
filter components.

# Test Points

![Test points](media/pcb_test_point.png)

There are numerous test points located on both sides of the board to which users can solder
surface mount test point loops, for example KOA Speer RCWCTE.
This gives you the ability to clip oscilloscope leads to enable monitoring various signals while
debugging the gate driver configuration.

### Caution!  This board may use high voltages that are hazardous to humans.
- Be very careful when connecting test leads to potentially high voltage points on the circuit board.
- High voltages may also break test equipment if proper grounding and isolation practices are not followed.
- Please be careful!

List of test points on the board:
| Label |  Name    | Location | Description                                     |
| ----- | ------   | -------- | ----------------------------------------------- |
| TP1   | HS Gate  | Bottom   | High side MOSFET gate pin                       |
| TP2   | LS Gate  | Bottom   | Low side MOSFET gate pin                        |
| TP3   | VDDB     | Bottom   | Low side post-regulated gate driver voltage     |
| TP4   | VDDA     | Top      | High side post-regulated gate driver voltage    |
| TP5   | VDDA_ISO | Bottom   | High side post-isolation, pre-regulated voltage |
| TP6   | PWMHF    | Top      | High side post-filter PWM input signal          |
| TP7   | PWMLF    | Top      | Low side post-filter PWM input signal           |
| TP8   | DISF     | Top      | Disable post-filter input signal                |
| TP9   | VCC      | Top      | Logic side power for gate driver chip           |
| TP10  | VGD      | Bottom   | Pre-isolation gate driver voltage               |
| TP11  | VDDB_ISO | Bottom   | Low side post-isolation, pre-regulated voltage  |

# Reference Components

- Gate driver chip: Texas Instruments UCC21520
- MOSFET: Infineon AIMZH120R020M1TXKSA1
- Isolated DC-DC module: Gaptec 3S7B-2424S3UP
- Linear Regulator: Texas Instruments LM317

# Example Bill of Materials (BOM)

An example BOM is located in the file HBD1E_BOM.csv.
