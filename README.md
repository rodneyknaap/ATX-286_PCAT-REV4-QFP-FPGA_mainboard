# 286 PC/AT QFP FPGA ATX mainboard revision 4  
![A picture of the mainboard design front side](REV4_QFP_FPGA_MICRO-ATX_TOP.png)  

ATX 286 PC/AT REV4 QFP FPGA mainboard design based on original IBM 5170 PC/AT technology  

This project is Revision 4 of my open source 286 AT ATX PC mainboard design project.
This will be the first actual system design undergoing testing and debugging phases using FPGA technology.

As with the first and third revisions, this design is based on original IBM 5170 PC/AT concepts and reverse engineered circuits from the 5170 U87 PAL.  

With this project as with previous revisions the goal will be to maintain as much of the original PC/AT technology as possible.
In REV1 and REV3D I am aware of the fact that I have employed partial asynchronous design.
With the REV3E I have released a system controller programming project which attempts to apply more synchronous regions in the system control logic inherited from the IBM 5170.  
Any asynchronous areas are for the time being fully intentional as they are because I have not been able yet to use a fully synchronous design in the system controller CPLDs of previous revisions.  
Development and testing with a CPLD is severely limited in numbers of available registers and output enable functions.
So far, this has prevented me from getting to a functional design using a fully synchronous design method, however a more synchronous version of the REV3 system controller has now been released by using a higher frequency clock input, the CPU then runs at 16MHz. Please refer to the REV3E repository for this quartus project.  

Now that we are going to use an FPGA here in the REV4 stage, it's my hope that we can create a new system control model which doesn't depend on any asynchronous setting and resetting of registers in the design, or at least greatly reduce doing so in the areas that really matter to begin with.  
A higher clock speed and a sufficient number of registers will help to realize this where there will be more subtle timing control possible thanks to the higher clock "resolution".
In addition, I plan to use the available FPGA memory blocks to generate variable timing where the clock transition moments are created in FPGA memory blocks. The bitstream from the selected memory block will then drive a single system of decoded shift register outputs that result in a single system control model. So the variation of the system control speeds will be created in the FPGA memory based clock shapes. In addition, the clock shapes from the memory blocks will serve as the dynamic clock basis for the 286 CPU to align it with system control timing.  
The bits in the memory blocks will allow us to drive different cycle scenarios of the 286 CPU dynamically depending on early decoding in different parts of the 286 cycles. Or at least, that's the plan! So the clocking bitstreams will be initially identical until the cycle decoding outputs are established after which the bitstreams will diverge to different transition timing.  

# How this version of the FPGA iteration has evolved  
I have started the FPGA work on another design repository using a large 672 pin Cyclone II BGA FPGA, however this QFP stage is now first going to be developed. I first wanted to test a few design aspects such as the serial flash based configuration using AS mode of the FPGA and core AT controller replacement designs on a test board, and while building up that design, I added and compiled in more and more functionality.   
Eventually I realized that I would actually probably be able to create a fully functional PC/AT design using the 208 pin Cyclone II FPGA by reducing the design complexity in the following ways:  
- we don't feature a separate memory address and data bus  
- we will use the 286 high address lines A17-A22 to drive the LA17-LA22 lines on the ISA slot during CPU cycles. However the FPGA would theoretically still be able to drive these lines were it necessary for example during DMA, because the CPU will be in tri-state, allowing the FPGA to output these lines. There is no DMA to VGA memory and otherwise there is no target RAM present on the slots for DMA. So for this particular system we can ignore outputting these lines for DMA purposes while the CPU is on hold because DMA will be taking place on onboard SRAM which are decoded inside the FPGA. The 286 will drive the LA lines to write to VGA RAM or read the VGA BIOS ROM, which is decoded by the VGA controller chip. In a later stage we could test with a bus master because we will interpret the /MASTER input from the ISA slots.  
- we will reduce DMA to only channel 2(floppy drive controller) and channel 1(sound card)  
- we will ignore a few signals which won't be needed such as the 0WS line, REFRESH and IO_CH_CHK
- no memory bus means that the design will be system bus only, SRAMs and 8 bit mode system ROM will all operate on the system bus instead
- the SRAM will be in 3.3V logic, directly attached on the FPGA side of the system bus, while the ISA slots and system ROM will be located on the 5V side of the system bus.
- we will feature an external PLCC FDC and UART, the PLCC FDC also provides the FDC /DC status read port bit to the CPU.

To expand into a fully integrated PC/AT mainboard and being able to make do with less expansion cards on the slots, a 208 pin CPLD has been added to the design to provide:  
- IO decoding  
- POST LED displays  
- more advanced status LED decoding to indicate system operation:  
  RESET active  
  CPU holding  
  CPU Memory cycle  
  CPU I/O cycle  
  CPU BIOS routines running  
  VGA access  
  CPU conventional memory access  
  CPU XMS memory access  
  DMA cycles  
- dual IDE ports  
- LPT port  
- clock division for the system timer and UART  

In the smaller QFP Cyclone II FPGA we will have enough logic capacity to replace all the core AT controller chips. Hopefully this will work out, but so far it appears to be the case so that's the plan with this project.  

In addition we will attempt to create EMS memory functionality which operates identically to the REV3E EMS by manipulating the system bus.  
The remaining logic capacity will then hopefully support developing the new system control model.  

The project will consist of a Micro ATX form factor mainboard, there are 6 SRAMs on the mainboard which can be used used for XMS and EMS memory intended to support running RealDOOM with drivers developed by sqpat:  
https://github.com/sqpat/RealDOOM  

The system ROM is a single 8 bit mode chip on the lower system data bus.  

The mainboard supports the 80286 16 bit CPU, system bus driving is completely done by the FPGA via bus switch ICs.  

A Harris 286 rated at 20MHz or higher is recommended, the highest clock speed verified ones are known to be made in early 1990s years.  
Basically for composing the core PC/AT system based on IBM 5170 technology all logic is now contained within the FPGA.  
In addition, we will integrate the core PC/AT controller chips in the FPGA:  
- the 8042 keyboard controller  
- the IRQ controllers  
- the DMACs  
- the RTC  
- the DMA page mapper chip  
- the system timer  

## Purpose and permitted use, cautions for a potential builder of this design
This project was created for historical purposes out of love for historical computing designs and for the purpose of enabling computing enthousiasts with a sufficient level of building and troubleshooting expertise to be able to experience the technology by building and troubleshooting the hardware described in this project. Due to the level of this project, it may be suitable as a project for students to get into. If there are any questions from teachers who like to teach about this technology I would be happy to answer them. It may be really interesting to analyse the elaborate and complex CPU timing and 8 bit to 16 bit data byte translation and DMA mechanisms in an educational setting.

Besides the GPL3 license there are a few warnings and usage restrictions applicable:
No guarantees of function or fitness for any particular or useful purpose is given, building and using this design is at the sole responsibility of the builder.

Do not attempt this project unless you have the necessary electronics assembly expertise and experience, and know how to observe all electronics safety guidelines which are applicable.

It is not permitted to use the computer built from this design without the assumption of the possibility of loss of data or malfunction of the connected device. To be used strictly for personal hobby and experimental purposes only. No applications are permitted where failure of the device could result in damage or injury of any kind.

If you plan to use this design or any part of it in new designs, the acknowledgement of the designer and the design sources and inspirations, historical and modern, of all subparts contained within this design should be included and respected in your publication, to accredit the hard work, time and effort dedicated by the people before you who contributed to make your project possible.

No guarantee for any proper operation or suitability for any possible use or purpose is given, using the resulting hardware from this design is purely educational and experimental and not intended for serious applications. Loss of data is likely and to be expected when connecting any storage device or storage media to the resulting system from this design, or when configuring or operating any storage device or media with the system of this design.

When connecting this system to a computer network which contains stored information on it, it is at the sole responsibility and risk of the person making the connection, no guarantee is given against data loss or data corruption, malfunctions or failure of the whole computer network and/or any information contained inside it on other devices and media which are connected to the same network.

When building this project, the builder assumes personal responsibility for troubleshooting it and using the necessary care and expertise to make it function properly as defined by the design. You can email me with questions, but I will reply only if I have time and if I find the question to be valid. Which will probably also lead to an update here. I want to primarily dedicate my time to new project development, I am not able to do any user support, so that's why I provide the elaborate info here which will be expanded if needed.

These disclaimers and conditions may seem unfriendly but remember that they are by no means meant to reflect on you as a reader personally or individually, just imagine that all possible people and unwise use and situations still need to be covered since this project is openly published on the internet, which means any person on the planet is able to find the information, thus also the comments are meant for every possible person who wants to use the information. I am reasonably assuming that 99% of people will be civilized enough to observe respect and common sense.

# Revision 4 design of a PC/AT mainboard based on QFP FPGA technology  
For background information and previous acknowledgements, please first see the REV1 and REV3E design repository, as well as the REV4 large BGA design repository. 
Please note: the REV4 PC/AT BGA FPGA system design using the 672 pin FPGA will continue after this system has been built and debugged/developed.

The information provided here is purely meant to describe the differences and changes in the REV4 QFP FPGA PC/AT system design.

## using LA23 to prevent accidental decoding by the VGA controller
The system design will attempt to feature EMS memory on the system bus. This poses some design challenges because on the same system bus we have memory (VGA) and IO which responds to the address and command states that appear on the system bus. When invalid address states are asserted on the system bus this could potentially lead to false decodes and crashes/freezing of the system.  
Invalid system address states could be produced during zero wait state timing of the system address bus as well as during EMS enabled RAM cycles where the address states are output from the EMS page registers.
In order to prevent the VGA controller and IO decoder on the mainboard to be falsely triggered by these address events, I am planning to use the LA23 line. By raising this line during non slot targeting memory cycles by the 286, this will eliminate the VGA controller on the VGA card to be falsely triggered to decode VGA RAM or VGA BIOS ROM cycles. In addition, LA23 is routed to the IO decoder CPLD and may be included there to disable all IO decodes on the chip during LA23 being raised high. Our system of this project contains 6MB of SRAM, which means that the system RAM will ever be only featured in the lower half of the memory map.  

The EMS system is planned to be able to take control only of the upper 4MB SRAM chips, where in any case, 2MB of "XMS" RAM will always remain available to guarantee a stable DOS support that contains conventional RAM.  

The PCB layout is now finished.

## A big thank you goes out to everyone who has expressed their appreciation and support for my work, both here on GitHub and elswhere!  

# Special thanks go to GitHub users:  
- htminuslab who developed various very useful HDL core AT controller projects which I will attempt to test and use for this project, for example:  
https://github.com/htminuslab/HTL8237  
- LimeProgramming who developed the excellent USB to serial mouse project:  
https://github.com/LimeProgramming/USB-serial-mouse-adapter  
- maniekx86 who kindly permitted me to use his HDL 8042 keyboard controller which he created for his project:  
https://github.com/maniekx86/M8SBC-486  

## Project goals  
The purpose of my project is to preserve historic technology by doing a best effort attempt to recreate closed designs which would otherwise be doomed to eventually become lost in time, such as following the concepts used in period chipsets and most notably preserving the IBM 5170 and 5162 technology which is at the basis of all AT PC technology that followed in the years after the IBM PC team created these systems.  

A secondary purpose besides preservation is to achieve higher clock speeds and gain more efficiency from the 286 CPU, and reaching a more "clean" type of design, ie. more integration and less separate controller ICs in the system, resulting in a cleaner look of the board and overall cleaner system.  

I have literally zero funds to finance this project so if anyone wants to help me in any way please reach out.  

Also if someone in the PC industry is still around after so many years have passed, and pleased to find my work which is also meant to attribute the importance of their technology (one can hope) please reach out, I would love to hear about it and exchange valuable experiences which will inspire me and all other retro PC enthusiasts. Your work mattered and we all hope for more detailed insider information from this era where internet forums were not established yet.  

# Micro ATX form factor PCB layout is now finished (21-6-2026)  
The gerber files and a few PDF support documents are available in the project directory.  

A few further notes regarding the PCB:  
- USB or AT (PS/2) keyboard and mouse are both possible, please refer to the jumper block near the keyboard connector which enables to either connect the keyboard and mouse pins from the FPGA to the back connectors, or to connect these with the PS2X2PICO for which there is a footprint on the mainboard. Please refer to the project by NoOne here on GitHub, and thanks go out to his great work making it possible to convert the PS/2 mouse and keyboard to USB:  
https://github.com/No0ne/ps2x2pico  
- USB to serial mouse conversion created by LimeProgramming is also included with a dedicated footprint for the RP2040 PICO which can be programmed with his project to connect a USB mouse or mouse receiver and connect as a serial mouse to the DOS driver or windows 3.1 support etc:  
https://github.com/LimeProgramming/USB-serial-mouse-adapter  
- Support for the PicoGUS by polpo as featured here on GitHub is also included here on the mainboard, with many thanks to polpo for all his hard work!
Please note: the logic provided here is untested and unverified with the PicoGUS programmed RP2040 so no guarantees are given that this will work and please refrain from contacting polpo about my non standard implementation here on this board, instead please wait for my test results as an indication if and how well it can work. Particularly I will need to test PicoGUS IO compatibility with a faster clocked 286 CPU. Otherwise the DMA speeds are likely low enough.
More details about polpos project which is always under development can be found on his GitHub page:  
https://github.com/polpo/picogus
- the design is in prototype phase. Ordering PCBs and building the system is at the builder's own responsibility to make this work correctly. You are free to contact me however I promise no support, though I am open for reading your messages at least which is welcome as long as in a civilized tone.
- I will publish a quartus project for programming the FPGA and CPLD as soon as I am satisfied with a certain level of stability that makes the system sufficiently reliable.  

## Debugging work on the REV4 QFP FPGA stage  
I am going to start development work on the FPGA where the initial goal will be to get the 286 CPU to be able to initialize and show POST diagnostic codes or beep sequences. In addition, status LEDs are available to indicate what the CPU is doing. The POST codes and status LEDs could then possibly provide clues to continue the debugging process. In order to output status information the 286 CPU will need to at least be able to read the ROM and do the necessary I/O reads and writes. There are a lot of factors involved to be able to achieve this level of functionality. To a certain degree the HDL core AT controller equivalents will be needed to provide beep sounds and get the system through the POST sequence far enough that the CPU will be able to initialize the VGA BIOS and start to display on the screen. Part of CPU operations will depend on the existing initially asynchronous system control which may or may not be functional up to certain levels. This aspect of the design will become apparent as soon as I am able to test with the existing designs developed up to the REV3E stage. Hopefully these can function up to a certain level even if only in a very limited way so we may then continue to apply more and more synchronous areas to evolve the system control.  

![Photo of bottom area under the Cyclone II](Cyclone_II_bottom_side_area.jpg)   

Please note in the photo how I soldered the same linear regulator as the main core supply voltage for the PLL, which is a pretty easy process to do after removing a small section of solder mask on the ground place in the area involved.  

## Update october 25th, 2026  
Okay so I think I have all the components collected now that I need to build the REV4 QFP mainboard.  
So I will document the process of building the mainboard here in case anyone wants to build one for themselves.  
It has been a long process because the mainboard design contains a lot more technical complexity. Which starts with the level shifting, higher frequency operation of the Cyclone II compared to the MAX7000S CPLDs. The Cyclone FPGA is capable of switching with a much higher speed so I am taking a lot of precautions. We are using tiny 0402 sized 100nF bypass capacitors on the FPGA pins and other areas that are expected to have higher switching speeds of signals. Apparently, the tiny size of the 0402 capacitors results in a smaller inductance value, though I still have some doubths whether this actually will form any issue here in this project. Anyway, it's an opportunity to get some experience designing with 0402 components and assembling them on the PCB.  

Well, the 0402 capacitors are absolutely tiny, and I have needed a small period of working with these and adjusting my methods to solder them.  I would advise anyone building this project to start off with the 0402 capacitors on the bottom of the PCB and get these straightened out first of all. Next I would suggest adding the remaining components such as the linear regulators supplying the core voltage and PLL voltage. I supply these separately in the system so they will influence eachother less. So I take 3.3V from the ATX PSU and pass it through some diodes to decrease the voltage further, then pass them off to some low drop regulators directly underneath the Cyclone II. The mainboard has such incredible heat dispersal, I doubt the regulators will get more than only a little warm with the huge thermal conduction.  So I used a Microchip regulator on the design for the PLL voltage however because of some tax rules it was not shipped. So I just used one of the larger capacity regulator D packages for the PLL supply voltage. Just scrape off some solder mask and it goes onto the board just fine. In fact, I will make a note to feature a dual footprint so a future builder can have more options to choose some regulator package that is available.  

Okay, so problem of the regulator was quickly solved. I did make the mistake of soldering the power diodes before the 0402 capacitors so I was somewhat hampered for a few of the capacitors. Anyway, so that's why I advise just solder the 0402 caps first before anything else so you can reach them well. I would advise a short cone tip on the iron, to make sure you have proper heat conduction to the capacitors, and also the somewhat pointy tip allows you to direct heat precisely onto the pad so you can make good solder bonds. For larger components I would suggest a short wedge tip because it can direct even more heat to the solder point if you can touch it well with the iron.  

After the 0402 capacitors I proceeded to the regulators and all other components underneath the Cyclone II. I do this first so I don't need to heat up the mainboard anymore after I get the Cyclone II soldered on the top side since the pins are tiny and we want to maintain a good connection on them.  

After doing the entire bottom area under the Cyclone II, I proceeded to solder on all the 0805 SMD components to the bottom since those are needed anyway and we would want these done first, and I also soldered the 0805 top SMD resistors and capacitors since later there will not be access to these after soldering in a ISA slot etc.  

After all the SMD stuff is on, I will solder the ATX power circuits and PSU connector. Then I will add the IO decoder CPLD and the JTAG header for it. I will then first proceed to program an initial IO decoder project which takes care of the necessary general IO decoding and some clock division. I will leave the PicoGUS logic for after I have gotten the system up and running on the Cyclone II which is still a way to go to achieve this. Of course, when we have a running PC/AT and done some simple testing, I will continue with the PicoGUS because I am eager to start testing this. Using the Cyclone II, we should have ample means to control all the I/O cycles in case these will need some adjusting. Our target clock speed will be around 25MHz for the 80286 CPU, so we will need to see whether the typical cycle control can be sufficient. For now I am optimistic, unless experiencing any issues, I expect that we should be fine.  

So after programming the initial test project for the CPLD, I will then start working on finalizing the Cyclone II quartus project. We are using a 32MHz oscillator to provide the PLL input clock, so I will set some test frequency for 286_CLK. Also I will be porting the new synchronous 80286 system control design which I have developed on the REV3E project to this FPGA, and I kind of expect it to work without any issues. Unless we get some glitching or whatever unexpected FPGA related issues of course. And I still need to test the core IO controller logic as well to see if these components can work well with this system design. So we have a POST display which can help a lot with diagnostics information, depending on whether the 80286 will be able to perform IO cycles correctly of course. We will be testing with the MR BIOS generic 80286 BIOS and XT-IDE. The IO decoder has dual IDE ports and a printer port, and this time features a number of LED outputs which are intended to signal lots of system activity which the enthusiast could bring out on a front panel just for a more enjoyable experience to use the new system. As mentioned, we plan to support RealDOOM with the existing EMS driver which sqpat developed in cooperation with me. However, because of the limited pin count on the QFP Cyclone II, I may need to use certain design tricks to prevent the VGA controller in the system to get a false decode from EMS generated address lines which after all need to be passed to the SRAMs, which are jointly on the system bus with the ISA slots. So it's a highly compressed PC/AT design here which we have needed to do more with less, so to speak, in terms of available pins.  

Okay so I will update on the progress as soon as I have more information. The assembly has started and is on the way. More news and tips are coming up!  

I am also developing a variable clock oscillator which we can use to find the clock speed limits on the REV3E system. I am using a Si5351A Adafruit module which I plan to program using a separate board with a ATMEGA328P controller and a small display. So we will create some project that allows to wire up the Si5351A onto the REV3E system and plug the programmer board project into the clock module which then can set any clock frequency freely using the Adafruit clock module. I briefly thought about going the analog route to use LC oscillation and a FET transistor circuit however I found these somewhat tricky and it would need a whole new PCB design type where the focus is on high frequency circuits and stray capacitance and inductance. I have not been able to find my book on oscillators and the circuits I found so far are not really working out. I was only able to get to around 44MHz I think. Anyway just let it be for now and I will test with the Adafruit Si5351A module first. Maybe if I make a next PCB order, I will also design a test and experimentation PCB where I have lots of ground area and some soldering islands where it's very easy to add and remove any components and maintain a better HF response from the circuit as wired up. I think then I should easily be able to reach from 80-120MHz which is my target range. If this pans out I may develop it into a small module with a simple trimmer capacitor or potentiometer with a varactor diode which may also do the trick. I will then make a project about it here. Also the variable oscillator with the Adafruit module will get featured here on GitHub as soon as I get it up and running. The REV3E is showing a lot of promise to possibly be able to operate at 25MHz now, however we will need a 100MHz clock input on the system controller which is a little challenging to come by, and I also have given it more thought that even after verifying 25MHz, we still will want to find the actual real clock limit anyway. So the better path is to get a variable clock oscillator going, comperable to the FPGA here which can be reprogrammed to feature a range of possible clock speeds on the 80286.  

Thanks for your interest, and if you like the project, I want to ask you to consider adding a star here which helps me to show me how much interest there generally is for this specific project among the people who come across the repository. This will also influence motivation to continue on with the 80486 which will be up next. There we will choose a suitable 486 CPU which can work well with system that increasingly uses 3.3V. I also have some leftover 3.3V MAX7000 CPLDs which I accidentally ordered from Ebay before, which could also work well in expanding the 486 system IO and various general system functions in our first 80486 design.  

# Updates regarding the project assembly process can be found on my website  
On my website you can find a detailed write up of the process of building the REV4 QFP system:  
[https://www.knaapic.nl/phpBB3/viewtopic.php?p=23#p23](https://knaapic.nl/phpBB3/viewtopic.php?p=23#p23)  
Please note: in the photos you can see the state of assembly in each phase.  
Phase 1 is enough to power on and measure all voltages and clocks.  
And the IO Decoder CPLD can be test programmed as well.  
Check my post in the link above for a lot more tips, details and photos of how to do the assembly phase 1.  

A lot of information will be elaborated more deeply on my website, which is not present on the GitHub pages, particularly my visions, experiences and design steps are elaborated there as they were coming up. A lot of additional insider knowledge and know-how can be found on my website. The PC/AT design is not only of importance for the 80286 CPU, but also in maintaining a correct and fully compatible PC/AT standard when transitioning into other CPUs like the 80486 and the Pentium, and onward from there. We can't skip or omit the core controller functions if we want a properly functioning system comperable with the original time period systems. Do note that what I am working on in my projects is to preserve and maintain the original system functions and behaviors, where I intentionally will be fully maintaining all the design aspects as originally created by IBM in the 5170, which was the foundation of all PC systems evolving from there. So we want to maintain the functions, and inherit these into later CPU based systems in the future.  
  
The forum link is:  
[https://knaapic.nl/community/project-to-create-an-atx-80286-mainboard-based-on-the-ibm-5170](https://knaapic.nl/community/main-forum/project-to-create-an-atx-80286-mainboard-based-on-the-ibm-5170)  
The forum thread offers more in-depth articles describing the actual process of development of the project.  
More forum threads will be created there as projects are evolving, such as the future  80486 design.  
  
Thank you for your interest, I look forward to hearing from you!  
Please consider if you are able to help, to reach out to me to help me source parts and test system boards, or if you are able to help in any other way.  
From my projects, you can see that I respect the original technology and provide a clear path to preserving the technology in increasingly modern forms. The goal is not any form of emulation, but preserving the original system behavior and design aspects, I will go as far to maintain the core elements in the designs as far as this is technically possible to do.  By choosing increasingly more modern technology, we can get a more and more portable design documented which will be increasingly more convenient to reproduce with other hardware like FPGA development boards, for example. We would want to keep the time period CPUs able to function with the new system design recreation around them using more modern technology.  
  
Kind regards,  
  
Rodney  

Updated last on october 30th, 2026.
