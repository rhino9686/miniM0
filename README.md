# My first M0 Design: Mini M0
### Motivations

MSPM0 microcontrollers are an important new device from TI.

Being brand new, there's not a lot of public designs done that aren't hefty launchpads or niche additions to large reference designs. 

<img src="img/M0Launchpad.png_large" alt="drawing" width="600"/>

I was motivated to do a project to make myself a tiny litle M0 dev board similar to the [STM Nucleo boards.](https://www.st.com/content/ccc/resource/technical/document/data_brief/b1/d8/13/d4/b0/b7/4b/6e/DM00214578.pdf/files/DM00214578.pdf/jcr:content/translations/en.DM00214578.pdf)


<img src="img/NUCLEO-F303K8.jpg" alt="drawing" width="200"/>


Instead of making a large bulky M0 launchpad, I made a very minimalist board with basic GPIO pins and a programming header. In this document I will show how to recreate my work step by step for anyone making their first MSPM0 design.


There are quite a few MSPM0 models to choose from. I used the MSPM0G3507 as our MCU as it is smack dab in the middle of the catalog in terms of price and performance. The lessons from this design will scale up to more complex devices and down to smaller cost-sensitive ones.
### How to start

This can be done with schematic/PCB software of your choosing. I chose to go with KiCAD in this. The first step is importing the MSPM0G3507 schematic symbol and footprint into a blank schematic. I in particular used the *MSPM0G3507SRHBR* package.

<img src="img/blankMCU.png" alt="drawing" width="400"/>

### Part Selections

You will see for an equivalent STM32 processor from STMicro, there is a requirement for heavy input capacitance on the VDD and AVDD pins. MSPM0 has a lesser requirement. One simply needs a small 1-3 capacitors equaling roughly 10 uF, with the option to add more based on your application. I stuck with the basic 10u.
<img src="img/stm32Cap.png" alt="drawing" width="400"/>

The MSP takes in a VIN range of 1.62-3.6V, so one can easily make a USB port-powered input and step down the voltage to 3.3V.

My design used the basic USB-C (non-PD) port that negotiates a 5V, 3A rail with any common USB-C cord. 
<img src="img/simpleUSB.png" alt="drawing" width="400"/>

In any full design, one will want ESD protection for this port, since it connects to an external source of static. I included a basic ESD diote for this, the TPD6E004.

<img src="img/esdDiode.png" alt="drawing" width="400"/>

Finally, I have an LDO that takes the voltage down from 5V to 3.3V for the MSPM0. I used the TPS7A20 for this.
<img src="img/LDO.png" alt="drawing" width="400"/>


Once all the main components are gathered on the same schematic as this, one can proceed to fill out all the proper connections and passives.
<img src="img/full_schem_unrouted.png" alt="drawing" width="400"/>

### First Connections

For the USB-C port, I recreated the connections found at this link: https://forum.digikey.com/t/simple-way-to-use-usb-type-c-to-get-5v-at-up-to-3a-15w/7016


<img src="img/usbMapping.png" alt="drawing" width="400"/>

<img src="img/myUSBC.png" alt="drawing" width="400"/>

I also wired the 5V line to the ESD diode to protect against voltage spikes from connecting the USB cable.

<img src="img/finishedESD.png" alt="drawing" width="400"/>

Note: If you were using the D+ and D- lines of the USB port, you'd connect them to these IOx pins, but we left them floating here. The CC1 and CC2 lines can alos be protected optionally, but that's out of the scope of this design.

Finally, we connect the 5V line to a 5V-3V3 LDO to provide a suitable voltage for our M0 MCU.

<img src="img/LDO_popped.png" alt="drawing" width="400"/>

That's the basics. With these connectiond, our MCU is good to power up. It won't do too much until we add some way of connecting GPIO, activating peripherals, and adding a programming header.

<img src="img/processorSchem.png" alt="drawing" width="400"/>

To make things easier organizationally, I like to add labels to every MCU pin, then reference label in other parts of schematic.

### Communication

To activate our communication lines, we will need to assign pins for I2C and UART

To do this, I will open up my selected device in Code Composer Studio. Once again, I am using MSPM0G3507.

To download CCS, you can find it [here](https://www.st.com/content/ccc/resource/technical/document/data_brief/b1/d8/13/d4/b0/b7/4b/6e/DM00214578.pdf/files/DM00214578.pdf/jcr:content/translations/en.DM00214578.pdf)


There's also a comprehensive walkthrough of CCS Theia and a MSPM0 board here which may be useful reading. I will go through a similar walkthrough for a blank project.

**Page 1: where you will input "MSPM0G3507" and click "Create a new project".**

<img src="img/CCS_p1.png" alt="drawing" width="600"/>

**Page 2: select "empty_driverlib.src".**
**You can name your project whatever you want, and pick the CCS_TI Arm Clang Compiler.**

<img src="img/CCS_p2.png" alt="drawing" width="600"/>

**You'll now have this README page and some tabs on the left.**
**The one you want to select is the "empty.syscfg".**

<img src="img/CCS_p3.png" alt="drawing" width="600"/>

**This is where we se the configuration GUI, where we can activate all sorts of peripherals and generate code for drivers.**

<img src="img/CCS_p3.5.png" alt="drawing" width="600"/>

**We will need to make one small adjustment, as we need to switch the IDE ofer to the RHBR package with 32 pins.**
<img src="img/CCS_p4.png" alt="drawing" width="600"/>

**Hit "switch" and then change the "LQFP-64" option to "VQFN-32"**

<img src="img/CCS_p5.png" alt="drawing" width="600"/>

**Now we have the correct pins that correspond to our schematic symbol**

<img src="img/CCS_p6.png" alt="drawing" width="600"/>

**Now, to start making our modifications**
**I will activate I2C by pressing the (+) icone next to them**

<img src="img/CCS_p7.png" alt="drawing" width="600"/>

**This will automatically assign the I2C to pins 1(PA0) and 2(PA1). [SDA->PA0 and SCL->PA1]**
**You can change the pins in the PinMux menu at any time to your preference**

<img src="img/CCS_p8.png" alt="drawing" width="600"/>

**I will also activate the UART pins and accept their default mapping to pins 14(PA10) and 15(PA11). [TX->PA10 and RX->PA11]**

<img src="img/CCS_p9.png" alt="drawing" width="600"/>

**Finally, I will add a couple GPIO outputs for us to seee the output behavior of simple programs**
**For me, these assigned to pins 6(PA2) and 7(PA3)**
<img src="img/CCS_p10.png" alt="drawing" width="600"/>

**FThe end result is that your device picture now has a graphical map for pin assignments, a handy tool for your schematic pin assignments Hovering over any of the pins shows its function**

<img src="img/CCS_p11.png" alt="drawing" width="600"/>

<img src="img/CCS_p12.png" alt="drawing" width="600"/>

**We also see that, for the purpose of programming, Pin 24(PA20) is by default reserved for SWCLK, Pin 23(PA19) for SWDIO.**
**We'll return to this when it comes time to make the programming header**

<img src="img/CCS_p13.png" alt="drawing" width="600"/>

<img src="img/CCS_p14.png" alt="drawing" width="600"/>





<img src="img/bigHeaders.png" alt="drawing" width="600"/>


<img src="img/clockingsignal.png" alt="drawing" width="600"/>

<img src="img/complicatedUsb.png" alt="drawing" width="600"/>

<img src="img/ConnectorsHeaders.png" alt="drawing" width="600"/>




<img src="img/mspGuide.png" alt="drawing" width="600"/>



