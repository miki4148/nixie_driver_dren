# Modular nixie tube clock system
![Project overview](files/project_overview.png)

A versatile and customizable driver for Nixie tube clocks. This project focuses on developing hardware and firmware to drive Nixie tubes, allowing you to create various retro-style displays with ease.

---

## Features

- **Modular design**: Interchangeable pieces with efficient management of high voltages required by nixie tubes.
- **Customizable Firmware**: Adjust timing  of various displayed things at will.
- **Support for Various Tubes**: Compatible with IN-12A, IN-12B and more.
- **Easy Assembly**: Simple PCB design with detailed instructions.
- **Real-Time Clock Integration**: In the future, RTC modules may be integrated into the project.

---

## Hardware

### Components:
1. Nixie Tubes (e.g. IN-12A or IN-12B)
2. Regulated Power Supply (12V DC output)
3. Microcontroller (e.g. STM32 family)
4. Maybe RTC Module? (e.g. DS3231)
5. Darlington Arrays (e.g. SN75468)
6. Demultiplexers (e.g. CD4028B)
7. Boost controller (UCC38XX family)
8. Resistors, Capacitors, Connectors etc.
9. PCB (Custom-designed for this project)


## Driver: 
The circuit consists of a boost converter which steps up the supply voltage to 170VDC. Such voltage is sufficient for ionization of the low-pressure gas mixture inside the tube at room temperature (without the need of white-hot cathode). 

After long battle against procrastination, layout for the driver board emerged. Detailed description and schematic will follow in subsequent sections.
![Driver front](/files/NixieDriverEvenSmallerNoDot/img/NixieDriverEvenSmallerNoDot_FRONT.png)
Insulation Displacement Cable connector was chosen as the main medium of communication w/ the micro. This was a direct result of author's recent traumatic experiances and many hours of fruitless work trying to wire previous device using old solid-copper wires which were nearly unsolderable and brittle beyond comprehension. So ribbon cable it is. 
Alternative and secondary I/Os were banished to ordinary pin headers for simplicity.  

The other half of the circuit is basically massive demultiplexing and buffering array. The good news is that not every cathode needs to be operated idependently but rather only one (two maybe) digit per tube lights up at one time. This points to BCD (binary-coded decimal) converters (4028 IC used) and a simple buffering for commas as they may have to be used independently. 

![Driver back](/files/NixieDriverEvenSmallerNoDot/img/NixieDriverEvenSmallerNoDot_BACK.png)
Note author's love for hardcore poetry insertion.


## Dual tube adapter
Someone must hold the tubes in place so here it is:
![Tube holder front](/files/NixieDriverTwoTubes/img/NixieDriverTwoTubes_FRONT.png)

Holes in pcb in the middle of the tubes' footprints are an answer to the glass short feature at the base, a remainder after sealing process. 
Other holes increase versatility in case someone wished to insert leds in them to achieve obligatory flashing in the space between glowing numerals. 

![Tube holder back](/files/NixieDriverTwoTubes/img/NixieDriverTwoTubes_BACK.png)
Note the attempt to match writing on the pcb to native Muttersprache of the late glassblowers responsible for birthing of IN-12s.


---
<!-- 
## Software Setup

### Prerequisites
- TBA


## Usage
Assemble the hardware as per the provided schematics.
Power the circuit using the recommended voltage specifications.

At vero eos et accusamus et iusto odio dignissimos ducimus qui blanditiis praesentium voluptatum deleniti atque corrupti quos dolores et quas molestias excepturi sint occaecati cupiditate non provident, similique sunt in culpa qui officia deserunt mollitia animi, id est laborum et dolorum fuga. Et harum quidem rerum facilis est et expedita distinctio. Nam libero tempore, cum soluta nobis est eligendi optio cumque nihil impedit quo minus id quod maxime placeat facere possimus, omnis voluptas assumenda est, omnis dolor repellendus. Temporibus autem quibusdam et aut officiis debitis aut rerum necessitatibus saepe eveniet ut et voluptates repudiandae sint et molestiae non recusandae. Itaque earum rerum hic tenetur a sapiente delectus, ut aut reiciendis voluptatibus maiores alias consequatur aut perferendis doloribus asperiores repellat. 


---
-->


# Design process
Initially, the goal was to power and control 6-8 tubes (up to 88 cathodes) which very quickly became a routing nightmare. Also the lebensraum cost accumulated very quickly so small pcbs became a necessity. 
But first things first. 

## Assumptions
For dual tube version:
1. DC-DC step-up converter from 12V in to ~170V out;
2. Switching frequency: ~110kHz
3. Output regulation method: PWM 
4. Power delivered to the load: 2.04mW (4 symbols, up to 3mA per cathode at 170V);
5. Small size: 50mm x 50mm boards (cheap, easy to throw away)
6. 22 cathodes, 2 out of which will stay on and 2 decimal points w/ independent control
7.  

## First try
Ok. so let's try single phase boost topology in Constant Current Mode. 
![Boost converter schematic](/files/NixieDriverEvenSmallerNoDot/img/NixieDriverModule_SchBoost.png)

In that case helpful can be ic dedicated to boost converter control such as UCC38xx family
![](files/NixieDriverEvenSmallerNoDot/img/NixieDriverModule_SchDriver.png)

### Duty cycle
DC needed is given by
```math
U_{OUT} = \frac{U_{IN}}{1-D}  \implies  D = 1 - \frac{12V}{170.92V} \approx 0.9298
```
so half of the UCC38xx family is already out. 

Let's see: 
<!-- <img src="NixieDriverModule_UCC38xxComp.png" alt="UCC38xx family comparison" > -->
![](files/NixieDriverEvenSmallerNoDot/img/NixieDriverModule_SchCtrlNote.png)
After DC calculation, only '00, '02 and '03 are left. 

As input voltage is 12V, let's look at histeretic UnderVoltage LockOut thresholds. '02 version will never even turn on and '03's range is far too low to be usable. That leaves '00 as the last chip standing. 


### Inductor
Assuming 75% efficiency, mean input current $` I_{IN,AVG} = 12mA / 0.75 = 16mA`$. 

For continuous inductor current, half of peak-to-peak value of the ripple shouldn't be bigger than that mean.

As voltage across the inductor $` U_{L} = L \frac{\Delta i}{t_{ON}} `$ and duty cycle $` D = \frac{t_{ON}}{T} `$, ripple current through the inductor is: 
```math
\Delta i = \frac{U_{L} D}{f \cdot L} = \frac{12V \cdot 0.9298}{110k \cdot }
```

### Diode
[ES1D](https://www.onsemi.com/pdf/datasheet/es1d-d.pdf) (SMA) was chosen as a cheap yet fast and powerful diode for its 
- **200 V** Maximum Repetitive Reverse Voltage and
- **1 A** Average Rectified Forward Current with
- **30 A** pulse capability. 

### Switch
[BSC900N20NS3 G](https://www.infineon.com/dgdl/Infineon-BSC900N20NS3-DS-v02_02-en.pdf?fileId=db3a30432ad629a6012b144f6b0619db) (PG-TDSON-8) was deemed worthy: 

- **200 V** $`V_{DS}`$ 
- **90 mΩ** max $`R_{DS(on)}`$ 
- **15.2 A** $`I_{D}`$ 


### Output capacitor
$`2 \mu F`$ 

## Simulation

![](/files/NixieDriverModule_BoostPlecs.png)


TODO:  y LaTeX does not work in md?!


![](files/NixieDriverEvenSmallerNoDot/img/NixieDriverModule_AllLayers.png)



![](files/NixieDriverEvenSmallerNoDot/img/NixieDriverModule_SchIO.png)
![](files/NixieDriverEvenSmallerNoDot/img/NixieDriverModule_SchOuts.png)
![](files/NixieDriverEvenSmallerNoDot/img/NixieDriverModule_SchDemux.png)
![](files/NixieDriverEvenSmallerNoDot/img/NixieDriverModule_SchTransArray.png)

![](img/NixieDriverTwoTubes_ANGLE.png)

![](files/NixieDriverEvenSmallerNoDot/img/NixieDriverEvenSmallerNoDot_ANGLE.png)
![](files/NixieDriverEvenSmallerNoDot/img/NixieDriverEvenSmallerNoDot_BACK_blank.png)
![](files/NixieDriverEvenSmallerNoDot/img/NixieDriverEvenSmallerNoDot_FRONT_blank.png)
![](files/NixieDriverEvenSmallerNoDot/img/NixieDriverEvenSmallerNoDot_LEFT.png)
![](files/NixieDriverEvenSmallerNoDot/img/NixieDriverEvenSmallerNoDot_TOP.png)

![](files/NixieDriverEvenSmallerNoDot/img/NixieDriverModule_DIMs.png)
![](files/NixieDriverEvenSmallerNoDot/img/NixieDriverModule_FabBack.png)
![](files/NixieDriverEvenSmallerNoDot/img/NixieDriverModule_FabFront.png)
