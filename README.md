# My first M0 Design: Mini M0
### Motivations

MSPM0 micros are an important new device from TI

Being brand new, there's not a lot of public designs done that aren't hefty launchpads or niche additions to large reference designs. 

I was motivated to do a project to make myself a tiny litle M0 dev board similar to the [STM Nucleo boards.](https://www.st.com/content/ccc/resource/technical/document/data_brief/b1/d8/13/d4/b0/b7/4b/6e/DM00214578.pdf/files/DM00214578.pdf/jcr:content/translations/en.DM00214578.pdf)


<img src="img/M0Launchpad.png_large" alt="drawing" width="600"/>

Instead of making large bulky M0 launchpad, I'll make a very minimalist board with basic GPIO pins and a programming header.


There are quite a few MSPM0 models to choose from. We will use the MSPM0G3507 as our MCU as it is smack dab in the middle of the catalog in terms of price and performance. The lessons from this design will scale up to more complex devices and down to smaller cost-sensitive ones.
### How to start

This can be done with schematic/PCB software of your choosing. I chose to go with KiCAD in this. The first step is importing the MSPM0G3507 schematic symbol and footprint into a blank schematic. I in particular used the *MSPM0G3507SRHBR* package

<img src="img/blankMCU.png" alt="drawing" width="600"/>

### Part Selections

You will see for an equivalent STM32 processor from STMicro, there is a requirement for heavy input capacitance on the VDD and AVDD pins. MSPM0 has a lesser requirement. You will simply need a small 1-3 capacitors equaling roughly 10 uF, with the option to add more based on your application. I stuck with the basic 10u.
<img src="img/stm32Cap.png" alt="drawing" width="600"/>

The MSP takes in a VIN range of 1.62-3.6V, so one can easily make a USB port-powered input and step down the voltage to 3.3V.

My design used the basic USB-C (non-PD) port that negotiates a 5V, 3A rail with any common USB-C cord. In any full design, one will want ESD protection for this port, since it connects to an external source of static. I included a basic ESD diote for this, the TPD6E004.
<img src="img/simpleUSB.png" alt="drawing" width="600"/>
<img src="img/esdDiode.png" alt="drawing" width="600"/>

Finally, I have n LDO that takes the voltage down from 5V to 3.3V for the MSPM0. I used the TPS7A20 for this.
<img src="img/LDO.png" alt="drawing" width="600"/>


Once all the main components are gathered on the same schematic as this, one can proceed to fill out all the proper connections and passives.

<img src="img/bigHeaders.png" alt="drawing" width="600"/>


<img src="img/clockingsignal.png" alt="drawing" width="600"/>

<img src="img/complicatedUsb.png" alt="drawing" width="600"/>

<img src="img/ConnectorsHeaders.png" alt="drawing" width="600"/>



<img src="img/esdDiode.png" alt="drawing" width="600"/>

<img src="img/mspGuide.png" alt="drawing" width="600"/>

<img src="img/myUSBC.png" alt="drawing" width="600"/>

<img src="img/usbMapping.png" alt="drawing" width="600"/>