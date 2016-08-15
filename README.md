# PSVCD Introduction.

PSVCD stands for Playstation Vita Cartridge Dump. 
Basically this project summarizes my half year research on the ways to do a hardware dump of PS Vita cart.
I think I should indicate that this research has no relation to Cobra Black Fin project and team.

Currently the process of dumping PS Vita carts is quite involved because you have to create custom board.
On the other hand - this approach does not have any firmware dependency since it is not related to software hacking.
Current version of the board is constructed from DIP elements but I think in the future I will create PCB layout and smaller board.

So the good news are that carts actually can be dumped. You should be able to do it if you complete all the steps of this readme.
At current point I do not know if any of these dumps can be played on multiple different instances of PS Vita.
Most likely it is not possible currently. Process of writing content to PS Vita cart is also not established yet, though I think it can be done.
Also most of the cart content is still encrypted. I do not know any ways to decrypt it now.
Though I am sure that people who work with PSP and PS3 can do it because many files look similar to these consoles.

Last final note. This readme should not be considered as full or complete at current point.
I hope I will be adding more and more details on the future.

# Previous work

I know that some research was done before me. Unfortunately clear results were not published. 
Previous works include:
- Dump of "Monster Hunter" and "Wipe Out" made my Katsu
- Cobra Black Fin dongle created by corresponding team

There were also some software dumps made by Mr.Gas and game modding/decryption made by Major_Tom.

# Motherboard. Game cart slot schematics.

There can be multiple ways to access game cart data lines on motherboard.
- You can desolder game cart slot and solder to internal pins.
- You can solder to test pads, located on motherboard. 

First of all you will need to disassemble PS Vita body and take motherboard out.
On the opposite side you will see game cart slot. 

Consider looking at pics/pic4.png
This is overall schematic for game cart slot and surroundings with all markings that you will need.

If you want to use first approach - consider looking at pics/pic1.png
Test pads are marked as TP*
Capacitors are marked as C* (though I am not sure they are really capacitors, they can be other smd parts)
Resistors are marked as R* (though I am not sure they are really resistors, they can be other smd parts)
Unknown parts are marked as UNK*

Some guesses on schematics:
C1, C2, C3 - these are most likely supply filter capacitors
C4, R2 - these can be RC circuit for generating game cart insert impulse

If you want to use second approach - consider looking at pics/pic2.png
Marking is similar.

Some info on schematics:
R3, R4, R5, R6, R7 - these are pull-up resistors for DATA and CMD lines.

I have chosen first approach with desoldering game cart slot since at that point I did not know all schematics.
You may see at pics/pic3.png that motherboard also has LED1 and place for one more led that is not soldered.
I have soldered 1x10 2mm female header to the pins because it fits the hole for game cart in PS Vita body.

Finally you will have to create 1x10 2mm male -> 1x10 2.54mm male adapter.
This will be required for further usage of any prototype board.
This adapter can be easily made with some pin headers.
Consider looking at pics/pic5.png to see how it should look like.

# Game cart. Schematics and plugging into prototype board.

I admit that this is not the best approach, so feel free to advice.
It is not good because you have to disassemble game cart.
The best one would be to print game cart slot on 3d printer but I do not own one.
Unfortunately there is no easy way to desolder game cart slot from PS Vita motherboard.
You will definitely need desoldering gun but this is not enough. 
Back side of cart slot is made from plastic and is glued to the motherboard.
Even if you will manage to desolder game cart slot - construction will not be stable.

So I have created simple adapter that can be used with prorotype board.
You will need:
- Game cart
- Memory Stick Duo adapter
- Memory Stick Duo card slot - I desoldered one from old card reader.
- 1x10 2.54mm pin header 

Steps are the following:
- Disassemble Memory Stick Duo adapter
- Solder pin header to adapter
- Disassemble game cart - these come as monolithic chip with memory and controller on board.
- Put game chip under pin headers of adapter then close the adapter.

Consider looking at pics/pic6.png to see how it should look like.

# Configuring FTDI FT232h.

At this point you should be able to connect PS Vita and game cart to any prototype board.
Next step would be creating custom board that will allow to dump game card.
But before that we will need to configure FTDI chip that serves as heart of the board.

For custom board you will need FT232h chip. 
I guess FT2232h can also be used - it just has more pins and two channels instead of one.
FT232h can come on a breakboard - Adafruit FT232H, FTDI UM232H etc. You can use custom board as well. 

For configuring FT232h we will need FT_Prog utility that can be downloaded from FTDI web page.
Settings for FT232h are as following:

- USB Config Descriptor -> Self Powered: Enabled.
  We will use self powered configuration because all parts of the custom board including FT232h will be
  powered from one voltage regulator. I found out that some FT232h chips do not give 3.3 volts 
  on corresponding 3v3 pins when USB power mode is selected. We MUST use 3.3 volts or game cart can be damaged.
  
- USB String Descriptors -> Product Desc: USB FIFO.
  This is an identifier for the device that will be later used in C++ code. It will come in handy if you use
  other FTDI devices that are also plugged into your PC. 
  For example I also used Lattice MachXO2 FPGA for some investigations. And it has FT2232h chip on board.

- Hardware Specific -> Suspend on ACBus7 Low: Enabled.
  This setting is required if FT232h is used in self powered mode. ACBus7 line must be tied to 5V through resistor.
  It will be explained later.

- Port A -> Driver -> D2XX Direct: Enabled.
  We will use fast direct drivers and not virtual com port.

- Port A -> Hardware -> 245 FIFO: Enabled.
  This setting is required when Sync FIFO mode or MPSSE mode is used.

## Troubleshooting.

Problem 1.

There can be cases when you will not be able to program FT232h chip. In most cases you will see this error:
"Index was outside the bounds of the array".
Unfortunately this is a bug in FTDI software that is mixed together with flaw in your breakboard.
When you observe this error take a closer look at your EEPROM. Most likely you will have 93C46 type.
It is indicated in FTDI datasheets that 93C46 can be used with FT232h but unfortunately it can not.

There are 2 solutions for this problem.
- FT_Prog is written in C#. You can decompile it with Reflector and fix the code a bit so that FT232h can be programmed even with smaller 93C46.
  I have this fix so maybe I will share it in the future.
- Desolder 93C46 EEPROM and solder 93C56 EEPROM or bigger one.

Problem 2.

Some FT232h chips do not give 3.3v on corresponding pins when used in USB powered mode. 
They may give 3.6v or 3.8v etc. To fix this you have to reconfigure FT232h chip to work in self powered mode.
This may require some hadrware changes as well.

For example on my breakboard I had to desolder Zero Ohm resistor that connected USB pin and VREGIN pin.
Then I have soldered another 39K Ohm resistor that was connected to AC7 and to USB pin.
Please consider looking into FTDI datasheets to understand what is required for self powered configuration.

It looks like that famous Adafruit breakboard also has Zero Ohm resistor.
FTDI UM232H is more clever - you can use jumpers without desoldering anything.

# Building custom prototype board.

Heart of all system is a custom board that allows to interaction between PC and PS Vita game cart.
Consider looking at schematic pics/pic7.png for further details.
Custom board consists of multiple sections that are described below.

## Voltage regulator section

When powered from USB we have 5 volts. Any SD cards or MMC cards work from 3.3 volts or even lower 1.8 volts.
Using 5 volts will damage the card. 
Voltage regulation section is located in top left corner of schematic file.

Required parts are:
- DS1099-B USB port type B. You can use Type A if you prefer. 
- Two leds: these are optional and are used just to show that power is on.
- R21, R22: 220 Ohm resistors for leds.
- LD1117V33: Voltage regulator that will transform USB 5V to 3.3V
- C1: Filtering capacitor 100 uF.
- C5, C6, C7: Filtering capacitors 1uF.
- S1: Dip Switch 1 pin: this is optional. Used to switch custom board power on and off.
- SV3, SV4: Two 1x10 2.54mm female headers. These are used to wire any other places of custom board to VCC3V3 or GND.

## Data lines section

This section contains multiple female headers that can be used for wiring different devices.
There are also other components that can be used to configure each individual line.
Section can be found in the middle left part of schematic file.

Required parts are:
- Two 2x10 2.54 male pin headers. These are used in conjunction with jumpers for each data line.
- Two 2x10 2.54 female headers. These are used to connect any external devices (game cart, logic analyzer etc) or to do internal wiring.
- JP1, JP2, JP3, JP4, JP5, JP6, JP7, JP8, JP9, JP10. These are jumpers that can be used to pull
  each individual data line to VCC3V3.
- R1, R2, R3, R4, R5, R6, R7, R8, R9, R10. These are 4.7K Ohm pull-up resistors.
- JP11, JP12, JP13, JP14, JP15, JP16, JP17, JP18, JP19, JP20. These are jumpers that can be used
  to pull each individual data line to GND.
- R11, R12, R13, R14, R15, R16, R17, R18, R19, R20. These are 4.7K Oh, pull-down resistors.

## Data multiplexing section

This section is used to select 1 of 8 data lines and feed output to FT232h chip.
Section can be found in the bottom left part of schematic file.
Data transfer between game cart and PS Vita is bidirectional. 
That means that we may use tristate buffers as well. Though buffers are optional.
Output of multiplexer can be controlled with G (enable) signal.

Required parts are:
- 74HC244N: Octal tristate buffer.
- 74HC151N: 8-dine to 1-dine data multiplexer.

Main idea is to connect data lines D0-D7 to 74HC244N. Output of 74HC244N should be connected to 74HC151N.
74HC151N will allow to select one of 8 data lines. Others will be tristate.
Address lines of 74HC151N are connected to GPIO pins of FT232h.
Enable pins of 74HC244N should be connected to GPIO pin of FT232h.

This will allow to select and read 1 of 8 data lines while others will be tristate. 
Address of line and read/write mode will be controlled by FT232h.

## Data demultiplexing section

This section is used to select 1 of 8 lines and use it as output to game cart.
Section can be found in the bottom right part of schematic file.

Required parts are:
- Two 74HC125N: Quadruple bus buffer gates with tristate outputs.
- 74HCT138N: 3 to 8 line decoder, inverting

Main idea is to connect data lines D0-D7 to output pins of 74HC125N tristate buffers.
Outputs of 74HCT138N decoder should be connected to enable pins of 74HC125N.
Input pins of 74HC125N should all be connected together to GPIO pin of FT232h
Address lines of 74HCT138N decoder should be connected to GPIO pins of FT232h.
Enable pin of 74HCT138N should be connected to GPIO pin of FT232h.

This will allow to select and write to 1 of 8 data lines while others will be tristate. 
Address of line and read/write mode will be controlled by FT232h.

## Game cart initialisation bypass section

PS Vita game cart requires special initialization sequence before game cart can be read.
This sequence can not be reproduced at current point. Though it is already partially known.
To bypass this initialization a simple trick can be used:
- Allow real PS Vita to initialize game cart. 
- Then connect custom board. 
- Then disconnect PS Vita.
- Now you can read game cart.

Make sure that during these steps game cart is always powered.

Required parts are:
- SV5, SV6, SV7: Three 1x10 2.54mm female headers.
- S2, S3: Two 1x10 Dip switches.

Main idea is to connect PS Vita data lines to SV6 and custom board data lines to SV5.
On the other hand game cart will be connected to SV7.

## Custom board core section

Core of the custom board is FT232h chip.
There are some other components required so that Ft232h runs as expected.

Required parts are:
- R23, R24, R25, R27, R28, R29, R30, R31, R32: 4.7K Ohm pull-up/pull-down resistors.
- Four 1x2 2.54mm male pin headers. 
- JP21, JP22, JP23, JP24: jumpers for selecting pull-down or pull-up configuration.
- R26: 39K Ohm pull-up resistor.

Main idea is to connect address lines of 74HC151N to GPIOH pins AC0-AC2 of FT232h.
Address lines of 74HCT138N decoder should be connected to GPIOH pines AC3-AC5 of FT232h.
GPIOH pin AC6 serves as read/write mode pin and should be connected to 74HC244N buffer and 74HCT138N decoder.
Single pin can be used because 74HC244N and 74HCT138N use different levels of enable signal.
74HCT138N uses high and 74HC244N uses low.
All address lines and R/W line can be pulled together either to GND or to VCC by using jumpers JP23 or JP24. 
Due to properties of my prototype board I was not able to solder separate jumper for each line. But you can do it.

GPIOH AC7 line should be connected to USB 5V pin of FT232h through 39K Ohm pull-up resistor.
GPIOH AC8, AC9 pins can not be used in MPSSE mode of FT232h so they are not connected.
MPSSE mode will be explained later.
VREGIN pin of FT232h should be connected to 3.3 volt output of voltage regulator.

3V3, VCCIO pins should be connected to 3.3 volt and two GND pins should be connected to ground.

GPIOL AD0 pin will serve as CLK in MPSSE mode.
GPIOL AD2 pin will serve as DIN in MPSSE mode so it is connected to 74HC151N output.
GPIOL AD5 pin will serve as DWAIT in MPSSE mode so it is connected to 74HC151N output also.
GPIOL AD1 pin will serve as DOUT in MPSSE mode so it is connected to input of 74HC125N buffers.
GPIOL AD3 pin can only serve as CS in MPSSE mode so it is not used and not connected.
GPIOL AD4, AD6, AD7 pins are not used so they are not connected.

DOUT, DIN/DWAIT lines can be pulled together either to GND or to VCC by using jumpers JP21 or JP22.











