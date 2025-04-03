# HBD1E
Kicad project for a flexible gate driver board to drive two MOSFETs with TO-247-4 package and
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
- Most gate driver chips support configurable dead time.

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

- PWMH: PWM signal for the high-side MOSFET Q1
- PWML: PWM signal for the low-side MOSFET Q2
- DISABL: Pull this value low to enable the driver chip.  There is a pull up resistor on this circuit,
so the control component must pull this signal low to enable switching.
- GND: Ground common to VCC and VGD
- VCC: Power to the control side of the gate driver chip.
- VGD: Power to the gate driver regions, optionally through isolation modules.

For the reference gate driver chip UCC21520, VCC and VGD may be the same for applications with lower
gate voltages.  Or, you may choose to use a higher voltage VGD to gain headroom through the isolated
modules, and regulate it post-isolation to a precise gate drive voltage.

Note that there are no on-board connections for the high-current drain and source pins for either
MOSFET.  These must be connected separately, either through an appropriately-designed wire or
laminated bus.

# Schematic

![Schematic drawing](media/schematic.png)

# PCB Details

PCB components:
![PCB components](media/pcb_components.png)

Front copper layer:
![PCB front copper layer](media/pcb_copper_front.png)

Back copper layer:
![PCB back copper layer](media/pcb_copper_back.png)

# Configuring Channel Isolation

The front of the board has marked locations for two DC-DC isolated modules, U3 (high side) and U5 (low side).

![Isolated module marking](media/pcb_isolation.png)

These modules take power from the Vgd pin, and supply it to the regions of the PCB driving the gates for
the high side MOSFET Q1 and the low side MOSFET Q2.

To make either region isolated, solder a DC-DC isolated converter module in the marked location.
Our test boards used a Gaptec 3S7B-2424S3UP, but pin-compatible 7-SIP components are available from multiple manufacturers.

To make one of these power domains non-isolated, solder a wire connection from + to + and another from - to -.
Ordinary AWG 20 gauge hookup wire is adequate.

# Configuring Gate Drive Voltage Regulation

On the back side of the board, under the isolated modules U3 and U5, are locations for LM317 linear regulators
and associated components.

![Gate drive voltage regulation](media/pcb_isolation.png)

- High Side: (under U3) U4, D3, D4, C10, C11, R14, R15, R16
- Low Side: (under U5) U2, D1, D2, C7, C9, R11, R12, R13

Note that we used 2 resistors in series for the connection from the LM317 adjust pin to VSSx.
The power dissipation from the minimum regulation current may exceed the limit for a typical
1/10 watt resistor, so we use 2 resistors in series to spread the thermal load.

Note also that the LM317 is a very robust device that may not require all the configured components.
The adjust capacitors may be omitted if your testing shows that ripple and other transients do not cause any
downstream issues when switching the MOSFETs.  Each of the adjust capacitors is paired with a diode:

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

# Reference Components

- Gate driver chip: Texas Instruments UCC21520
- MOSFET: Infineon AIMZH120R020M1TXKSA1
- Isolated DC-DC module: Gaptec 3S7B-2424S3UP
- Linear Regulator: Texas Instruments LM317

